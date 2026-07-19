---
tags:
  - microservizi
  - topologia-di-servizio
  - stream-processing
  - architettura-distribuita
  - osservabilità
feature:
type: article
author: Parth Jain, Rakesh Sukumar, Yingwu Zhao, Renzo Sanchez-Silva, Nathan Fisher
source: https://netflixtechblog.com/building-service-topology-at-scale-architecture-challenges-and-lessons-learned-f4b792f3f0d8
date: 2026-07-19
---

# Building Service Topology at Scale: Architecture, Challenges, and Lessons Learned

## Sunto

Netflix ha costruito un sistema di mappatura delle dipendenze tra servizi in tempo reale, chiamato Service Topology, capace di elaborare milioni di record di flusso al secondo su più regioni geografiche. Il sistema fornisce un grafo interrogabile e sempre aggiornato di tutte le relazioni inter-servizio nel fleet di microservizi Netflix, elaborando oltre un miliardo di edge al giorno. L'architettura si articola su tre livelli indipendenti: il livello di rete (basato su eBPF flow logs in un database a grafo distribuito), il livello IPC (metriche applicative via Server-Sent Events in uno store a grafo separato) e il livello di tracing (tracce distribuite in formato Parquet colonnare). La separazione fisica consente l'ottimizzazione indipendente per i pattern di throughput e query di ciascun livello.

Il cuore del sistema è un pipeline di aggregazione distribuita a tre stadi. Il primo stadio (FlowLog Ingestion) consuma i flussi Kafka multi-regione, filtra i record non validi, crea batch in finestre di 5 minuti e distribuisce gli aggregatori tramite consistent hashing. Il secondo stadio (Intermediary Resolution) risolve gli intermediari di rete raggruppando i flussi e unendo percorsi in entrata e in uscita per creare edge diretti tra applicazioni (es. App A → App B invece di App A → Load Balancer → App B). Il terzo stadio esegue l'aggregazione finale, arricchisce i dati con metadati esterni e persiste nel database a grafo con throttling controllato.

L'approccio a tre stadi distribuisce il lavoro su più istanze, prevenendo i "hot node" che concentrano il traffico per i servizi più popolari. Tra le decisioni tecniche fondamentali spicca la scelta di un'architettura streaming-first con gestione del backpressure: quando gli stadi downstream rallentano, segnalano agli upstream di fermarsi, permettendo a Kafka di bufferizzare i dati naturalmente. Si è scelto Server-Sent Events invece di gRPC perché quest'ultimo rappresentava un collo di bottiglia per le risposte in streaming alla loro scala.

In produzione sono emersi numerosi ostacoli: lag dei consumer Kafka risolto aumentando le partizioni e ottimizzando i parametri di fetch; hot node causati dalla distribuzione power-law dei servizi popolari, risolti con il pipeline multi-stadio; pressione sulla heap JVM e pause del garbage collector, affrontate eliminando conversioni inutili e usando aggregatori mutabili sui path critici nonostante le convenzioni di immutabilità di Scala. Il sistema include anche una capacità di time-travel query che permette di interrogare la topologia storica attraverso snapshot degli aggregatori con finestre temporali e tracking delle mutazioni a livello di proprietà nel database a grafo.

La lezione principale è che la scala cambia tutto: approcci validi a 100 richieste/secondo falliscono a 100.000/secondo. L'immutabilità dei dati, principio nobile in Scala, ha richiesto compromessi pragmatici sui path critici. Il mantra del team è "ottimizzare un collo di bottiglia alla volta": nei sistemi distribuiti, risolvere un vincolo ne rivela immediatamente il successivo. Service Topology è oggi in produzione, usato quotidianamente per l'analisi degli incidenti, la valutazione del blast radius e la comprensione delle dipendenze nell'architettura distribuita di Netflix.

## Codice

Il pipeline di aggregazione a tre stadi usa consistent hashing per distribuire il lavoro. Il seguente schema illustra la logica di risoluzione degli intermediari nel secondo stadio:

```
Stadio 1: FlowLog Ingestion
  Kafka (multi-region) → filtraggio → batch 5min → consistent hashing → Aggregatori

Stadio 2: Intermediary Resolution
  (App A → LB) + (LB → App B) → join per intermediario → edge diretto (App A → App B)

Stadio 3: Final Aggregation
  Aggregazione finale → arricchimento metadati → throttling → Graph DB
```

Il backpressure propagation con Pekko Streams:

```scala
// Approccio semplificato: backpressure automatica in Akka/Pekko Streams
// Quando lo stage downstream rallenta, il buffer upstream si riempie
// e la produzione viene automaticamente sospesa senza perdita di dati
source
  .via(stage1Filter)
  .via(stage2IntermediaryResolution)
  .via(stage3FinalAggregation)
  .runWith(graphDatabaseSink)
```
