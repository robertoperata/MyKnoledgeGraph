---
tags:
  - streaming
  - rust
  - distributed-systems
  - event-driven
  - database
  - architecture
type: article
author: Peter (Tansu creator), host non identificato
source: https://www.youtube.com/watch?v=pJQ7hcsI1Dw
date: 2026-05-05
---

# tansu — pluggable stateless kafka (written in rust)

**Durata:** 3h 30m  
**Lingua originale:** inglese

---

## Sintesi

Intervista approfondita con il creatore di **Tansu**, un broker Kafka-compatibile scritto in Rust, stateless, con storage engine pluggabile. Peter, con oltre 10 anni di esperienza con Kafka (inclusa l'implementazione di un client Kafka completo in Erlang per sistemi di trading ad alta frequenza), ha costruito Tansu partendo dalle frustrazioni accumulate: complessità eccessiva, replicazione ridondante su storage già ridondanti, offset retention invisibili, difficoltà di deployment, e scarso supporto per ecosistemi non-Java.

Tansu è un singolo binario da 50MB, Apache licensed, che espone l'API Kafka ma non è Kafka: non usa segmenti, non usa Raft, non replica tra broker. Tutta la durabilità è delegata allo storage engine (SQLite, PostgreSQL, S3, memoria). Il modello è analogo a WarpStream/bufstream: broker stateless che leggono e scrivono dallo stesso storage, senza coordinarsi tra loro. Ogni broker è "Spartacus" — non sa che gli altri esistono.

L'obiettivo dichiarato è coprire l'80% dei casi d'uso con la semplicità massima: developer experience, deployment senza Zookeeper né cluster obbligatorio, schema validation nel broker, CLI completa, generazione di test data, e potenziale per bindings Python via Rust FFI. Tansu non punta a competere con Confluent o Apache Kafka nei deployment enterprise da petabyte — punta agli sviluppatori, ai team piccoli, agli ecosistemi non-Java, e a chi vuole Kafka senza l'overhead operativo.

---

## Punti chiave

- **Architettura stateless**: ogni broker Tansu è indipendente, non sa degli altri broker. La durabilità è delegata allo storage engine (Postgres, S3, SQLite). Nessuna replicazione tra broker → nessun Raft, nessun leader election tra broker. Equivalente al modello WarpStream/bufstream.

- **Storage engine pluggabili**: `memory://` (effimero), `sqlite://` (singolo file, ottimo per test), `postgres://` (produzione SQL), `s3://` (locking ottimistico con conditional writes). Esiste anche `null://` per tuning. Il broker riceve una URL, non sa nulla del come lo storage gestisce la durabilità.

- **Schema validation nel broker**: Tansu valida ogni singolo record (non il batch) contro Avro, JSON Schema, o Protobuf. Lo schema è un file (`{topic}.proto`, `{topic}.avsc`, `{topic}.json`) in un bucket S3. Una sola versione dello schema — nessun schema registry. Raccomandazione: usare Protobuf che ha evoluzione built-in tramite numerazione dei campi.

- **Record come righe SQL**: a differenza di Kafka che persiste batch opachi, Tansu spacca il batch e scrive ogni record come riga separata in SQL. Questo abilita schema validation e query dirette sul database con strumenti SQL normali. Trade-off: throughput inferiore, ma SQL engineers possono interrogare direttamente il dato.

- **Nessuna retention sugli offset**: una scelta deliberata. In Kafka il default è 7 giorni di retention degli offset dei consumer group — una fonte frequente di sorprese in produzione. In Tansu, un offset committed rimane per sempre.

- **CLI completa integrata nel binario**: `tansu broker`, `tansu topic` (create/list/delete), `tansu cat` (produce/consume, converte Protobuf ↔ JSON), `tansu proxy` (debug del protocollo Kafka), `tansu generator` (genera test data da schema Protobuf). Tutto in 50MB, Docker image from-scratch (nessun OS).

- **Generazione test data da schema**: `tansu generator` usa le definizioni Protobuf per generare dati realistici (email, nomi, date) tramite script Rhai. Lo schema diventa unica fonte di verità sia per la validazione che per la generazione.

- **Scripting engine Rhai + WASM**: Rhai è un linguaggio scripting basato su Rust, compilabile in WASM, già usato per la generazione test data. Il piano è renderlo il motore del proxy programmabile: plugin che redirezionano topic verso broker diversi, trasformano messaggi, emulano topic inesistenti — simile a WireGuard per Kafka.

- **Integrazione Apache Iceberg e Delta Lake**: Tansu può scrivere i dati prodotti direttamente in tabelle Iceberg o Delta Lake, abilitando un path diretto da stream a lakehouse senza connettori separati.

- **Motivazione non-Java**: il creatore viene da Erlang, ha scritto un client Kafka completo in Erlang (incluso sticky partition rebalancing, anticipando la feature di Apache Kafka). Rust è stato scelto per pattern matching, assenza di GC, e facilità di creare C bindings (→ Python). La visione è un client Rust che possa essere usato da Python, Erlang, Go via FFI.

- **Competitor positioning**: Tansu è più vicino a Red Panda (single binary, developer friendly, no JVM) e WarpStream (leaderless, storage esterno) che ad Apache Kafka. Non compete con Confluent o Redpanda per deployment da petabyte.

---

## Citazioni notevoli

> "Tanu is Kafka compatible but it's not Kafka. It's different because storage is separate. It's stateless. Doesn't replicate data."  
> — Peter (creatore di Tansu)

> "Every Tanu broker is Spartacus. I am Spartacus. It doesn't matter if one disappears because nothing is lost."  
> — Peter

> "The schema registry doesn't make sense to me. The idea that you're going to make an HTTP request to a service that might be unavailable to fetch a schema which is already compiled into your program just doesn't make sense."  
> — Peter

> "We joked that we hadn't built a platform to process transactions. We'd actually built a metrics processing system — we had so many metrics for the Kafka cluster."  
> — Peter (sulla complessità operativa di Kafka in produzione)

> "Completable Future has no future."  
> *(citato nel contesto della complessità operativa di Kafka)*

> "It's all the itches I've scratched for 10 plus years of Kafka."  
> — Peter, sulla quantità di feature di Tansu

---

## Architettura in dettaglio

### Produce flow (storage Postgres)

```
Producer → Tansu broker
    │
    ├── 1. Verifica sequenza idempotent producer (tabella sequences)
    ├── 2. Incrementa offset per topic-partition (dentro transazione SQL)
    ├── 3. Decomprimi il batch
    ├── 4. Per ogni record: valida contro schema (Avro/Protobuf/JSON)
    └── 5. Inserisci ogni record come riga separata (tabella records)
```

### Storage engines

| Engine | Caso d'uso | Note |
|---|---|---|
| `memory://` | Test effimeri | Nessuna persistenza |
| `sqlite://` | Dev, CI/CD, test riproducibili | File singolo, copiabile |
| `postgres://` | Produzione SQL | Transazioni ACID, scalabilità verticale |
| `s3://` | Produzione cloud | Locking ottimistico (conditional writes), no coordinator |
| `null://` | Benchmark/tuning | Scarta tutto |

### Scaling orizzontale con Postgres

Più broker Tansu puntano allo stesso Postgres. Non comunicano tra loro. Il locking è gestito dalle transazioni SQL. Ogni broker può servire qualsiasi partizione (nessun leader per partizione).

---

## Confronto con Kafka e alternative

| Feature | Apache Kafka | Tansu | WarpStream/bufstream |
|---|---|---|---|
| Linguaggio | Scala/Java | Rust | Go |
| Replicazione | Tra broker (RF=3) | Delegata allo storage | Delegata a S3 |
| Deployment minimo | 3 broker + ZK/KRaft | 1 broker + DB/S3 | 1 broker + S3 |
| Schema validation | Solo client-side | Nel broker | No |
| Config options | 300+ | Opinionated, minime | — |
| Retention offset | 7 giorni (default) | Permanente | — |
| Record storage | Batch opachi su disco | Righe SQL | Oggetti S3 |
| Protocollo | Kafka | Kafka-compatible | Kafka-compatible |
| Licenza | Apache 2.0 | Apache 2.0 | Proprietario |

---

## Contesto storico: client Kafka in Erlang

Peter ha implementato un client Kafka completo in Erlang (~2018) per un sistema di trading ad alta frequenza, partendo dalle definizioni EBNF del protocollo (pre-JSON files). Include:
- Producer e consumer completi
- **Sticky partition rebalancing** — implementato in anticipo rispetto ad Apache Kafka, adattando il protocollo consumer group per garantire che in caso di rebalance ogni client ricevesse le stesse partizioni di prima
- Offset commit su Kafka (non Zookeeper)

---

## Riferimenti e risorse

- **GitHub**: https://github.com/tansu-io/tansu
- **Docs**: https://docs.tansu.io
- **Cargo install**: `cargo install tansu` (installa da crates.io)
- **Docker**: immagine from-scratch, 50MB, ARM + x64, nessun OS
- **Licenza**: Apache 2.0
- **Rhai scripting**: https://rhai.rs (linguaggio scripting Rust-based, compilabile WASM)
- **Apache Iceberg**: formato tabellare open-table per analytics (integrato in Tansu)
- **Delta Lake**: formato tabellare open-source di Databricks (integrato in Tansu)
- **librdkafka**: client C per Kafka (usato da Python, Go, altri — alternativa a cui Tansu potrebbe affiancarsi)
- **lipstreaming** (Datadog): client Kafka in Rust proprietario con bindings Java/Go/Python
- **WarpStream**: broker Kafka-compatible stateless su S3 (acquisito da Confluent)
