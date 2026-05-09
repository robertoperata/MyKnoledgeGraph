---
tags:
  - streaming
  - event-driven
  - cloud
  - architecture
  - platform-engineering
type: article
author: Filip Yonov (Head of Streaming Services, Aiven)
source: https://aiven.io/blog/apache-kafkas-80-percent-problem
date: 2025-10-08
---

# Apache Kafka's 80% Problem

## Sunto

L'articolo di Filip Yonov (Aiven) parte da un dato scomodo: circa l'**80% dei cluster Kafka opera a scala piccola** (sub-1 MB/s di throughput) pagando prezzi pensati per sistemi big-data. Questa è "l'80% problem" di Kafka: un mismatch strutturale tra workload e modello di costo.

I dati citati sono concreti: il 56% dei cluster è sotto ~1 MB/s (da survey Confluent e Redpanda); solo il 9% delle organizzazioni ha adottato lo streaming a livello enterprise; la flotta Aiven (4.000 servizi) ha una mediana di ~9.81 MB/s, con il 20% dei cluster che genera l'80% del throughput totale. Per contestualizzare: 1 MB/s = 86 GB/giorno. Un'operazione e-commerce a 5 ordini/secondo genera 12.5 KB/s — servirebbe una crescita di 80x per raggiungere 1 MB/s.

Il problema economico è brutale: un deployment Kafka con 3 broker, 3 AZ, RF=3 costa **~$300.000/anno** tutto incluso (infrastruttura + personale). Lo stesso volume di dati persistito su S3 costerebbe **meno di $5.000/anno**. Il managed service riduce a ~$50.000/anno ma rimane sproporzionato per workload piccoli.

> "Self-manage a three-AZ, RF=3 posture that rounds to at least $300,000/yr once infra + people are counted, or buy managed service that is priced around $50,000/yr while persisting the same data in plain S3 can be easily done <$5k/yr."

La risposta di Aiven è **Inkless** — cluster con *storage class* multiple sullo stesso Kafka, da classic replicato a diskless su object storage. Un singolo cluster può servire topic a bassa latenza (<20ms) e topic cost-optimized (<2s, fino al 90% di risparmio) simultaneamente. Il caso studio OpsHelm: costi ridotti da >$50k/yr a <$10k/yr consolidando MSK+NATS in un singolo cluster Inkless BYOC.

---

## Segmentazione del mercato Kafka

| Segmento | % cluster | Profilo | Problema |
|---|---|---|---|
| **Small-data** | ~80% | Sperimentazione, produzione siloed, sub-1 MB/s | Pagano infrastruttura big-data per workload piccoli |
| **Savvy self-managers** | ~15% | Datadog, Uber scale, team infra dedicati | Kafka va bene così, ma costi operativi alti |
| **True big-data** | ~5% | Hyperscaler, enterprise, stream processing massiccio | Mercato affollato di vendor |

Obiettivo dichiarato di Aiven: ridurre il segmento small-data dall'80% al 50% rendendo Kafka accessibile e conveniente per workload piccoli.

---

## Storage class di Inkless Cluster

| Storage Class | Latenza E2E | SLA | Risparmio vs classic | Stato |
|---|---|---|---|---|
| Classic 3-zone | <20 ms | 99.99% | — (baseline) | GA |
| Classic 3-node (single-AZ) | <20 ms | 99.9% | ~3× cheaper | GA |
| Diskless Standard | <2 s | 99.99% | fino al 90% | GA |
| Diskless Express | <1 s | 99.99% | significativo | Coming |
| Diskless Global | TBD | 99.999% | — (multi-region) | Coming |
| Lakehouse | N/A | — | analytics zero-copy | Coming |

---

## Contesto storico: perché Kafka è così

Kafka è nato a **LinkedIn** per sostituire pipeline punto-a-punto con un commit log condiviso. L'architettura distribuita con replicazione e partizionamento è ottimizzata per scala massiccia — ma introduce overhead fisso per deployment piccoli. I 300+ parametri di configurazione beneficiano gli specialisti ma overwhelmano i team tipici.

> "Flexibility reads as ambiguity for non-expert teams."

---

## Sviluppi recenti dell'ecosistema Kafka (al momento dell'articolo)

- **Kafka 4.0**: KRaft come default (addio ZooKeeper)
- **Tiered Storage**: retention infinita
- **Queues for Kafka (KIP-932)**: in early access
- **Diskless Topics (KIP-1150)**: replicazione su object storage — la base di Inkless

---

## Visione strategica

> "Make the central log cheap to start and safe to scale."

L'argomento di mercato: rendere lo streaming economico per il segmento small-data allarga il mercato totale. Più team adottano Kafka per workload piccoli → più workload superano la soglia critica → più deployment diventano platform-wide.

---

## Link esterni

- [Aiven Inkless BYOC](https://aiven.io/inkless) — prodotto annunciato nell'articolo
- [Confluent 2025 Data Streaming Report](https://www.confluent.io/blog/2025-data-streaming-report/) — fonte dei dati sul 56% dei cluster sub-1 MB/s
- [Redpanda Survey (TheNewStack)](https://thenewstack.io/ai-will-drive-streaming-data-adoption-says-redpanda-survey/) — secondo survey citato
- [KIP-1150: Diskless Topics](https://cwiki.apache.org/confluence/display/KAFKA/KIP-1150%3A+Diskless+Topics) — specifica tecnica upstream
- [RudderStack: Why PostgreSQL Over Kafka](https://www.rudderstack.com/blog/why-rudderstack-used-postgres-over-apache-kafka-for-streaming-engine/) — caso reale di team che ha scelto alternative per workload piccoli
- [OpsHelm Case Study](https://aiven.io/blog/opshelm-multicloud-changelog-with-diskless-apache-kafka) — 5× riduzione costi con Inkless BYOC
- [Inkless GitHub Repository](https://github.com/aiven/inkless) — fork open-source di Kafka 4.0 con KIP-1150

---

## Immagini

Nessuna immagine con contenuto tecnico rilevante (le immagini presenti sono grafici promozionali).
