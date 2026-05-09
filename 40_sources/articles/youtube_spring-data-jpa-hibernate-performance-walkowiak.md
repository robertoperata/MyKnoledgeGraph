---
tags:
  - java
  - database
  - performance
  - architecture
type: article
author: Maciej Walkowiak
source: https://www.youtube.com/watch?v=exqfB1WaqIw
date: 2024-03-13
---

# Performance Oriented Spring Data JPA & Hibernate

**Speaker:** Maciej Walkowiak (consulente freelance, Spring developer)  
**Evento:** Devoxx  
**Durata:** 43:44  
**Lingua originale:** inglese

---

## Sintesi

Maciej Walkowiak, consulente freelance che cambia spesso progetto, identifica i problemi di performance JPA/Hibernate che incontra sistematicamente in produzione: cattivo connection management, troppe query, query lente, mapping errati che fetchano più dati del necessario. Il talk mostra come diagnosticarli e risolverli con strumenti pratici e pattern concreti.

Il punto di partenza è contrastintuitivo: il problema raramente è Java o il database troppo piccolo — il problema è quasi sempre il codice o il modo in cui vengono usati i framework. Scalare verticalmente o aggiungere istanze è una soluzione temporanea; il vero fix richiede capire cosa genera il traffico al database.

La prima sezione affronta il **connection management**: Spring Boot usa Tomcat (200 thread) con HikariCP (10 connessioni di default) — un mismatch evidente. Due default pericolosi da disabilitare subito: `spring.jpa.open-in-view=true` (tiene le connessioni aperte durante la serializzazione della risposta HTTP) e `spring.datasource.hikari.auto-commit=true` (apre la transazione all'inizio del metodo `@Transactional` anche prima che arrivi a toccare il database). Il `TransactionTemplate` è la risposta per chi ha bisogno di controllo fine sui confini della transazione.

La seconda sezione smonta un'applicazione bancaria che sembra corretta ma genera query inattese: l'ID assegnato manualmente causa un SELECT indesiderato prima dell'INSERT (Hibernate chiama `merge` invece di `persist`), le associazioni `@ManyToOne` sono eager per default e generano N+1 sulle liste, `save()` senza `@Transactional` sull'usecase apre/chiude sessioni Hibernate separate per ogni operazione. Le soluzioni: `@Version` per segnalare "entity nuova", `getReferenceById()` per creare proxy senza query, LAZY fetch ovunque + JOIN FETCH esplicito nelle query, `@DynamicUpdate` per aggiornare solo le colonne cambiate.

La terza sezione introduce la regola fondamentale: **fetch entities solo se si ha intenzione di modificarle**. Per i casi read-only usare le *projections* — Spring Data genera automaticamente il SQL ottimale dal tipo di ritorno del metodo. Le *Dynamic Projections* permettono di avere un unico metodo nel repository che accetta la forma della projection come parametro, tenendo il repository lean.

---

## Parte 1 — Connection Management

### Il mismatch Tomcat/HikariCP

Spring Boot default:
- Tomcat: **200 thread** (200 richieste concorrenti)
- HikariCP: **10 connessioni** al database

Non è un bug — è by design. La ricerca HikariCP mostra che con carichi elevati è meglio avere poche decine di connessioni e far fare la coda alle richieste, piuttosto che aprire centinaia di connessioni che competono per le risorse del database.

**Il vero problema** non è la dimensione del pool ma quanto a lungo le connessioni vengono tenute occupate.

### `spring.jpa.open-in-view=true` — il default pericoloso

Ogni avvio di Spring Boot logga questo warning ma nessuno lo legge:

```
spring.jpa.open-in-view is enabled by default. Therefore, database queries may
be performed during view rendering.
```

Con open-in-view abilitato, la connessione al database viene acquisita all'inizio della richiesta HTTP e rilasciata solo quando la risposta viene serializzata — dopo che il controller ha già terminato. Risultato: se il controller fa chiamate esterne (REST call, sleep, ecc.) la connessione è bloccata per tutto quel tempo.

```yaml
# application.properties — DA FARE SUBITO
spring.jpa.open-in-view=false
```

Prima: connessione tenuta per 210ms (200ms di sleep nell'endpoint).  
Dopo: pochi millisecondi.

### `spring.datasource.hikari.auto-commit=false` — lazy transaction start

Con auto-commit=true (default), Spring apre la transazione (e acquisisce la connessione) all'inizio del metodo `@Transactional`, anche se la prima operazione sul database è al fondo del metodo. Se prima c'è una chiamata esterna, la connessione è bloccata inutilmente.

```yaml
spring.datasource.hikari.auto-commit=false
```

Con auto-commit=false, Spring ritarda l'apertura della connessione fino al momento in cui serve davvero.

**Limite:** funziona solo se la chiamata esterna è *prima* dell'accesso al database. Se è *dopo*, la connessione è già aperta.

### `TransactionTemplate` — controllo fine della transazione

`@Transactional` è conveniente ma dichiarativo: non permette di definire esattamente quali parti del metodo devono essere nella transazione. `TransactionTemplate` è la soluzione underutilizzata:

```java
// PRIMA: l'intera durata del metodo tiene la connessione
@Transactional
public void processPayment(String id) {
    Account account = accountRepository.findById(id);
    externalPaymentService.call(); // ← connessione bloccata durante la chiamata
    account.markPaid();
}

// DOPO: solo la parte che tocca il database è in transazione
public void processPayment(String id) {
    externalPaymentService.call(); // ← nessuna connessione aperta
    transactionTemplate.execute(status -> {
        Account account = accountRepository.findById(id);
        account.markPaid();
        return null;
    });
}
```

### `PROPAGATION_REQUIRES_NEW` — il costo nascosto

`@Transactional(propagation = REQUIRES_NEW)` apre una **nuova connessione** separata mentre la transazione corrente è ancora aperta. Questo:
1. Richiede che il pool abbia almeno 2 connessioni disponibili
2. La transazione esterna tiene la sua connessione bloccata per tutto il tempo dell'inner transaction

```java
// Anti-pattern: due connessioni simultanee
@Transactional
public void outer() {
    inner(); // ← apre una seconda connessione
}

@Transactional(propagation = REQUIRES_NEW)
public void inner() { ... }

// Preferire TransactionTemplate per delimitare esplicitamente
```

---

## Parte 2 — Hibernate e JPA

### Anti-pattern 1: ID assegnato manualmente → SELECT indesiderato prima di INSERT

`SimpleJpaRepository.save()` chiama `isNew()` sull'entity. Se l'entity ha già un ID, Hibernate la considera *non nuova* e chiama `merge()` invece di `persist()`. `merge()` controlla se l'entity è nel persistence context — se non la trova, fa una SELECT per caricarla.

```java
// PROBLEMA: ID assegnato manualmente → SELECT + INSERT (invece di solo INSERT)
BankTransfer transfer = new BankTransfer(paymentNetworkId, from, to, amount);
bankTransferRepository.save(transfer); // SELECT poi INSERT
```

**Soluzioni:**

```java
// Opzione 1: aggiungere @Version — version null → entity nuova
@Entity
public class BankTransfer {
    @Id
    private String id;
    @Version
    private Long version; // null all'inizio → isNew() = true
}

// Opzione 2: implementare Persistable
@Entity
public class BankTransfer implements Persistable<String> {
    @Transient
    private boolean isNew = true;

    @Override
    public boolean isNew() { return isNew; }

    @PostPersist @PostLoad
    void markNotNew() { isNew = false; }
}
```

### Anti-pattern 2: usecase senza `@Transactional` → sessioni Hibernate separate

Senza `@Transactional` sull'usecase, ogni chiamata al repository apre e chiude una sessione Hibernate separata. Le entity caricate in una sessione non sono "gestite" nella sessione successiva → Hibernate le considera unmanaged e fa query aggiuntive.

```java
// PROBLEMA: 3 sessioni separate → query inattese
public void registerTransfer(String id, String from, String to, BigDecimal amount) {
    Account sender = accountRepository.findById(from);    // sessione 1
    Account receiver = accountRepository.findById(to);    // sessione 2
    BankTransfer transfer = new BankTransfer(id, sender, receiver, amount);
    bankTransferRepository.save(transfer); // sessione 3 — sender/receiver non managed
}

// SOLUZIONE: @Transactional → una sola sessione condivisa
@Transactional
public void registerTransfer(...) { ... }
```

### Anti-pattern 3: fetch di entity solo per usarle come FK

Se serve un'entity solo come riferimento per una foreign key (non per leggerne i dati), non serve caricarla dal database:

```java
// PRIMA: due SELECT inutili
Account sender = accountRepository.findById(from).orElseThrow();
Account receiver = accountRepository.findById(to).orElseThrow();

// DOPO: proxy con solo l'ID, nessuna query
Account sender = accountRepository.getReferenceById(from);    // zero query
Account receiver = accountRepository.getReferenceById(to);    // zero query
```

**Trade-off:** `getReferenceById()` non verifica che l'entity esista. Un ID inesistente causa una `ConstraintViolationException` al flush, non immediatamente.

### Anti-pattern 4: `@ManyToOne` eager di default → N+1

Le associazioni `@ManyToOne` e `@ManyToMany` sono **eager** per default in JPA. Su query che restituiscono liste, questo genera N+1 query.

```java
// PROBLEMA: per ogni BankTransfer, una query separata per sender e receiver
List<BankTransfer> transfers = transferRepository.findBySenderId(senderId);
// 1 query per la lista + N*2 query per le associazioni eager

// SOLUZIONE: rendere LAZY + JOIN FETCH esplicito
@ManyToOne(fetch = FetchType.LAZY)
private Account sender;

// Query con JOIN FETCH esplicito
@Query("SELECT b FROM BankTransfer b JOIN FETCH b.sender JOIN FETCH b.receiver WHERE b.sender.id = :id")
List<BankTransfer> findBySenderId(@Param("id") String id);

// Alternativa: Entity Graph
@EntityGraph(attributePaths = {"sender", "receiver"})
List<BankTransfer> findBySenderId(String id);
```

### `@DynamicUpdate` — aggiornare solo le colonne cambiate

Hibernate aggiorna **tutte le colonne** di default. Su tabelle con molte colonne o con colonne con dati large (BLOB, TEXT lunghi), può essere inefficiente:

```java
@Entity
@DynamicUpdate  // solo le colonne cambiate vengono incluse nell'UPDATE
public class BankTransfer { ... }
```

**Trade-off:** Hibernate deve tracciare quali campi sono cambiati → più memoria e CPU. Valutare caso per caso, specialmente su entity con molte colonne o dati large.

---

## Parte 3 — Projections

### La regola fondamentale

> **Fetch entities SOLO se si ha intenzione di modificarle.** Per i casi read-only, usare projections.

Le entity nel persistence context occupano memoria e CPU (dirty checking). Se si carica un'entity solo per serializzarla in JSON non serve tutto questo overhead.

### Spring Data Projections (records)

```java
// Projection come record
public record AccountSummary(String id, String name, String email) {}

// Spring Data inferisce il SQL dal tipo di ritorno
interface AccountRepository extends JpaRepository<Account, String> {
    List<AccountSummary> findByCity(String city);
    // genera: SELECT a.id, a.name, a.email FROM account a WHERE a.city = ?
}
```

Spring Data analizza il tipo di ritorno, identifica i campi, e genera il SQL ottimale — zero overhead rispetto al fetch dell'entity completa.

**Nota:** con JPQL si possono usare record; con SQL nativo la projection deve essere un'interfaccia (scelta di design poco elegante secondo il presenter).

### Dynamic Projections — repository lean

Con molti use case diversi il repository si riempie di metodi con nomi bizzarri e type diversi. Le *Dynamic Projections* risolvono il problema:

```java
interface AccountRepository extends JpaRepository<Account, String> {
    // Un unico metodo per tutte le projections
    <T> List<T> findByCity(String city, Class<T> projectionType);
}

// Uso
List<AccountSummary> summaries = repo.findByCity("Roma", AccountSummary.class);
List<AccountDetail> details = repo.findByCity("Roma", AccountDetail.class);
```

Il repository non cresce al crescere dell'applicazione. Il test del repository non cambia; ogni use case testa la propria projection.

**Limite:** il WHERE clause deve essere derivato dal nome del metodo — non è possibile scrivere query custom con JPQL.

---

## Strumenti di diagnostica

| Tool | Tipo | Uso |
|---|---|---|
| **FlexiPool** (Vlad Mihalcea) | OSS | Logging preciso di acquire/release della connessione |
| **datasource-proxy-spring-boot-starter** (Arthur Gavlyukovskiy) | OSS | Integrazione facile dei proxy datasource con Spring Boot |
| **OpenTelemetry / Micrometer** | OSS | Distributed tracing — visualizza ogni query come span |
| **Digma** | Commerciale/freemium | Plugin IDE che segnala N+1 e anomalie nel codice |
| **Hibernate Optimizer** (Vlad Mihalcea) | Commerciale | Analisi automatica dei mapping — "pair programming con Vlad" |
| **QuickPerf** (Jean-Bieh…) | OSS | JUnit assertions sul comportamento del datasource |

### QuickPerf — assert DB behavior nei test

```java
@QuickPerfTest
@Test
public void assignPhoneNumber() {
    // Esegue il test normalmente...
    service.assignPhoneNumber(accountId, "+39 02 123456");
    
    // ...e verifica il comportamento atteso del datasource
}
// Annotazioni disponibili (da aggiungere al test):
// @ExpectSelect(1)
// @ExpectUpdate(1)
// @ExpectDelete(0)
// Se il comportamento reale differisce → test failure con dettagli
```

Utile per garantire che i mapping non vengano degradati da modifiche successive (regression testing del comportamento SQL).

---

## Citazioni notevoli

> "There are two types of people: those who care about performance, and those who eventually will — because with these frameworks you will hit performance issues even with relatively low traffic applications."  
> — Maciej Walkowiak

> "Fetch an entity only if you have an intention to modify it. For reading, fetch projections instead."  
> — Maciej Walkowiak (regola fondamentale)

> "TransactionTemplate is a super nice and underutilized feature."  
> — Maciej Walkowiak

> "You don't have to use Hibernate. SQL is also very high level, very convenient. Just keep it as an option."  
> — Maciej Walkowiak

---

## Riepilogo degli anti-pattern e fix

| Anti-pattern | Causa | Fix |
|---|---|---|
| Connessione bloccata durante view rendering | `open-in-view=true` | `spring.jpa.open-in-view=false` |
| Connessione acquisita troppo presto | `auto-commit=true` | `spring.datasource.hikari.auto-commit=false` |
| Connessione bloccata durante chiamate esterne | `@Transactional` granularity troppo alta | `TransactionTemplate` per delimitare esattamente |
| Due connessioni simultanee | `REQUIRES_NEW` propagation | Ristrutturare con `TransactionTemplate` |
| SELECT + INSERT invece di solo INSERT | ID assegnato manualmente | `@Version` o `Persistable` |
| Query extra per entity non managed | Usecase senza `@Transactional` | Aggiungere `@Transactional` sull'usecase |
| SELECT inutili per associazioni FK | `findById()` per entity usate solo come FK | `getReferenceById()` |
| N+1 su liste | `@ManyToOne` eager | `FetchType.LAZY` + JOIN FETCH esplicito o Entity Graph |
| UPDATE di tutte le colonne | Default Hibernate | `@DynamicUpdate` (valutare trade-off) |
| Entity caricate solo per serializzazione | Uso di entity per read-only | Projections (record o Dynamic Projections) |

---

## Riferimenti e risorse

- **FlexiPool** ([github.com/vladmihalcea/flexi-pool](https://github.com/vladmihalcea/flexi-pool)) — Vlad Mihalcea; connection pool sizing e monitoring
- **datasource-proxy-spring-boot-starter** ([github.com/gavlyukovskiy/spring-boot-data-source-decorator](https://github.com/gavlyukovskiy/spring-boot-data-source-decorator)) — Arthur Gavlyukovskiy; integrazione Spring Boot per proxy datasource
- **Hibernate Optimizer** ([hibernate-orm-tools.com](https://vladmihalcea.com/hibernate-performance-advisor/)) — Vlad Mihalcea; strumento commerciale di analisi performance
- **QuickPerf** ([github.com/quick-perf/quickperf](https://github.com/quick-perf/quickperf)) — assertions JUnit sul comportamento SQL
- **Digma** ([digma.ai](https://digma.ai)) — plugin IDE per rilevamento N+1 e anomalie
- **HikariCP Connection Pool Sizing** — articolo sul sito HikariCP che spiega perché pool piccoli sono migliori di pool grandi
