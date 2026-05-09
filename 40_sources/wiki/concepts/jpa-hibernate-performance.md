---
title: JPA & Hibernate Performance Patterns
type: concept
tags: [thread2-java]
sources:
  - "[[youtube_spring-data-jpa-hibernate-performance-walkowiak]]"
  - "[[spring_transactional-declarative-annotations]]"
updated: 2026-05-09
related:
  - "[[concepts/java-concurrency]]"
  - "[[concepts/garbage-collection]]"
  - "[[patterns/transactional-outbox]]"
  - "[[patterns/modular-monolith]]"
  - "[[concepts/spring-transactional]]"
---

# JPA & Hibernate Performance Patterns

## Il problema di fondo

Con ORM ad alto livello come Hibernate, il comportamento a runtime è spesso sorprendente. Il codice appare corretto ma genera query inattese. Le cause più frequenti identificate in produzione (Walkowiak, Devoxx 2024):

1. Cattivo **connection management** — connessioni tenute aperte troppo a lungo
2. Troppo **N query invece di 1** — associazioni eager, sessioni separate
3. **Fetch di più dati del necessario** — entity complete dove bastano projections

> "There are two types of people: those who care about performance, and those who eventually will." — Maciej Walkowiak

---

## Connection Management

### Anti-pattern 1: `spring.jpa.open-in-view=true` (default Spring Boot)

La connessione al database viene acquisita all'inizio della richiesta HTTP e rilasciata solo dopo che la risposta è stata serializzata. Se il controller fa chiamate esterne (REST, sleep, ecc.) **la connessione è bloccata** per tutto quel tempo.

```yaml
# Fix immediato
spring.jpa.open-in-view: false
```

### Anti-pattern 2: transazione aperta troppo presto

Con `auto-commit=true` (default), Spring acquisisce la connessione all'inizio del metodo `@Transactional`, anche se il primo accesso al DB è alla fine del metodo.

```yaml
# Lazy transaction start: la connessione viene acquisita solo quando serve
spring.datasource.hikari.auto-commit: false
```

### Anti-pattern 3: `@Transactional` a grana troppo grossa

`@Transactional` è dichiarativo — non permette di limitare la transazione a un sottoinsieme del metodo. Se c'è una chiamata esterna nel mezzo, la connessione rimane bloccata.

**`TransactionTemplate`** risolve il problema con controllo esplicito:

```java
// PRIMA: connessione tenuta per tutta la durata (inclusa la chiamata esterna)
@Transactional
public void processPayment(String id) {
    Account account = accountRepository.findById(id);
    externalService.call();   // ← connessione bloccata qui
    account.markPaid();
}

// DOPO: connessione acquisita solo dove serve
public void processPayment(String id) {
    externalService.call();   // ← nessuna connessione
    transactionTemplate.execute(status -> {
        Account account = accountRepository.findById(id);
        account.markPaid();
        return null;
    });
}
```

### Anti-pattern 4: `PROPAGATION_REQUIRES_NEW`

Apre una **seconda connessione** mentre la transazione corrente è ancora aperta. Richiede 2 connessioni dal pool simultaneamente; la transazione esterna rimane bloccata per tutta la durata dell'inner.

Usare `TransactionTemplate` per delimitare esplicitamente i confini invece di `REQUIRES_NEW` quando possibile.

### Sizing del connection pool

Spring Boot default: **200 thread Tomcat** + **10 connessioni HikariCP**. Il mismatch è intenzionale: la ricerca HikariCP mostra che con carichi elevati poche decine di connessioni sono più efficienti di centinaia — l'overhead di switching supera il beneficio del parallelismo.

Il vero obiettivo non è dimensionare il pool, ma **minimizzare il tempo di lease** di ogni connessione.

---

## Entity Anti-Patterns

### Anti-pattern 5: ID assegnato manualmente → SELECT prima di INSERT

`SimpleJpaRepository.save()` chiama `isNew()`. Se l'entity ha già un ID, Hibernate la considera *non nuova* → chiama `merge()` → cerca l'entity nel persistence context → se non la trova, esegue una **SELECT inutile**.

```java
// Fix 1: @Version — version null → isNew() = true → persist() invece di merge()
@Entity
public class BankTransfer {
    @Id private String id;
    @Version private Long version; // null = entity nuova
}

// Fix 2: implementare Persistable
@Entity
public class BankTransfer implements Persistable<String> {
    @Transient private boolean isNew = true;
    @Override public boolean isNew() { return isNew; }
    @PostPersist @PostLoad void markNotNew() { isNew = false; }
}
```

### Anti-pattern 6: usecase senza `@Transactional` → sessioni separate

Senza `@Transactional` sull'usecase, ogni chiamata al repository apre e chiude una sessione Hibernate separata. Le entity caricate in una sessione sono "unmanaged" nella successiva → Hibernate fa query aggiuntive.

```java
// PROBLEMA: 3 sessioni separate → SELECT extra inattesi
public void register(String id, String from, String to) {
    Account sender = accountRepo.findById(from);     // sessione 1
    Account receiver = accountRepo.findById(to);     // sessione 2
    BankTransfer t = new BankTransfer(id, sender, receiver);
    transferRepo.save(t);                            // sessione 3: sender/receiver unmanaged
}

// FIX: una sola sessione condivisa
@Transactional
public void register(...) { ... }
```

### Anti-pattern 7: `findById()` per entity usate solo come FK

Se un'entity serve solo come riferimento per una foreign key (non si leggono i suoi dati), `findById()` esegue una SELECT inutile. `getReferenceById()` crea un proxy con solo l'ID:

```java
// PRIMA: SELECT per sender e receiver (anche se non ne leggiamo i dati)
Account sender = accountRepo.findById(from).orElseThrow();

// DOPO: zero query — proxy con solo l'ID
Account sender = accountRepo.getReferenceById(from);
// Trade-off: nessuna verifica di esistenza → ConstraintViolationException al flush se non esiste
```

### Anti-pattern 8: `@ManyToOne` EAGER di default → N+1

Tutte le associazioni `@ManyToOne` e `@ManyToMany` sono **EAGER per default** in JPA. Su query che restituiscono liste, ogni elemento triggera query separate per le associazioni.

```java
// FIX: sempre LAZY + JOIN FETCH esplicito per use case
@ManyToOne(fetch = FetchType.LAZY)
private Account sender;

// Query esplicita con JOIN FETCH
@Query("SELECT b FROM BankTransfer b JOIN FETCH b.sender JOIN FETCH b.receiver WHERE b.sender.id = :id")
List<BankTransfer> findBySenderId(@Param("id") String id);

// Alternativa: Entity Graph (più leggibile per relazioni complesse)
@EntityGraph(attributePaths = {"sender", "receiver"})
List<BankTransfer> findBySenderId(String id);
```

### `@DynamicUpdate` — solo le colonne cambiate

Hibernate aggiorna **tutte le colonne** di default. Con tabelle con molte colonne o dati large (BLOB, TEXT):

```java
@Entity
@DynamicUpdate  // genera UPDATE con solo le colonne cambiate
public class BankTransfer { ... }
```

**Trade-off:** Hibernate deve tracciare quali campi sono cambiati → più memoria e CPU. Valutare su entity con molte colonne o dati large.

---

## Projections: la regola fondamentale

> **Fetch entities SOLO se si ha intenzione di modificarle.** Per i casi read-only, fetch projections.

Le entity nel persistence context occupano memoria (oggetti completi) e CPU (dirty checking). Per serializzare JSON o rispondere a query di lettura non serve nulla di tutto ciò.

### Spring Data Projections (records)

```java
// Projection come record
public record AccountSummary(String id, String name, String email) {}

// Spring Data inferisce il SQL dal tipo di ritorno → SELECT solo id, name, email
interface AccountRepository extends JpaRepository<Account, String> {
    List<AccountSummary> findByCity(String city);
}
```

### Dynamic Projections — repository lean

Con molti use case, il repository si riempie di metodi con nomi bizarri. Le *Dynamic Projections* mantengono il repository lean:

```java
// Un unico metodo parametrizzato sulla forma della projection
<T> List<T> findByCity(String city, Class<T> projectionType);

// Uso per ogni use case diverso
repo.findByCity("Roma", AccountSummary.class);
repo.findByCity("Roma", AccountDetail.class);
```

**Limite:** il WHERE clause deve essere derivato dal nome del metodo — non è possibile usare JPQL custom.

---

## Strumenti di diagnostica

| Tool | Uso | Tipo |
|---|---|---|
| `spring.jpa.show-sql=true` | Log SQL in sviluppo | Gratuito (attenzione: non in produzione) |
| **FlexiPool** + **datasource-proxy-spring-boot-starter** | Log preciso acquire/release connessione | OSS |
| **OpenTelemetry / Micrometer** | Ogni query come span nel trace | OSS |
| **Digma** | Plugin IDE — segnala N+1 nel codice | Commerciale/freemium |
| **QuickPerf** | JUnit assertions sul comportamento SQL (`@ExpectSelect(1)`) | OSS |
| **Hibernate Optimizer** (Vlad Mihalcea) | Analisi completa dei mapping | Commerciale |

**QuickPerf** permette di scrivere regression test sul comportamento SQL — se qualcuno introduce un N+1 in futuro, il test fallisce:

```java
@QuickPerfTest
@Test
@ExpectSelect(1)
@ExpectUpdate(1)
@ExpectDelete(0)
public void assignPhoneNumber() {
    service.assignPhoneNumber(accountId, "+39 02 123456");
}
```

---

## Connessioni

- [[concepts/java-concurrency]] — il connection pool è thread-bounded; troppi thread con transazioni lunghe esauriscono il pool
- [[concepts/garbage-collection]] — dirty checking di Hibernate consuma memoria e CPU; entity inutili nel persistence context aumentano la pressione sul GC
- [[patterns/transactional-outbox]] — richiede transazione atomica tra operazione DB e pubblicazione messaggio; i pattern di transaction management qui si applicano
- [[patterns/modular-monolith]] — Fontaine mostra come il DB relazionale con schema per modulo sia la giusta granularità; le performance JPA si applicano dentro ogni modulo
- [[concepts/spring-transactional]] — semantica completa di @Transactional: proxy mode, self-invocation, rollback su checked exception, readOnly hint, REQUIRES_NEW anti-pattern
