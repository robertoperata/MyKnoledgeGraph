---
title: Java Memory Model
type: concept
tags: [thread2-java]
sources:
  - "[[Java Threads Basics]]"
  - "[[Java Threads Demystified]]"
updated: 2026-04-09
related:
  - "[[concepts/java-concurrency]]"
  - "[[concepts/garbage-collection]]"
---

# Java Memory Model (JMM)

## Definizione

Il Java Memory Model (JSR 133) definisce quando le scritture di un thread sono garantite visibili alle letture di un altro. Senza le garanzie del JMM, le ottimizzazioni di CPU e compiler (cache, riordino istruzioni) possono portare a comportamenti inattesi in codice multi-thread.

## Problemi hardware che il JMM risolve

1. **CPU Caches**: ogni core CPU ha la propria cache. Modifiche a un valore possono non essere visibili ad altri core.
2. **Instruction Reordering**: compiler e CPU possono riordinare le istruzioni per ottimizzare. Il riordino è sicuro dal punto di vista di un singolo thread ma può violare assunzioni in contesti multi-thread.
3. **Word Tearing**: i valori a 64 bit (`long`, `double`) possono essere scritti in due operazioni da 32 bit da thread diversi (solo su variabili non-volatile).

## Happens-Before Relationship

La relazione happens-before garantisce che un'azione A sia visibile a B se A happens-before B.

Regole principali:
- **Program Order**: ogni azione in un thread happens-before ogni azione successiva nello stesso thread
- **Monitor Lock**: l'unlock happens-before ogni successivo lock sullo stesso monitor
- **Volatile**: una scrittura su campo volatile happens-before ogni successiva lettura dello stesso campo
- **Thread Start**: `Thread.start()` happens-before ogni azione nel thread avviato
- **Thread Join**: tutte le azioni in un thread happens-before il ritorno di `join()` in un altro thread
- **Transitività**: se A happens-before B e B happens-before C, allora A happens-before C

## `synchronized` vs `volatile`

| | `synchronized` | `volatile` |
|---|---|---|
| Visibilità | ✅ (via lock/unlock) | ✅ |
| Mutual exclusion | ✅ | ❌ |
| Atomicità composta | ✅ | ❌ |
| Uso tipico | Sezioni critiche con più operazioni | Flag, riferimenti condivisi semplici |

```java
// volatile: visibilità sì, ma i++ non è atomico!
private volatile int counter = 0;
counter++;  // race condition: lettura + incremento + scrittura non sono atomiche

// synchronized: visibilità + mutual exclusion
synchronized(this) { counter++; }

// oppure
AtomicInteger counter = new AtomicInteger(0);
counter.incrementAndGet();  // atomico via CAS
```

## Connessione con la GC

Il Java Memory Model e il GC condividono l'uso dei **safepoints**: punti nell'esecuzione del thread dove lo stato della JVM è "ben compreso". I safepoint sono necessari sia per garantire la visibilità tra thread (monitor release/acquire) sia per il GC (che deve fermare i thread in un safepoint per raccogliere i root).

## Connessioni

- [[concepts/java-concurrency]] — il JMM è la base teorica per tutta la programmazione concorrente in Java
- [[concepts/garbage-collection]] — GC e JMM condividono il concetto di safepoint
