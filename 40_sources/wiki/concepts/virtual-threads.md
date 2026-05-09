---
title: Virtual Threads
type: concept
tags: [thread2-java, thread1-microservices]
sources:
  - "[[Java Threads Basics]]"
  - "[[Java Threads Demystified]]"
  - "[[microservizi_pattern_summary]]"
  - "[[devoxx_modern-java-playful-way]]"
  - "[[youtube_java-25-lts-features-jchampions]]"
updated: 2026-05-08
related:
  - "[[concepts/java-concurrency]]"
  - "[[concepts/completable-future]]"
  - "[[concepts/structured-concurrency]]"
  - "[[patterns/request-reply-correlation-id]]"
---

# Virtual Threads (Java 21+)

## Definizione

I Virtual Thread sono thread gestiti dalla JVM, non dall'OS. Sono leggerissimi (pochi KB vs ~1MB dei platform thread), si possono creare a milioni, e quando si bloccano su un'operazione di attesa vengono **smontati** dal platform thread sottostante ("carrier thread"), che diventa libero di eseguire altri virtual thread.

## Motivazione

La programmazione reattiva (WebFlux/Reactor) risolve il problema del blocking I/O ma introduce un modello di programmazione complesso (pipeline dichiarative, stack trace difficili). I virtual thread offrono la stessa scalabilità con il **codice imperativo classico**.

## Confronto Platform vs Virtual Thread

| Aspetto | Platform Thread | Virtual Thread |
|---|---|---|
| Stack size | ~1MB | ~pochi KB |
| Costo di creazione | ~1ms | ~microsecondi |
| Numero massimo | ~migliaia | ~milioni |
| Scheduling | OS preemptive | JVM non-preemptive |
| Uso ideale | CPU-intensive | IO-intensive |

## Come usarli

```java
// Creazione diretta
Thread vt = Thread.ofVirtual().start(() -> task());

// Via executor (preferito)
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(task);
}

// Spring Boot 3.2+ — una riga nel config
spring.threads.virtual.enabled: true
```

## Pinning — stato aggiornato (Java 24/25)

Un virtual thread non può smontarsi dal carrier thread in due casi:

| Causa | Stato | Raccomandazione |
|---|---|---|
| `synchronized` block/method | ✅ **Risolto in Java 24** | Nessuna limitazione dal Java 24 |
| Native method call (JNI) | ❌ Non risolvibile | Evitare chiamate native bloccanti nei virtual thread; probabilmente non sarà mai risolto |

**Prima di Java 24**: era necessario sostituire `synchronized` con `ReentrantLock` nelle sezioni critiche che contenevano operazioni bloccanti. Dal Java 24 questa sostituzione non è più necessaria per il pinning — `synchronized` funziona correttamente con i virtual thread.

```java
// Dal Java 24: synchronized funziona correttamente
synchronized (this) { future.get(); }  // ✅ nessun pinning

// Alternativa ancora valida (lock espliciti)
lock.lock();
try { future.get(); } finally { lock.unlock(); }
```

> "Virtual threads are now quite reliable — synchronized pinning is fixed in Java 24." — Ario (JChampions Conference 2026)

## Connessione con Microservizi

Il blog post sui microservizi (microservizi_pattern_summary) mostra come abilitare i virtual thread nel pattern **Request-Reply con Correlation ID**:
- Con platform thread: `future.get()` blocca il thread Tomcat → pool di 200 thread → max ~200 richieste concorrenti
- Con virtual thread: `future.get()` smonta il virtual thread dal carrier → migliaia di richieste concorrenti con pochissimi carrier thread
- Il codice non cambia rispetto al platform thread — basta una riga di configurazione

> **Tensione:** I virtual thread non sono la soluzione per tutto. Per workload CPU-intensive (calcolo, stream paralleli), i platform thread e il Fork/Join framework sono ancora la scelta corretta.

## Stream Gatherers con mapConcurrent (Java 24)

```java
// Evitare: parallelStream rischia di esaurire il common ForkJoinPool
results = queries.parallelStream().map(this::performSearch).collect(toList());

// Preferire: mapConcurrent usa virtual thread internamente
results = queries.stream()
    .gather(Gatherers.mapConcurrent(5, this::performSearch))
    .collect(toList());
```

`mapConcurrent` usa virtual thread internamente: sicuro in produzione, nessun rischio di esaurire il ForkJoinPool.

## Connessioni

- [[concepts/java-concurrency]] — i virtual thread sono un'evoluzione del modello di threading Java
- [[concepts/completable-future]] — usato insieme ai virtual thread nel pattern Request-Reply
- [[concepts/structured-concurrency]] — la Structured Concurrency usa virtual thread per i subtask
- [[patterns/request-reply-correlation-id]] — virtual thread abilitano alta concorrenza con codice sincrono semplice
- [[concepts/scope-values]] — ScopeValue è il contesto immutabile propagato ai virtual thread; sostituzione di ThreadLocal
