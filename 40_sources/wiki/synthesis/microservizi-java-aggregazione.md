---
title: Microservizi e Java — Aggregazione Dati ad Alta Concorrenza
type: synthesis
tags: [thread1-microservices, thread2-java]
sources:
  - "[[microservizi_pattern_summary]]"
  - "[[Java Threads Basics]]"
  - "[[Java Threads Demystified]]"
  - "[[Microservices Collaboration]]"
updated: 2026-04-09
related:
  - "[[patterns/request-reply-correlation-id]]"
  - "[[patterns/cqrs-read-model]]"
  - "[[concepts/completable-future]]"
  - "[[concepts/virtual-threads]]"
---

# Domanda originale

**Come si applica il CQRS e il Request-Reply in un sistema Java ad alta concorrenza? Quale impatto hanno Virtual Thread e CompletableFuture sulla scalabilità dei microservizi che aggregano dati da più domini?**

---

# Risposta sintetica

Due sorgenti distinte — il blog post sui microservizi e i corsi Java — convergono su una risposta coerente: la scelta del pattern di aggregazione (CQRS vs Request-Reply) e la scelta dell'implementazione concorrente (platform thread, virtual thread, WebFlux) sono decisioni ortogonali ma strettamente connesse.

## Il problema dell'aggregazione

Un microservizio che riceve una richiesta HTTP e deve aggregare dati da altri servizi si trova di fronte a due domande:

1. **Dove vivono i dati?** (pattern di aggregazione)
2. **Come gestisco la concorrenza durante l'attesa?** (modello di threading)

---

## Asse 1 — Pattern di aggregazione

| Pattern | Consistenza | Latenza runtime | Scenario |
|---|---|---|---|
| Request-Reply + Correlation ID | Forte | Alta (dipende dai servizi) | Dati freschi obbligatori |
| CQRS + Read Model | Eventuale | Zero (lettura locale) | Letture frequenti |
| API Composition / BFF | Forte | Media (REST parallelo) | Aggregazione semplice |

La scelta dipende principalmente da: quanto devono essere freschi i dati e quanto può aspettare il chiamante.

---

## Asse 2 — Implementazione concorrente

### CompletableFuture: aggregazione parallela
`CompletableFuture.allOf(futureB, futureC)` permette di parallelizzare chiamate indipendenti:
```java
// Tempo totale = max(latenzaB, latenzaC), non la somma
CompletableFuture.allOf(futureB, futureC).get(3, TimeUnit.SECONDS);
```

Usato sia nel Request-Reply (dove i future vengono completati dal Kafka consumer) che nel BFF puro (dove le chiamate REST sono parallele).

### Virtual Thread: scalabilità senza riscrittura
Con i platform thread, `future.get()` blocca un thread OS → pool da 200 thread → max ~200 richieste concorrenti.

Con i virtual thread, `future.get()` smonta il VT dal carrier thread → il carrier è libero per altri VT → migliaia di richieste concorrenti.

```yaml
spring.threads.virtual.enabled: true  # una riga
```
Il codice non cambia. Il vantaggio è enorme per workload IO-intensive (che è esattamente ciò che è il Request-Reply via Kafka).

### Pinning — il gotcha che connette i due mondi
I virtual thread si bloccano (pinning) se eseguono `future.get()` dentro un blocco `synchronized`. Nel codice del Request-Reply con Correlation ID, la mappa `correlationId → Future` deve essere thread-safe. Se si usa `Collections.synchronizedMap()` (che usa `synchronized`), si rischia il pinning.

Soluzione: usare `ConcurrentHashMap` (che non usa `synchronized` per i read-path) o `ReentrantLock` per le sezioni critiche.

---

## La connessione trasversale

```
Java Threads Basics         ←→    microservizi_pattern_summary
CompletableFuture.allOf()   ←→    aggregazione parallela nel Request-Reply
Virtual Threads             ←→    alta concorrenza senza WebFlux
ConcurrentHashMap           ←→    mappa correlationId → Future thread-safe
Garbage Collection          ←→    carrier threads = meno GC roots
```

---

## Conclusione

Per un sistema Java a microservizi ad alta concorrenza:

1. **Se i dati devono essere freschi**: usa Request-Reply + Correlation ID + CompletableFuture per la parallelizzazione + Virtual Thread per la scalabilità
2. **Se i dati possono essere eventualmente consistenti**: usa CQRS + Read Model — azzera la latenza runtime e rimuove il problema della concorrenza dell'attesa
3. **In sistemi reali**: i due pattern coesistono. CQRS per la maggior parte dei dati, Request-Reply per i pochi casi di strong consistency obbligatoria

---

## Sorgenti utilizzate

- [[microservizi_pattern_summary]] — pattern di aggregazione e implementazioni concorrenti in Java
- [[Java Threads Basics]] — CompletableFuture, Virtual Thread, ConcurrentHashMap
- [[Java Threads Demystified]] — thread safety, confinement, pooling
- [[Microservices Collaboration]] — temporal coupling, broker-based request/response

## Lacune emerse

- **Kafka Streams e KTable**: la nota in `The Essential Learning Foundations.md` sulla join di topic non è coperta dalla knowledge base
- **Testing di codice concorrente con Kafka**: come testare il pattern Request-Reply + Correlation ID (race conditions nella mappa)
- **Observability**: come tracciare i virtual thread attraverso i microservizi (OpenTelemetry + virtual threads)
