---
title: Apache Kafka
type: technology
tags: [thread1-microservices]
sources:
  - "[[microservizi_pattern_summary]]"
  - "[[infoq_uber-uforwarder-kafka-push-proxy]]"
  - "[[infoq_multi-cloud-event-driven-architectures]]"
  - "[[getkafkanated_kip-881-kip-392-inter-az-kafka-costs]]"
updated: 2026-05-08
related:
  - "[[patterns/event-driven]]"
  - "[[patterns/cqrs-read-model]]"
  - "[[patterns/request-reply-correlation-id]]"
  - "[[concepts/domain-event]]"
---

# Apache Kafka

## Ruolo nell'architettura

Kafka è il message broker usato come substrate per l'event-driven architecture nei microservizi. Permette di disaccoppiare temporalmente publisher e subscriber: il publisher non deve sapere chi consuma, il subscriber non deve essere up quando il publisher scrive.

## Usi nei pattern

### Request-Reply con Correlation ID
Kafka usato per simulare request/response async:
- Topic di request: il servizio richiedente pubblica
- Topic di response: i servizi rispondenti pubblicano le risposte
- Il Correlation ID collega request e response

### CQRS + Read Model
Kafka come backbone di sincronizzazione:
- Il servizio owner pubblica eventi su un topic
- Il servizio consumer legge da quel topic per aggiornare il suo read model locale
- Permette replay degli eventi per ricostruire il read model da zero

### 202 Accepted + Webhook/SSE
Kafka come coda per il processing asincrono:
```java
@PostMapping("/compute")
public ResponseEntity<JobResponse> startComputation(@RequestBody Input input) {
    String jobId = UUID.randomUUID().toString();
    jobRepository.save(new Job(jobId, PENDING));
    eventPublisher.publish(new ComputationRequestedEvent(jobId, input));
    return ResponseEntity.accepted().body(new JobResponse(jobId));
}
```

## Integrazione Java (Spring)

```java
// Publisher
@Autowired KafkaTemplate<String, Object> kafkaTemplate;
kafkaTemplate.send("topic.request", new RequestMessage(correlationId, input));

// Consumer
@KafkaListener(topics = "topic.response")
public void onResponse(ResponseMessage msg) {
    // elaborazione...
}
```

## Nota dal corso ML Foundations

Il file `The Essential Learning Foundations.md` contiene una domanda interessante:
> "In un microservizio con stream application che fa join di più topic su una KTable, quando l'applicazione scala up, tutte le istanze continuano a leggere da tutte le partizioni?"

Questo è un aspetto avanzato della Kafka Streams topology che non è coperto in dettaglio dalle sorgenti disponibili. **Lacuna identificata**: Kafka Streams e KTable non sono coperti dalla knowledge base.

## Consumer Proxy Pattern (uForwarder, Uber)

A scala (1.000+ consumer, trilioni di messaggi/giorno), i consumer group tradizionali hanno limitazioni strutturali. Il **Consumer Proxy** centralizza offset management, retry, dead letter queue e backpressure, esponendo un'interfaccia gRPC ai servizi consumer.

Vedi [[patterns/kafka-consumer-proxy]] per il dettaglio.

## Ottimizzazione Multi-Cloud

Per deployment Kafka cross-cloud, la configurazione del producer impatta significativamente la latenza:
- Compressione Snappy riduce la banda
- Batching con LingerMs: -40-60% di latenza inter-cloud
- Partitioning per account per ordinamento intrinseco

Vedi [[patterns/multi-cloud-event-driven]] per il framework completo.

## Ottimizzazione costi inter-AZ (KIP-392 + KIP-881)

Il traffico dati tra availability zone è la **componente di costo dominante** di un cluster Kafka su cloud. Con il comportamento di default su un cluster a 3 AZ:
- ~2/3 delle fetch dei consumer finisce su broker in zone diverse
- Tutto il traffico di replicazione attraversa zone (= 2× il traffico producer con RF=3)

**Esempio concreto:** 10 MB/s write + 5× fanout → **$36.9k/anno** su AWS, di cui $20.5k solo i consumer read.

### KIP-392 — Fetch From Follower (Kafka 2.4+)

Permette ai consumer di leggere dai follower nella propria AZ invece che sempre dal leader. Sicuro perché i dati sotto l'*high watermark* sono già replicati su tutti i follower ISR.

```properties
# Consumer (e Connect Worker, MirrorMaker2)
client.rack=eu-west-1a

# Broker
replica.selector.class=org.apache.kafka.common.replica.RackAwareReplicaSelector
broker.rack=eu-west-1a  # già spesso configurato
```

### KIP-881 — Rack-Aware Partition Assignment (Kafka 3.5+)

Risolve due problemi che KIP-392 non affronta:
1. **Sbilanciamento broker**: se i follower non sono distribuiti uniformemente, fetch-from-follower può sovraccaricare alcuni broker
2. **RF < numero AZ**: con 5 AZ e RF=3, le AZ 4-5 non hanno repliche locali → KIP-881 assegna ai consumer solo partizioni con replica nella propria AZ

Supporto per protocollo consumer group:
- **v1 (classic)**: assignor di default gestiscono rack come criterio secondario ✅
- **v2 (KIP-848)**: non ancora supportato; tracciato in KAFKA-19387 ❌

### ⚠️ Gotcha AWS: IP pubblici

Traffico nella stessa AZ con **IP pubblici IPv4** viene fatturato come cross-zona. Per risparmiare davvero: usare IP privati (same VPC, VPC peering, o Private Link).

**Risparmio atteso:** ~50% del costo del cluster su workload con fanout elevato.

## Kafka NON è un event store

> "Kafka is not an event store, despite claims to the contrary." — Chris Richardson

Kafka è eccellente come message broker ma **non** permette di recuperare eventi per ID (requisito fondamentale di un event store per l'Event Sourcing). Per l'Event Sourcing servono store specializzati (EventStoreDB, eventuate.io) che permettono di caricare la sequenza di eventi di un aggregate per ID.

## Transactional Outbox con Kafka

Il [[patterns/transactional-outbox]] è la fondazione per pubblicare affidabilmente su Kafka. Due approcci:
- Transaction Log Tailing: Debezium legge il WAL di Postgres/MySQL binlog e pubblica su Kafka → zero latency
- Polling Publisher: query `SELECT FROM outbox WHERE published=false` → semplice ma con overhead

## Connessioni

- [[patterns/event-driven]] — Kafka è il broker più comune per l'event-driven
- [[patterns/cqrs-read-model]] — Kafka trasporta gli eventi che alimentano i read models
- [[patterns/request-reply-correlation-id]] — Kafka per request/response async con correlation ID
- [[patterns/kafka-consumer-proxy]] — pattern per centralizzare la logica consumer a scala
- [[patterns/multi-cloud-event-driven]] — Kafka ottimizzato per deployment cross-cloud
- [[concepts/domain-event]] — Kafka come transport per i domain events
- [[patterns/transactional-outbox]] — fondazione per la pubblicazione affidabile su Kafka
