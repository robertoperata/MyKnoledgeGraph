---
title: Java Concurrency
type: concept
tags: [thread2-java]
sources:
  - "[[Java Threads Basics]]"
  - "[[Java Threads Demystified]]"
  - "[[Concurrent Programming Core Concepts]]"
updated: 2026-04-09
related:
  - "[[concepts/virtual-threads]]"
  - "[[concepts/completable-future]]"
  - "[[concepts/java-memory-model]]"
---

# Java Concurrency

## Definizione

La concurrency in Java permette a più task di fare progresso in periodi di tempo sovrapposti. Non è la stessa cosa della parallelism (stessi task su dati diversi in contemporanea). La concurrency riguarda la struttura del programma; la parallelism riguarda l'esecuzione.

## Rischi fondamentali

| Rischio | Descrizione |
|---|---|
| **Race condition** | Il risultato dipende dal timing relativo di più thread |
| **Deadlock** | Due o più thread si bloccano a vicenda aspettando lock che l'altro tiene |
| **Livelock** | I thread non sono bloccati ma non fanno progresso (si rispondono a vicenda) |
| **Starvation** | Un thread è privato permanentemente delle risorse di cui ha bisogno |

## Strategie di thread-safety

### 1. Immutability
Gli oggetti immutabili sono sempre thread-safe. In Java: `private final`, nessun setter, accesso esclusivo ai componenti mutabili.

> [!tip] "Immutability is the simplest way to achieve thread safety." — Maurice Naftalin

### 2. Confinement
- **Stack confinement**: accessibile solo tramite variabili locali
- **Thread confinement**: accessibile solo da un thread alla volta
- **ThreadLocal**: storage per-thread

### 3. Synchronization
- **`synchronized`**: intrinsic lock, reentrant, visibilità e mutual exclusion
- **`volatile`**: visibilità senza mutual exclusion
- **`java.util.concurrent.locks`**: `ReentrantLock`, `ReadWriteLock`, `StampedLock` — più flessibili ma richiedono unlock esplicito in finally

### 4. Atomic Variables
`AtomicInteger`, `AtomicReference`, `LongAdder` — operazioni atomiche lock-free via CAS (Compare-And-Swap).

## Executor Framework

**Non pensare in termini di thread, pensa in termini di task.** Un task è un'unità di lavoro indipendente (es. una richiesta HTTP da un client).

```java
ExecutorService service = Executors.newFixedThreadPool(4);
Future<Result> future = service.submit(callable);
```

Factory methods principali:
- `newFixedThreadPool(n)`: pool fisso
- `newCachedThreadPool()`: cresce e si riduce con la domanda
- `newVirtualThreadPerTaskExecutor()`: virtual thread per ogni task (Java 21+)

## Concurrent Collections

- **`ConcurrentHashMap`**: letture concorrenti, operazioni atomiche (`computeIfAbsent`, `merge`), no client-side locking necessario
- **`CopyOnWriteArrayList/Set`**: ottimo per read-heavy, costoso in scrittura
- **`BlockingQueue`** (ArrayBlockingQueue, LinkedBlockingQueue): producer-consumer pattern, throttling naturale

## Synchronizers

| Synchronizer | Uso |
|---|---|
| `Semaphore` | Pool di risorse finite |
| `CountDownLatch` | Aspettare N eventi (one-shot) |
| `CyclicBarrier` | Aspettare che tutti i thread si sincronizzino (reusable) |
| `Phaser` | Variazione flessibile di CyclicBarrier |

## Fork/Join Framework

Decompone ricorsivamente un task grande in subtask più piccoli. Usa **work stealing**: i thread idle rubano task dalle code dei thread occupati.

Più facile da usare tramite **parallel streams**:
```java
list.parallelStream().filter(...).map(...).sum();
```
Condizioni per un uso efficace: workload CPU-intensive, ~50µs+ per elemento, sorgente splittabile (ArrayList sì, LinkedList no).

## Connessioni

- [[concepts/virtual-threads]] — alternativa ai platform thread per workload IO-intensive (Java 21+)
- [[concepts/completable-future]] — API per comporre operazioni asincrone
- [[concepts/java-memory-model]] — le garanzie di visibilità che rendono possibile la concurrency sicura
- [[concepts/garbage-collection]] — GC hints: più thread = più GC roots = pause più lunghe
- [[synthesis/microservizi-java-aggregazione]] — CompletableFuture usato nel pattern Request-Reply con Correlation ID
