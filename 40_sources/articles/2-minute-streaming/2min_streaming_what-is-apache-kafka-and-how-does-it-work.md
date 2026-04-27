---
tags:
  - apache-kafka
  - stream-processing
  - distributed-systems
  - messaging
  - kafka-streams
  - event-driven
feature:
type: article
author: Stanislav Kozlovski
source: https://stanislavkozlovski.medium.com/what-is-apache-kafka-and-how-does-it-work-16023aa2efee
date: 2026-04-27
---

# What is Apache Kafka and How Does It Work?

## Sunto

Apache Kafka è un sistema pub/sub distribuito, durevole, scalabile e fault-tolerant con capacità di stream processing e integrazione. Il suo pilastro fondamentale è la struttura dati **log append-only**: ogni record viene aggiunto in coda alla sequenza e riceve un offset univoco e monotonicamente crescente. Tutte le operazioni di lettura e scrittura sono sequenziali, il che sfrutta le caratteristiche prestazionali dei dischi (I/O sequenziale molto più veloce di quello casuale) e garantisce performance O(1) in scrittura. I messaggi sono coppie chiave-valore di byte grezzi; la serializzazione avviene lato producer, la deserializzazione lato consumer. I **Topic** forniscono separazione logica dei dati e si dividono in **Partizioni**, ognuna delle quali è un'istanza indipendente del log: questa struttura abilita la parallelizzazione delle letture, ma aumentare il numero di partizioni dopo la creazione rompe le garanzie di ordinamento sui dati preesistenti.

L'architettura distribuita di Kafka si basa sui **Broker** (nodi server) che formano il cluster e replicano le partizioni con un replication factor configurabile (default 3). Viene usata la **single-leader replication**: ogni partizione ha un leader che accetta le scritture, mentre i follower replicano attivamente. La gestione dei metadati e l'elezione dei controller erano storicamente affidate a ZooKeeper, ma sono state migrate a **KRaft** (Kafka Raft), un algoritmo ispirato a Raft che usa il topic speciale `__cluster_metadata` come log di eventi del cluster. Tre nodi controller formano un quorum: solo il controller attivo (leader del log metadata) accetta aggiornamenti, mentre gli altri sono hot standby. I broker inviano heartbeat ai controller e, se assenti per sei secondi, vengono recintati e le loro partizioni leader vengono riallocate.

I **Consumer Group** coordinano più istanze consumer che processano un topic in parallelo, distribuendo le partizioni tra i membri. Ogni partizione è assegnata a un solo consumer nel gruppo, il che preserva l'ordinamento locale senza lock distribuiti. L'avanzamento di lettura è persistito nel topic speciale `__consumer_offsets`, abilitando failover senza perdita di progresso. Kafka supporta **transazioni atomiche** multi-partizione tramite il Transaction Coordinator: i producer possono scrivere messaggi che i consumer vedono solo dopo il commit. L'idempotenza del producer previene duplicati in caso di retry di rete tramite ID monotonicamente crescenti. Quando le operazioni di lettura e scrittura coinvolgono esclusivamente Kafka (come in Kafka Streams), si ottiene l'**exactly-once processing** completo.

L'ecosistema Kafka comprende tre componenti principali. **Kafka Streams** è una libreria Java per lo stream processing continuo con API dichiarative (filter, map, join, windowed aggregation) che sfrutta le transazioni Kafka per garantire exactly-once. Lo **Schema Registry** è un servizio HTTP esterno che gestisce le associazioni `{schema, topic}`, necessario perché Kafka non supporta nativamente la tipizzazione dei messaggi (implementazioni principali: Confluent Schema Registry, Karapace, AWS Glue). **Kafka Connect** è un framework runtime per integrazioni source/sink con centinaia di plugin community (ElasticSearch, PostgreSQL, Snowflake, BigQuery) che riduce le integrazioni a configurazione no-code/low-code. Il **Tiered Storage** affronta il problema del volume a scala: offload asincrono dei dati freddi su S3 con riduzione dei costi superiore al 10x, separando il tier hot (disco locale del broker) dal tier cold (object store).

Kafka è la scelta ideale quando si richiedono: alta durabilità con deployment multi-AZ, alta disponibilità con failover robusto, alto fan-out in lettura, replayabilità degli eventi, o volumi elevati di dati append-only (tracking, metriche, log, CDC). Non è la scelta giusta per: task asincroni semplici (Redis è preferibile), semantica a coda classica (Postgres o RabbitMQ), IoT con protocolli leggeri (MQTT), latenze estreme inferiori a 15ms p99, o deployment di piccola scala. L'autore enfatizza di evitare l'over-engineering: la maggior parte delle organizzazioni non ha requisiti "big data" e può raggiungere i propri obiettivi con l'infrastruttura esistente. Le funzionalità emergenti includono: Queue API con acknowledgment a livello di record, Diskless Topics con architettura leaderless su S3 (potenziale riduzione costi >90%), e Iceberg Topics per storage in formato open table.

## Codice

Creazione di un producer e di un consumer Kafka con le API Java di base:

```java
KafkaProducer<String, String> producerClient = new KafkaProducer<>(props);
val record = new ProducerRecord<>("my-topic", desiredPartition, "key", "value");
producerClient.send(record);

KafkaConsumer<String, String> c = new KafkaConsumer<>(props);
c.assign(List.of(new TopicPartition("my-topic", 0)));
while (true) {
  ConsumerRecords<String, String> records = c.poll(Duration.ofMillis(100));
}
```

Pipeline Kafka Streams che filtra bot dalle page view, aggrega il traffico per finestre di un minuto e scrive i risultati su un topic di output:

```java
builder.stream("page-views")
  .filter((page, pageView) -> !isBot(pageView))
  .windowedBy(Duration.ofMinutes(1))
  .count()
  .toStream()
  .to("page-traffic-sums");
```
