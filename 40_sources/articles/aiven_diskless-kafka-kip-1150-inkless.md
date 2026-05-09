---
tags:
  - streaming
  - event-driven
  - cloud
  - distributed-systems
  - architecture
type: article
author: Filip Yonov (Head of Streaming Services, Aiven)
source: https://aiven.io/blog/diskless-kafka-is-the-tide-and-its-rising
date: 2025-05-15
---

# Diskless Kafka is the Tide, and it's Rising

## Sunto

L'articolo di Filip Yonov (Aiven) è il manifesto tecnico e strategico di **KIP-1150 Diskless Topics** e del progetto **Inkless** — un fork temporaneo di Apache Kafka 4.0 che rende disponibile subito la funzionalità, senza aspettare l'integrazione upstream che potrebbe richiedere anni.

L'idea centrale è semplice: i cluster Kafka su cloud spendono oltre l'**80% del budget** in traffico cross-AZ e SSD. Diskless Topics risolve entrambi scrivendo direttamente sullo object storage (S3, GCS, Azure Blob) invece che su disco locale. Il risultato è fino all'80% di riduzione del TCO, con la stessa API Kafka, gli stessi client, gli stessi ACL — senza vendor lock-in.

La novità critica rispetto a Tiered Storage (l'approccio precedente) è che Diskless interviene sul **write path caldo** (non solo sull'archivio freddo) e che può essere abilitato **per singolo topic**, consentendo cluster misti: topic a bassa latenza con replicazione classica e topic cost-optimized diskless sullo stesso cluster. Questo preserva il network effect di Kafka — il suo vero valore — evitando la frammentazione in cluster separati.

Aiven usa già Diskless internamente per i log di 170.000+ servizi free-tier (PostgreSQL, MySQL) a ~100 MB/s con p99 latency di 3–3.5s. Il target è sub-2 secondi con tuning base. Il fork Inkless sarà ritirato quando upstream Kafka assorbirà Diskless.

---

## Punti chiave

- **Il problema economico del cloud Kafka**: il traffico cross-AZ (replicazione tra 3 AZ) e il costo degli SSD consumano oltre l'80% del bill cloud. Ogni nuovo topic aggiunge costo, disincentivando l'adozione all'interno delle organizzazioni e erodendo il network effect che rende Kafka prezioso.

- **Il network effect come vero valore di Kafka**: la forza di Kafka non è solo il throughput o l'exactly-once. È che ogni nuovo consumer su un topic esistente aggiunge valore senza infrastruttura aggiuntiva. Appena si frammentano i cluster (uno per bassa latenza, uno per object storage), si perde questo vantaggio: "The moment engineers must ask 'Which cluster has the truth?' you've lost the zero-integration promise."

- **KIP-1150 Diskless Topics**: una singola flag per topic (`diskless.enable=true` o `topic.type=inkless`). Modifica ~1.7% del codice core di Kafka, completamente upstream-aligned. Il patch applica su Kafka 4.0 (KRaft, no ZooKeeper).

- **Architettura Diskless** (write path): i batch del producer vengono scritti direttamente sullo object storage; l'assegnazione degli offset è differita a un **lightweight SQL sequencer** che li commita in modo transazionale.

- **Architettura Diskless** (read path): i broker interrogano il sequencer per le coordinate dell'offset, controllano un **edge cache** prima, poi lo object storage per i dati freddi. Gli offset vengono ripristinati prima che i record escano dal broker.

- **Housekeeping**: un **background compactor** unisce i piccoli oggetti in oggetti più grandi, minimizzando la read amplification — problema tipico delle architetture S3-based.

- **Tiered Storage vs Diskless**: Tiered Storage lascia il write path caldo su disco e cross-AZ (dove il costo è massimo) e serve solo per l'archivio freddo. Diskless elimina il disco fin dal primo byte. I due approcci non sono equivalenti.

- **Cluster misti**: sullo stesso cluster possono coesistere topic classici (replicati, sub-100ms) e topic diskless (cost-optimized). Un singolo flag per topic. Questo permette di servire pagamenti (bassa latenza) e telemetria/backfill (alto volume, costo basso) con la stessa infrastruttura.

- **Inkless — fork temporaneo**: Inkless è il nome del fork open-source di Kafka 4.0 di Aiven. Il nome deriva dal fatto che Kafka scrisse "con inchiostro su carta", mentre Diskless persiste su object storage. Ogni commit è pubblico e viene proposto upstream come PR. L'obiettivo è ritirare il fork quando upstream assorbirà Diskless — non creare una community separata.

- **Produzione interna (Aiven)**: 170.000+ servizi free-tier (PostgreSQL, MySQL) che generano log via piccoli cluster Diskless in 3 regioni a ~100 MB/s (con compressione 4x). P99 latency: 3–3.5s. Target con tuning base: sub-2s. Disk usage quasi zero nonostante il throughput.

- **Prossimo milestone — KIP-1164**: sostituire il SQL sequencer con un **topic-based coordinator** che incorpora i metadati di ordinamento nei topic Kafka stessi, riducendo gli hop di coordinamento.

- **Azure**: eccezione alla regola — il traffico cross-AZ su Azure è gratuito, quindi Diskless non porta risparmio sui costi di rete. Il configuratore interattivo di Aiven mostra questi inflection point per cloud provider.

---

## Configurazione in un comando

```bash
# Abilitare Diskless su un topic esistente o nuovo
kafka-topics.sh --create --topic payments --config diskless.enable=true
```

```bash
# Testare Inkless in locale
git clone https://github.com/aiven/inkless.git
cd inkless && make demo
```

---

## Architettura Diskless vs Tiered Storage vs Direct-to-S3

| Approccio | Write path | Latenza | Caso d'uso |
|---|---|---|---|
| **Kafka classico** | Disco locale + replicazione cross-AZ | < 10ms | Bassa latenza, alta affidabilità |
| **Tiered Storage** | Disco locale (hot) + S3 (cold) | < 10ms (hot) | Retention lunga, archivio |
| **Direct-to-S3** | S3 direttamente (batch) | Secondi/minuti | Log, archivio, non real-time |
| **Diskless (KIP-1150)** | Object storage direttamente + sequencer | 2–3.5s (attuale) | Cost-optimized, near-real-time |

---

## Link esterni

- [KIP-1150: Diskless Topics](https://cwiki.apache.org/confluence/display/KAFKA/KIP-1150%3A+Diskless+Topics) — proposta upstream ufficiale
- [Inkless su GitHub](https://github.com/aiven/inkless) — fork open-source di Kafka 4.0 con Diskless abilitato
- [KIP-1164: Topic Based Batch Coordinator](https://cwiki.apache.org/confluence/display/KAFKA/KIP-1164%3A+Topic+Based+Batch+Coordinator) — prossimo milestone: coordinator senza SQL
- [KIP-1176: Slack's Tiered Storage for Active Log Segment](https://cwiki.apache.org/confluence/display/KAFKA/KIP-1176%3A+Tiered+Storage+for+Active+Log+Segment) — proposta concorrente di Slack
- [Kafka Monthly Digest April 2025](https://developers.redhat.com/blog/2025/05/06/kafka-monthly-digest-april-2025) — contesto sui 22 nuovi KIP del mese del lancio (Mickael Maison)
- [Aiven Diskless product page](https://aiven.io/products/diskless) — accesso BYOC
- [Antithesis](https://antithesis.com/) — tool citato per testing distribuito
- [Primo blog post su Diskless](https://aiven.io/blog/diskless-apache-kafka-kip-1150) — articolo introduttivo precedente

---

## Immagini

- Grafico throughput: ~100 MiB/s cluster throughput con disk usage quasi zero
- Grafico disk usage: ~30 MB di SSD usati (solo metadati KRaft e topic interni) nonostante alto throughput
- Diagramma del fork: relazione tra Inkless, upstream Kafka e deployment di produzione
- Screenshot post Mickael Maison sulla community Kafka
