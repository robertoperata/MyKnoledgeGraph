---
title: Concorrenza moderna in Java 21+ — Virtual Thread, Structured Concurrency e CompletableFuture
type: synthesis
tags:
  - thread2-java
  - thread1-microservices
sources:
  - "[[structured-concurrency]]"
  - "[[virtual-threads]]"
  - "[[completable-future]]"
  - "[[java-concurrency]]"
  - "[[garbage-collection]]"
updated: 2026-04-18
related:
  - "[[concepts/structured-concurrency]]"
  - "[[concepts/virtual-threads]]"
  - "[[concepts/completable-future]]"
  - "[[concepts/java-concurrency]]"
  - "[[concepts/garbage-collection]]"
---

# Domanda originale

**Come cambia la programmazione concorrente in Java 21+ con Structured Concurrency e Virtual Thread: quando rimpiazzano CompletableFuture e quando no?**

---

# Risposta sintetica

Java 21+ offre tre strumenti distinti che non sono varianti dello stesso problema: risolvono problemi diversi. La confusione nasce dal fatto che tutti e tre possono essere usati per "fare cose in parallelo", ma le loro garanzie, i modelli mentali e i casi d'uso ottimali sono differenti. La regola empirica: **Virtual Thread per il blocking I/O, Structured Concurrency per task concorrenti con ciclo di vita condiviso, CompletableFuture per l'aggregazione parallela di risultati.**

---

## I tre strumenti a confronto

| | Virtual Thread | Structured Concurrency | CompletableFuture |
|---|---|---|---|
| **Problema che risolve** | Scalabilità con blocking I/O | Ciclo di vita di task concorrenti | Composizione asincrona |
| **Modello mentale** | Codice sincrono che scala | Task padre/figlio con scope | Pipeline di trasformazioni |
| **Cancellazione** | Manuale | Automatica (scope guarantee) | Manuale (esponenziale con N) |
| **Thread leak** | Possibile | Impossibile | Possibile |
| **Stack trace** | Coerente | Coerente | Frammentato |
| **Java version** | GA Java 21 | Preview Java 21, stabile Java 25 |Stabile da Java 8 |
| **Caso d'uso ideale** | HTTP server, Kafka consumer | Fan-out con aggregazione | Parallelizzare N chiamate indipendenti |

---

## Virtual Thread: scalabilità senza riscrittura

I Virtual Thread non cambiano il modello di programmazione — cambiano la *scala* a cui funziona il codice bloccante classico.

```java
// Prima: pool di 200 thread → max 200 richieste concorrenti
// Dopo: virtual thread → migliaia di richieste concorrenti
spring.threads.virtual.enabled: true  // una riga
```

**Quando usarli:**
- Qualsiasi workload IO-intensive (HTTP, JDBC, Kafka, file I/O)
- Quando si vuole scala senza riscrivere in stile reattivo

**Quando NON usarli:**
- Workload CPU-intensive (calcolo, parallel stream) → platform thread + Fork/Join
- Sezioni critiche con `synchronized` e operazioni bloccanti → pinning issue

**Il gotcha del pinning:** un virtual thread non si smonta dal carrier thread dentro un blocco `synchronized`. Soluzione: `ReentrantLock` per sezioni critiche con I/O.

**Impatto sul GC:** milioni di VT ma pochi carrier thread → meno GC roots → pause GC più brevi. Con 200 richieste HTTP concorrenti: 200 platform thread da scansionare vs ~8 carrier thread.

---

## Structured Concurrency: task con ciclo di vita condiviso

La Structured Concurrency risolve il problema che CompletableFuture lascia aperto: *cosa succede agli altri task quando uno fallisce?*

```java
// ShutdownOnFailure: se uno fallisce, cancella gli altri automaticamente
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    var vectorSearch = scope.fork(this::performVectorSearch);
    var classicSearch = scope.fork(this::performClassicSearch);
    scope.join();
    return combine(vectorSearch.get(), classicSearch.get());
}
// Se vectorSearch lancia eccezione → classicSearch cancellato automaticamente
// Nessun thread leak, stack trace coerente
```

**Quando usarla:**
- Fan-out con N subtask che devono tutti completare (o tutti fallire insieme)
- Quando la cancellazione reciproca è un requisito
- Quando la leggibilità e il debugging sono prioritari

**Quando NON usarla:**
- Ancora in preview/non stabile fino a Java 25 → valutare in base alla versione in produzione
- Per pipeline di trasformazione senza ciclo di vita condiviso → CompletableFuture è più espressivo

**Integrazione con Virtual Thread:** i subtask della SC girano su virtual thread internamente. I due meccanismi si combinano: scala (VT) + ciclo di vita garantito (SC).

---

## CompletableFuture: aggregazione parallela e composizione

CompletableFuture non è obsoleto: rimane lo strumento migliore per la **composizione dichiarativa** di operazioni asincrone indipendenti.

```java
// Aggregazione parallela: tempo totale = max(A, B), non A + B
CompletableFuture<Order> orderFuture =
    CompletableFuture.supplyAsync(() -> orderService.get(id));
CompletableFuture<Customer> customerFuture =
    CompletableFuture.supplyAsync(() -> customerService.get(id));

CompletableFuture.allOf(orderFuture, customerFuture).join();
return compose(orderFuture.get(), customerFuture.get());
```

**Quando usarlo ancora:**
- `allOf()` / `anyOf()` per aggregare N future indipendenti
- Pipeline di trasformazione con `thenApply`, `thenCompose`, `handle`
- Integrazione con API esistenti che restituiscono `Future<T>`
- Pattern Request-Reply con Correlation ID (la mappa `correlationId → Future` rimane CF)

**Quando preferire le alternative:**
- Se N task devono cancellarsi a vicenda → Structured Concurrency
- Se il blocking semplice è sufficiente → codice sincrono su Virtual Thread

---

## Guida pratica alla scelta

```
Ho operazioni I/O bloccanti e voglio scala?
    → Virtual Thread (spring.threads.virtual.enabled: true)

Ho N task concorrenti che devono finire tutti o cancellarsi tutti?
    → Structured Concurrency (StructuredTaskScope.ShutdownOnFailure)

Ho N chiamate indipendenti e voglio il risultato più veloce possibile?
    → CompletableFuture.anyOf()

Ho N chiamate indipendenti e ho bisogno di tutti i risultati?
    → CompletableFuture.allOf() oppure Structured Concurrency

Ho una pipeline di trasformazioni asincrone concatenate?
    → CompletableFuture (thenApply/thenCompose)

Ho workload CPU-intensive?
    → Platform thread + parallel stream / Fork/Join
```

---

## Sorgenti utilizzate

- [[concepts/structured-concurrency]] — SC come successore moderno di CF per task concorrenti
- [[concepts/virtual-threads]] — VT per scala con codice bloccante
- [[concepts/completable-future]] — aggregazione parallela e composizione asincrona
- [[concepts/java-concurrency]] — Executor Framework, modello a task
- [[concepts/garbage-collection]] — impatto di VT sulle pause GC

---

## Lacune / Argomenti da approfondire

- Scoped Values (Java 21+): alternativa thread-safe a ThreadLocal per virtual thread
- `StructuredTaskScope` subtypes avanzati: `ShutdownOnSuccess`, scope custom
- Gatherers API (Java 24): `mapConcurrent` come alternativa a `parallelStream` con virtual thread
- Reactive Programming (WebFlux/Reactor): quando rimane preferibile ai virtual thread
- Testing di codice concorrente: strumenti per rilevare race condition (jcstress, ThreadSanitizer)
- Profiling e monitoring dei virtual thread: JFR, async-profiler con VT
- Pinning detection automatica: `-Djdk.tracePinnedThreads=full`
- `SequencedCollection` e nuove API Collections Java 21+
