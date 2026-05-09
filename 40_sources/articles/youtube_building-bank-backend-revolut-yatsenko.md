---
tags:
  - architecture
  - java
  - event-driven
  - distributed-systems
  - database
  - performance
type: article
author: Vlad Yatsenko
source: https://www.youtube.com/watch?v=FLS7NP8YMUQ
date: 2019-05-16
---

# Anyone Can Build a Bank: Creating a New Banking Backend

**Speaker:** Vlad Yatsenko (co-fondatore e CTO, Revolut)  
**Evento:** Devoxx  
**Durata:** 51:51  
**Lingua originale:** inglese

---

## Sintesi

Vlad Yatsenko racconta la traiettoria tecnica di Revolut dal lancio del 2015 (4 persone, monolite, 3 valute) fino ai 5 milioni di utenti e 90 microservizi del 2019, passando per le sfide specifiche del dominio bancario: correttezza prima di tutto, gestione delle valute, financial crime, fraud detection, e hedging del rischio in real-time.

Il punto di partenza è il **dominio bancario** come dominio di elezione per la correttezza: tutto deve essere consistente, le transazioni sono la priorità assoluta, e i bug costano denaro reale ai clienti. Da questa premessa deriva un'architettura guidata dal problema, non dalla tecnologia: PostgreSQL come database principale (scelto perché le transazioni ACID non sono negoziabili), jOOQ per SQL type-safe, e un framework in-house **event-driven** costruito per eliminare il "service spaghetti" dell'injection delle dipendenze.

L'evoluzione architetturale segue tre fasi: (1) **MVP monolite** — 4 app stateless su PostgreSQL, deploy in 30 secondi, team di 2 ingegneri; (2) **event-driven monolite** — stesse app ma internamente rearchitecturate con "actions" (comandi) che emettono eventi, eliminando le dipendenze cicliche tra service class; (3) **event-driven services** — i moduli vengono separati in servizi indipendenti grazie all'event store centralizzato che funge da ponte asincrono.

La sezione più tecnica riguarda il **fraud detection** (sistema "Sherlock") e il **risk hedging automatico**: entrambi sfruttano il modello event-driven per pre-calcolare score e posizioni di rischio in anticipo rispetto all'evento critico (autorizzazione carta: finestra di 500ms). Il modello dual-write — scrivere sia nell'event log che nello stato corrente — è la chiave che permette risposte immediate senza dover ricostruire lo stato dagli eventi.

Sul lato ingegneristico, l'ingegno culturale più rilevante è la **abolizione dei QA tradizionali**: tutti gli ingegneri fanno TDD (test-first, non "test dopo"), i QA si sono trasformati in "software engineers in test" che costruiscono framework e tooling. Il criterio di valutazione di ogni decisione tecnica è il problema, non la soluzione: questo elimina le "holy wars" tra ingegneri.

---

## Evoluzione dell'architettura Revolut

### Fase 1 — MVP Monolite (2015, team di 2)

```
Mobile App (iOS/Android)
    │
    ▼
API Service ──► Card Processing ──► Back-office
    │
    └──► PostgreSQL (unico DB)
```

- 4 applicazioni deployate da un unico codebase
- Applicazioni stateless → zero-downtime deployment
- Deploy: 30 secondi
- PostgreSQL come unica scelta di storage (ACID non negoziabile per il dominio finanziario)
- Monitoring: New Relic (nessuna infrastruttura interna con 2 persone)

### Fase 2 — Event-Driven Monolite (2016-2017)

Il codebase monolitico aveva sviluppato "service spaghetti": dipendenze cicliche tra service class che rendevano impossibile inizializzare un servizio senza Spring. Soluzione: framework in-house basato su **actions** (comandi) e **eventi**.

```
Action (comando)
    │
    ▼ emette
Event
    │
    ├──► Side effect A (promozione)
    ├──► Side effect B (push notification)
    └──► Side effect C (audit log)
```

Ogni azione emette un evento. I side-effect si sottoscrivono agli eventi senza injection diretta. Nessuna dipendenza ciclica possibile. Il codice è ancora in un unico deployment ma internamente modulare.

**Dual-write model:** ogni operazione scrive contemporaneamente nell'event log (per audit e event-driven architecture) e nello stato corrente (per query veloci). Scelta deliberata contro l'event sourcing puro — che richiederebbe di ricostruire lo stato da zero per ogni query.

### Fase 3 — Event-Driven Services (2017+)

L'**event store centralizzato** funge da ponte: i moduli vengono separati in servizi indipendenti che producono e consumano eventi dallo store. La separazione è possibile perché i moduli erano già isolati internamente.

```
Service A ──publish──► Event Store ──stream──► Service B
                                   ──stream──► Service C (Risk Calculator)
                                   ──stream──► Service D (Fraud Detection)
```

---

## Sfide tecniche del dominio bancario

### Transazioni: il rischio dell'annotation-based approach

Il pattern `@Transactional` di Spring ha un vincolo non ovvio: funziona solo su metodi **pubblici** (invocati tramite proxy AOP). Se si estrae la logica transazionale in un metodo privato per ottimizzare, la transazione scompare silenziosamente.

```java
// SBAGLIATO: @Transactional su metodo privato — non ha effetto
public void transferMoney(...) {
    // validazioni, fraud checks...
    doTransfer(fromId, toId, amount); // ← @Transactional ignorato
}

@Transactional
private void doTransfer(...) { ... }

// CORRETTO: gestione esplicita della transazione
public void transferMoney(...) {
    transactionTemplate.execute(status -> {
        // logica transazionale esplicita
        return result;
    });
}
```

La soluzione adottata da Revolut: usare `TransactionTemplate` (o equivalente) per definire esplicitamente i confini della transazione — meno dichiarativo, ma senza magie AOP che possono fallire silenziosamente.

> "Rule number one: never lose money. Rule number two: never forget rule number one." — Warren Buffett (citato da Yatsenko per il principio di correttezza)

### Gestione delle valute: complessità nascosta

Le valute sembrano un dominio semplice (moltiplicazione per un tasso di cambio) ma nascondono edge case reali:

| Problema | Esempio |
|---|---|
| **Bid/Offer** | Ogni tasso ha due valori (acquisto/vendita) — molti sviluppatori li confondono |
| **Denomination events** | I governi ridenominano le valute (nuovi ISO code, vecchi e nuovi validi in parallelo) |
| **Pegged currencies** | IMP (Lira di Man): pegged 1:1 alla GBP, ISO code proprio, nessun tasso da Bloomberg |
| **Edge case accumulati** | La classe ExchangeRate: ~300 righe, >100 test solo per la gestione dei casi limite |

### Scrollable statements per batch processing

Per processare grandi volumi di dati (es. standing order da eseguire in una data specifica) senza caricare tutto in memoria, PostgreSQL supporta i **cursori** (scrollable statements):

```java
// Framework nasconde i dettagli, ma internamente:
// SELECT ... FROM standing_orders WHERE execution_date = ? -- scrollable cursor
// Processa un record alla volta, nessun batch manuale, nessun OOM
stream.forEach(order -> processOrder(order));
```

Alternativa a complesse pipeline di batch processing quando i dati devono essere processati sequenzialmente e i volumi sono elevati.

### Risk hedging in real-time

**Problema:** Revolut garantisce l'exchange rate istantaneamente al cliente, ma la transazione reale sul mercato avviene dopo. Si accumula rischio di currency exposure.

**Prima soluzione (non scalabile):** query sulla tabella transactions per calcolare la posizione di rischio aggregata — lenta con volumi crescenti, timeout inevitabili.

**Soluzione event-driven:** un servizio dedicato si sottoscrive a ogni evento di transazione finanziaria e aggiorna la posizione di rischio in real-time (latenza: ~100ms). Il finance team visualizza la posizione aggiornata e può hedgeare automaticamente sul mercato.

### Fraud detection — sistema "Sherlock"

**Constraint critico:** per le autorizzazioni carta, il tempo totale (cliente → POS → acquiring bank → Visa/Mastercard → Revolut → risposta) è ~5 secondi; la finestra interna per la decisione è **500ms**.

**Problema:** con 500ms non c'è tempo per calcolare score ML in real-time su tutta la storia transazionale.

**Soluzione pre-computation:**

```
Evento transazione precedente
    │
    ▼
ML model (Python, data science team)
    │ calcola score
    ▼
Event store
    │ stream
    ▼
Transaction Authorization Engine
    │ consulta score pre-calcolato
    │ (risposta in < 500ms)
    ▼
Decisione: autorizza / declina
```

Il sistema pre-calcola lo score di ogni utente aggiornandolo ad ogni evento. Quando arriva l'autorizzazione, lo score è già pronto. Il trade-off: lo score è basato sullo stato precedente alla transazione corrente, non su di essa.

**Risultato:** riduzione del fraud rate di ~40× rispetto agli standard di settore.

---

## Principi ingegneristici

### TDD come requisito non negoziabile

Non "avere test", ma **test-first**: scrivere il test che descrive il problema prima di implementare. La distinzione è fondamentale per la qualità.

**Abolizione dei QA tradizionali:** i tester dedicati sono stati sostituiti da "software engineers in test" — ingegneri che costruiscono framework, tooling e infrastruttura per permettere agli altri di scrivere test efficienti. La responsabilità della qualità è degli ingegneri che scrivono il codice.

### Problem-driven, non solution-driven

Il criterio per ogni decisione tecnica (framework, tecnologia, approccio architetturale) è: **risolve il problema definito?** Non: "è la tecnologia più recente?" o "preferisco questo pattern?".

Vantaggi: elimina le "holy wars" tra ingegneri e focalizza il code review sul problema invece che sullo stile.

### Peer review focalizzata sul problema

Il pull request è una specifica del problema da risolvere. I reviewer si aspettano di capire il problema prima di guardare il codice. Il review è sul problema + soluzione, non sulla sintassi (risolta da convenzioni e linter).

### Technical debt come requisito evolutivo

Il debito tecnico non viene tracciato separatamente: quando raggiunge un livello che impatta la capacità di scalare, diventa un nuovo requisito di business. La decisione (scale out vs refactoring) è guidata dal problema concreto, non da astratte metriche di "pulizia".

---

## Scelte tecnologiche

| Tecnologia | Ruolo | Motivazione |
|---|---|---|
| **PostgreSQL** | Storage principale | ACID obbligatorio; replication/backup maturi |
| **jOOQ** | SQL type-safe | Errori SQL rilevati a compile-time; zero errori di sintassi in produzione |
| **Java** | Backend principale | Team skillset; ecosistema JVM |
| **Python** | ML/Data Science | Best fit per modeling; integrato via events |
| **Kotlin** | Android | Migrazione da Java 6 per ridurre technical debt mobile |
| **React** | Frontend | Migrazione da Angular (2016) |
| **Event store custom** | Async backbone | Disaccoppia i servizi; abilita replay e audit |

---

## Citazioni notevoli

> "Correctness is number one in finance. You should never lose your customers' money. All money is virtual — it's just numbers in some database."  
> — Vlad Yatsenko

> "We are problem-driven, not solution-driven. When you define a problem well, the differences in solution become less important — holy wars don't happen."  
> — Vlad Yatsenko

> "TDD doesn't mean having tests. It means test-first: write the test that describes the problem, then implement."  
> — Vlad Yatsenko

> "Once you start using jOOQ, you never have mistakes in your SQL statements."  
> — Vlad Yatsenko

---

## Riferimenti e risorse

- **jOOQ** ([jooq.org](https://www.jooq.org)) — libreria Java per SQL type-safe; citata come scelta fondamentale per eliminare errori SQL in produzione
- **PostgreSQL scrollable cursors** — `ResultSet.TYPE_SCROLL_INSENSITIVE` / `SCROLL` in JDBC; per streaming di grandi dataset senza caricare tutto in memoria
- **Revolut "Sherlock"** — sistema interno di fraud detection basato su ML pre-computation
- **Spring `TransactionTemplate`** — alternativa esplicita a `@Transactional` per evitare il pitfall dei metodi privati con Spring AOP
- **New Relic** — monitoring scelto nella fase MVP per zero effort di setup su applicazioni Java
