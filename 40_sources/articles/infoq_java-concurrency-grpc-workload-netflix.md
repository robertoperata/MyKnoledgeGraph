---
tags:
  - java
  - concurrency
  - performance
  - distributed-systems
type: article
author: Hugo Marques
source: https://www.infoq.com/presentations/java-grpc-workload/
date: 2026-01-12
---

# Java Concurrency from the Trenches: Lessons Learned in the Wild

## Sunto

Hugo Marques, software engineer di Netflix, presenta le lezioni apprese dall'implementazione di sistemi Java altamente concorrenti in produzione. Il caso di studio è un job batch che gira ogni 30 minuti su ~10.000 ordini per 10 regioni globali, con ~5.000 prodotti per ordine, che effettua chiamate gRPC raggruppate per 100 prodotti — per un totale di circa 270.000 RPS (o 2.700 batch/secondo) al picco.

L'architettura iniziale è lineare: Job → Regioni → Order Service → Prodotti → gRPC Service → Output su file. Il problema è che la semplice parallelizzazione a ogni livello della gerarchia produce comportamenti inaspettati, out-of-memory, e rischia di saturare i servizi downstream. La presentazione documenta **8 problemi distinti** affrontati in sequenza, con relative soluzioni — un percorso iterativo realistico verso un sistema stabile.

Il messaggio centrale è pragmatico: prima di ottimizzare, capire **cosa si sta proteggendo** (memoria propria, CPU, o servizi downstream), poi scegliere lo strumento adatto tra semafori e rate limiter, e privilegiare **semplicità** rispetto a eleganza architettonica. Le metriche reali guidano ogni decisione — niente ottimizzazione speculativa.

L'ultimo tratto della presentazione esplora i **Virtual Thread di Java 21/24**, che promettono di eliminare buona parte della gestione manuale dei thread pool per workload I/O-bound, ma con la cautela di evitare la creazione annidata e greedy di migliaia di thread virtuali contemporaneamente.

---

## I 8 Problemi Documentati

| # | Problema | Causa | Soluzione |
|---|---|---|---|
| 1 | Performance degradata | Parallel streams annidati a più livelli | Parallelizzare solo dove conta (sorgente dati) |
| 2 | Parallel stream silenziosamente sequenziale | OpenJDK serializza i parallel stream dentro `flatMap` | Parallelizzare al loop esterno |
| 3 | Thread context perso | Thread-local non propagato nel ForkJoinPool | Context propagation esplicita o executor custom |
| 4 | OOM (prima istanza) | Task asincroni request + response sullo stesso executor | Executor separati per request e response |
| 5 | DDoS dei servizi downstream | Nessun limite alle chiamate concorrenti (30K RPS) | Semaforo + Rate Limiter |
| 6 | OOM (seconda istanza) | Rate limiter da solo non blocca l'accumulo in coda | Semaforo alla creazione task + rate limiter downstream |
| 7 | Virtual Thread pinning | `synchronized` blocca il carrier thread | Migrazione a `ReentrantLock`; risolto definitivamente in Java 24 |
| 8 | OOM con Virtual Thread | Creazione annidata: 10K ordini × 50 prodotti = 500K+ thread | Semaforo che controlla il rate di creazione dei virtual thread |

---

## Esempi pratici

### FlatMap serializza silenziosamente i parallel stream

```java
// Questo gira sequenzialmente, NON in parallelo
stream.flatMap(item -> item.parallelStream())
```

### Thread-local perso in parallel stream

```java
context.set("user-123");
stream.parallel().forEach(item -> {
    // context.get() restituisce null nei worker thread del ForkJoinPool
});
```

### Le tre varianti di thenApplyAsync con CompletableFuture

```java
// Corre sul thread dello stage precedente (più veloce, meno flessibile)
.thenApply(fn)

// Corre sul ForkJoinPool comune
.thenApplyAsync(fn)

// Corre sull'executor specificato (massima flessibilità)
.thenApplyAsync(fn, executor)
```

### Executor separati per request e response (soluzione OOM #1)

```java
CompletableFuture.supplyAsync(request, requestExecutor)
    .thenApply(response)           // usa l'executor gRPC di default
    .thenApplyAsync(process, responseExecutor);
```

### Semaforo per proteggere le proprie risorse

```java
Semaphore semaphore = new Semaphore(50);
semaphore.acquire();
try {
    // esegui task
} finally {
    semaphore.release();
}
```

### Rate Limiter per proteggere i servizi downstream

```java
RateLimiter limiter = RateLimiter.create(3000.0);  // 3K RPS
```

> **Distinzione chiave:** il semaforo protegge le proprie risorse (memoria, CPU); il rate limiter protegge i servizi downstream.

### Bounded executor con coda limitata

```java
BlockingQueue<Runnable> queue = new LinkedBlockingQueue<>(queueSize);
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    coreThreads, maxThreads,
    timeout, unit, queue
);
```

### Virtual Thread con semaforo (creazione controllata)

```java
Semaphore semaphore = new Semaphore(50);
semaphore.acquire();
executor.supplyAsync(() -> {
    try {
        // lavoro del virtual thread
    } finally {
        semaphore.release();
    }
}, virtualThreadExecutor);
```

### Virtual Thread ottimizzati: elaborazione sequenziale dentro il thread

```java
// Invece di creare un virtual thread per ogni prodotto:
@Async
void processOrderBlock(Order order) {
    // elaborazione sequenziale dei prodotti DENTRO il virtual thread
    order.products.forEach(product -> {
        grpcCall(product);  // chiamate sequenziali nello stesso thread
    });
}
```

Risultati a confronto:

| Approccio | Tempo totale | RPS picco | Complessità |
|---|---|---|---|
| Bounded executor | ~12 min | 30K → 5K steady | Media |
| Virtual Thread + semaforo annidato | ~12 min | 16K picco | Alta |
| Virtual Thread + semaforo ottimizzato | ~13 min | 13K picco | **Bassa** |

> L'approccio più semplice (un solo semaforo come "leva di controllo") è il vincitore pratico: più facile da debuggare e spiegare al team.

---

## Link esterni

- [ExecutorService API (Java 8)](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ExecutorService.html) — documentazione ufficiale
- [CompletableFuture API (Java 8)](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html) — documentazione ufficiale
- [JEP 444 – Virtual Threads](https://openjdk.org/jeps/444) — proposta originale dei virtual thread in Java 21

---

## Immagini

Nessuna immagine presente
