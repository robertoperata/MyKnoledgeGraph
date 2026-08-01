---
tags:
  - spark-streaming
  - micro-batch
  - streaming-architecture
  - delta-index
  - batch-processing
  - data-engineering
feature:
type: article
author: Parveen Saini
source: https://www.infoq.com/articles/micro-batch-streaming-lessons-learned/
date: 2026-08-01
---

# From Batch to Micro-Batch Streaming: Lessons Learned the Hard Way in a Delta Index Pipeline

## Sunto

L'articolo documenta la migrazione in produzione di una delta index pipeline per la ricerca e il retrieval di annunci pubblicitari da job batch schedulati a streaming micro-batch continuo tramite Spark Structured Streaming. Il sistema gestisce milioni di documenti di annunci e campagne, con un indice delta nell'ordine delle decine di gigabyte e un indice completo nell'ordine delle centinaia di gigabyte. Il problema fondamentale non era il costo computazionale ma il "ritardo di scheduling e l'overhead di orchestrazione": il lag di freschezza nel caso peggiore era circa 10 minuti, non perché l'elaborazione richiedesse 10 minuti, ma perché i job aspettavano il successivo intervallo di scheduling fisso.

Prima di trovare la soluzione corretta, il team ha abbandonato due approcci che sembravano teoricamente validi. Lo streaming record-by-record è stato scartato perché la logica di indicizzazione operava a livello di raggruppamento prodotto/item (non su singoli record), rendendo gli aggiornamenti per record semanticamente disallineati: aggiornare un singolo record dell'indice poteva produrre stati parziali e risultati di ricerca inconsistenti. I file di successo e i marker di completamento dal sistema batch si sono rivelati inaffidabili in contesti di streaming continuo a causa dell'eventual consistency di S3: i marker apparivano in tempi inconsistenti e le listing delle partizioni erano temporaneamente incomplete, creando race condition impossibili da gestire in modo affidabile.

Il pattern di successo è il micro-batching deterministico basato su orologio a muro: un intervallo di trigger fisso di 30 secondi (molto più granulare della cadenza di arrivo delle partizioni di 5-7 minuti). Ogni ciclo elenca le partizioni visibili, identifica l'ultima per timestamp, la confronta con un watermark esterno persistente, e processa solo se la partizione è più recente del watermark. La scelta più controintuitiva — e corretta — è saltare le partizioni intermedie invece di elaborarle sequenzialmente: questo approccio è sicuro perché l'indice delta opera su finestre scorrevoli sovrapposte, garantendo che le partizioni saltate siano coperte nelle esecuzioni successive, con ricostruzioni periodiche dell'indice completo come backstop.

La gestione della memoria a lungo termine era una sfida non ovvia: processi JVM a lunga esecuzione mostravano crescita graduale dell'heap e pause GC crescenti nonostante tassi di input stabili, attribuibili all'accumulo di stato interno nella costruzione dell'indice. La soluzione adottata è stata configurare i job per riavvii automatici ogni 24 ore gestiti da un controller esterno (watchdog) — non come meccanismo di recovery per fallimenti, ma come "meccanismo operativo normale". Questo approccio semplifica significativamente il codice (nessuna logica di cleanup complessa), libera la memoria accumulata, e trasforma i deployment di nuove versioni in operazioni automatiche senza intervento manuale.

Il risultato finale è una riduzione del lag di freschezza nel caso peggiore da ~10 minuti a 30 secondi. La conclusione metodologica è che il sistema si applica bene a sistemi orientati al batch con requisiti di freschezza incrementali, object storage-based, e che privilegiano la freschezza rispetto alla completezza stretta di ogni singolo record. È inadatto per sistemi che richiedono elaborazione rigorosa per record, replay ordinato di tutti gli eventi storici, o garanzie forti di catch-up semantics. Come afferma l'autore: "Il miglior design di streaming è quello che funziona in modo affidabile in produzione, non quello che sembra più elegante in teoria."
