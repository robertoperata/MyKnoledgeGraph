---
title: Structured Concurrency
type: concept
tags: [thread2-java]
sources:
  - "[[devoxx_modern-java-playful-way]]"
  - "[[Java Threads Basics]]"
updated: 2026-04-17
related:
  - "[[concepts/virtual-threads]]"
  - "[[concepts/completable-future]]"
  - "[[concepts/java-concurrency]]"
---

# Structured Concurrency

## Definizione

La Structured Concurrency (preview da Java 21, revisionata in Java 25) è un modello di programmazione concorrente che tratta un insieme di task concorrenti come una singola unità di lavoro. I task figli non possono sopravvivere al loro scope padre: quando lo scope termina (normalmente o per errore), tutti i subtask vengono cancellati automaticamente.

## Motivazione

`CompletableFuture` risolve la composizione asincrona ma introduce problemi reali a scala:
- Con N future la gestione della cancellazione reciproca cresce esponenzialmente
- Thread leak: se un future fallisce, gli altri continuano a girare
- Stack trace illeggibili (la logica è spezzata tra callback)
- Non scala mentalmente: pensiero reattivo vs pensiero sequenziale

## Come funziona

```java
// Java 24 — ShutdownOnFailure
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    var vectorSubtask = scope.fork(this::performVectorSearch);
    var classicSubtask = scope.fork(this::performClassicSearch);
    scope.join();  // aspetta entrambi
    return combine(vectorSubtask.get(), classicSubtask.get());
}
// Se un subtask fallisce → l'altro viene cancellato automaticamente

// Java 25 — configurabile
try (var scope = StructuredTaskScope.open()) {
    // timeout, thread factory custom, naming per debugger/JFR
}
```

## Vantaggi rispetto a CompletableFuture

| Aspetto | CompletableFuture | Structured Concurrency |
|---|---|---|
| Cancellazione | Esplicita, esponenziale con N task | Automatica |
| Thread leak | Possibile | Impossibile (scope guarantee) |
| Leggibilità | Pipeline callback | Codice sincrono lineare |
| Debug | Stack trace frammentati | Stack trace coerenti |
| Pensiero | Reattivo | Sequenziale |

> *"Completable Future has no future."* — Dr. Vena (citato a Devoxx Poland 2025)

## Integrazione con Virtual Threads

Structured Concurrency usa i virtual thread internamente. Combinati:
- I subtask girano su virtual thread → overhead minimo
- Lo scope garantisce il ciclo di vita → nessun leak
- Il codice sembra sincrono → manutenibile

## Connessioni

- [[concepts/virtual-threads]] — i subtask girano su virtual thread
- [[concepts/completable-future]] — la SC è il successore moderno di CF per task concorrenti
- [[concepts/java-concurrency]] — evoluzione del modello di concorrenza Java
