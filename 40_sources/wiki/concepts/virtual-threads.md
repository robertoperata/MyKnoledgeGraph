---
title: Virtual Threads
type: concept
tags: [thread2-java, thread1-microservices]
sources:
  - "[[Java Threads Basics]]"
  - "[[Java Threads Demystified]]"
  - "[[microservizi_pattern_summary]]"
updated: 2026-04-09
related:
  - "[[concepts/java-concurrency]]"
  - "[[concepts/completable-future]]"
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

## Pinning — il gotcha principale

Un virtual thread non può smontarsi dal carrier thread quando:
- È dentro un blocco `synchronized`
- Sta eseguendo un metodo native

**Conseguenza**: il carrier thread si blocca, vanificando il vantaggio dei virtual thread.

**Soluzione**: usare `ReentrantLock` invece di `synchronized` per sezioni critiche che contengono operazioni bloccanti.

```java
// DA EVITARE con virtual threads
synchronized (this) { future.get(); }  // pinning!

// PREFERIRE
lock.lock();
try { future.get(); } finally { lock.unlock(); }
```

## Connessione con Microservizi

Il blog post sui microservizi (microservizi_pattern_summary) mostra come abilitare i virtual thread nel pattern **Request-Reply con Correlation ID**:
- Con platform thread: `future.get()` blocca il thread Tomcat → pool di 200 thread → max ~200 richieste concorrenti
- Con virtual thread: `future.get()` smonta il virtual thread dal carrier → migliaia di richieste concorrenti con pochissimi carrier thread
- Il codice non cambia rispetto al platform thread — basta una riga di configurazione

> **Tensione:** I virtual thread non sono la soluzione per tutto. Per workload CPU-intensive (calcolo, stream paralleli), i platform thread e il Fork/Join framework sono ancora la scelta corretta.

## Connessioni

- [[concepts/java-concurrency]] — i virtual thread sono un'evoluzione del modello di threading Java
- [[concepts/completable-future]] — usato insieme ai virtual thread nel pattern Request-Reply
- [[patterns/request-reply-correlation-id]] — virtual thread abilitano alta concorrenza con codice sincrono semplice
