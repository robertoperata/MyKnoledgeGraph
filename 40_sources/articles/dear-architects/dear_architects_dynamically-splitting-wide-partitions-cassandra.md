---
tags:
  - cassandra
  - time-series
  - database-performance
  - partitioning
  - netflix
  - nosql
feature:
type: article
author: Rajiv Shringi, Kaidan Fullerton, Oleksii Tkachuk, Kartik Sathyanarayanan
source: https://netflixtechblog.com/dynamically-splitting-wide-partitions-in-cassandra-for-time-series-workloads-0eded064f456
date: 2026-08-09
---

# Dynamically Splitting Wide Partitions in Cassandra for Time-Series Workloads

## Sunto

Il Netflix TechBlog documenta il problema delle "wide partition" nel sistema TimeSeries Abstraction di Netflix, che gestisce petabyte di dati temporali su Apache Cassandra 4.x. Il sistema eccelle nei write a bassa latenza (milioni/secondo), ma soffre quando singole partizioni accumulano troppi eventi nel tempo: le latenze di lettura degradano da single-digit millisecond a secondi, con pause GC eccessive, alta CPU e thread queueing. Semplicemente scalare l'infrastruttura non è una soluzione efficiente, perché i pattern di workload iniziali spesso non riflettono quelli produttivi reali.

Netflix ha implementato due strategie complementari. La prima — **Time Slice Re-Partitioning** — usa un background worker che monitora gli istogrammi delle partizioni tramite le API di introspezione di Cassandra (come `nodetool tablehistograms`). Quando le partizioni scendono sotto soglie configurate (tipicamente 2-10 MiB), il sistema aggiusta automaticamente le configurazioni dei time slice futuri. Questo approccio è efficace ma non gestisce i casi in cui solo specifici TimeSeries ID all'interno di una tabella sono problematici.

La seconda strategia — **Dynamic Partitioning per ID** — è una pipeline asincrona in tre fasi. Nella **fase di Detection**, le operazioni di lettura sono monitorate tracciando i byte letti per partizione; eventi di detection vengono emessi su Kafka quando le soglie sono superate, partendo dalle partizioni immutabili per ridurre la complessità. Nella **fase di Planning & Splitting**, il planner legge l'intera partizione per calcolare la strategia di split ottimale, usa checkpointing per gestire letture parziali, e implementa checksum (pre e post-split) per la validazione. Lo split avviene assegnando bucket di eventi aggiuntivi allo stesso time bucket. Nella **fase di Serving**, Bloom filter in-memory (latenza single-digit microsecond) tracciano le partition key splittate sui server TimeSeries; i lookup di metadata guidano la lettura delle partizioni splittate, delegando al `PartitionReader` esistente per leggere più partizioni più piccole in parallelo.

La sicurezza del sistema è garantita da tre meccanismi: le partizioni wide originali non vengono mai cancellate (abilitando opzioni di fallback), pipeline Data Bridge verificano gli split offline con job Spark, e una strategia di rollout in fasi con modalità "Comparison" valida che i path di lettura vecchi e nuovi servano dati identici. La tabella di metadata `wide_row` centralizza gli stati di split, i checkpoint e le informazioni di routing, fungendo da backbone dell'intera pipeline.

I risultati quantificati sono significativi: latenza media ridotta da secondi a double-digit millisecond, tail latency da diversi secondi a circa 200 millisecond, riduzione significativa dei timeout di lettura, eliminazione del thread queueing e riduzione della CPU. Il caso estremo: partizioni da 500MB+ possono essere interrogate con paginazione mantenendo la disponibilità. Le lezioni chiave sono due: ridurre la surface area partendo da soluzioni semplici che abbiano impatto significativo, e investire in meccanismi di validazione proporzionali alla complessità, portata e impatto potenziale del feature.

