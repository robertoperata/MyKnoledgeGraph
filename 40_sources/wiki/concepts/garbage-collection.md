---
title: Garbage Collection Java
type: concept
tags: [thread2-java]
sources:
  - "[[Java GC Tuning]]"
  - "[[youtube_java-25-lts-features-jchampions]]"
updated: 2026-05-08
related:
  - "[[concepts/java-concurrency]]"
  - "[[concepts/java-memory-model]]"
---

# Garbage Collection in Java

## Obiettivo del tuning GC

Due domande fondamentali (Kirk Pepperdine):
1. Qual è il **cost model** del garbage collection?
2. Quali sono gli **input** a quel cost model?

Le risposte permettono di scegliere il collector giusto e capire l'impatto di ogni tuning.

## Weak Generational Hypothesis

La base teorica di tutti i GC generazionali:
- La **grande maggioranza degli oggetti** vive per un periodo molto breve
- Più a lungo un oggetto sopravvive, più probabilmente sopravviverà ancora

Conseguenza: è molto più efficiente raccogliere solo i giovani. Il costo di un copying collector sul young space è proporzionale agli oggetti **vivi** (pochi), non alla dimensione totale (grande).

## Safepoint

Un safepoint è un punto nell'esecuzione del thread dove lo stato è "ben compreso". Il GC può fermare i thread solo ai safepoint.

I thread raggiungono i safepoint:
- Sul back edge dei **non-counted loop**
- All'uscita di metodi
- Ai boundary JNI

> [!warning] I **counted loop** (`for (int i = 0; i < n; i++)`) non hanno safepoint check all'interno. Un loop lungo può ritardare il raggiungimento del safepoint (TTSP alto).

Il TTSP (Time To SafePoint) è spesso la causa principale di pause GC inaspettatamente lunghe: non il GC stesso, ma il tempo che tutti i thread impiegano per raggiungere il safepoint.

## Allocazione: TLAB

Il **Thread Local Allocation Block** risolve il problema di contention sull'heap pointer condiviso:
- Ogni thread ha il proprio TLAB (un chunk dell'Eden space)
- L'allocazione dentro il TLAB è "bump and run" — nessuna sincronizzazione
- Solo un thread può scrivere nel proprio TLAB
- In caso TLAB pieno o oggetto troppo grande: allocazione diretta sull'heap

## Collectors principali

| Collector | Strategia | Uso ideale |
|---|---|---|
| Serial | STW, single-thread | Applicazioni single-thread |
| Parallel | STW, multi-thread | Throughput massimo, pause tollerate |
| G1 | Regionale, concurrent | Latenza + throughput bilanciati |
| ZGC | Concurrent, regioni tagged | Latenza ultra-bassa |
| Shenandoah | Concurrent | Latenza ultra-bassa (community) |

## G1 in dettaglio

G1 divide l'heap in ~2048 regioni (da 1MB a 32MB). Young generation da sinistra, Old generation da destra. Non è necessario configurare dimensioni esplicite di young/old space.

**Concurrent failure**: se l'heap si riempie durante un ciclo concurrent → Full GC (STW). Da evitare: il metric da monitorare è l'**allocation rate**.

## Concurrent GC e Barriers

I concurrent collector (G1, ZGC, Shenandoah) devono sincronizzarsi con i thread applicativi:
- **Write Barrier**: notifica il GC quando un riferimento cambia (es. card marking in G1)
- **Read Barrier**: usato in ZGC/Shenandoah per gestire oggetti che vengono spostati concurrently (colored pointers, load barriers)

Costo: le barrier aumentano la latenza media (p50, p80) pur riducendo le pause worst-case.

## Novità Java 25: Compact Object Headers ⭐

Riduzione della rappresentazione in memoria degli oggetti nel heap:
- **Risparmio stimato: ~25% del heap** (dipende dalle dimensioni degli oggetti)
- Zero modifiche al codice applicativo — miglioramento automatico
- Particolarmente impattante per applicazioni con molti oggetti piccoli

> "Finding a way to remove 25% of the heap after 25 years — just this could be a reason to move to Java 25." — Ario

## Stato collectors in Java 25

| Collector | Modalità generazionale | Note Java 25 |
|---|---|---|
| **G1** | Sempre generazionale | Default; miglioramenti continui |
| **ZGC** | ✅ Generazionale (default) | La modalità non-generazionale è stata **rimossa** perché inferiore |
| **Shenandoah** | ✅ Generazionale (disponibile) | Ora supporta la modalità generazionale |
| **Serial / Parallel** | Generazionale | Invariati |

Il ZGC generazionale è così superiore che il team ha rimosso la versione non-generazionale — non c'era motivo di mantenerla.

## AOT (Ahead-of-Time) Compilation Caching — Java 25

Il JIT (C1/C2) può **cachare le ottimizzazioni** tra una run e l'altra, eliminando il warm-up:

```bash
# Run con AOT caching (registra e usa il profilo)
java -XX:AOTMode=on -XX:AOTConfiguration=app.aotconf MyApp
# ~3× più veloce alla seconda esecuzione
```

In Java 25 non è più necessario un step separato di profiling. Il JVM salva automaticamente il profilo AOT durante la prima esecuzione e lo usa in quelle successive come baseline di ottimizzazione.

**Limitazioni attuali:** profilo CPU-specific; CI/CD integration non ancora standardizzata.

## Implicazione per l'architettura

> [!hint] Più thread nella JVM = più stack frame da scansionare per GC roots = pause GC più lunghe. In un microservizio su Kubernetes (es. 4 cores) con 200 thread per le richieste HTTP, i thread pool hanno un impatto diretto sul GC.

Questo crea una connessione con i [[concepts/virtual-threads]]: milioni di virtual thread ma solo pochi carrier thread → molti meno GC roots.

## Connessioni

- [[concepts/java-memory-model]] — condividono il concetto di safepoint
- [[concepts/java-concurrency]] — numero di thread impatta direttamente le pause GC
- [[concepts/virtual-threads]] — virtual thread riducono i carrier thread reali → meno GC roots
