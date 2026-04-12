---
title: Apache Kafka
type: technology
tags: [thread1-microservices]
sources:
  - "[[microservizi_pattern_summary]]"
updated: 2026-04-09
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

## Connessioni

- [[patterns/event-driven]] — Kafka è il broker più comune per l'event-driven
- [[patterns/cqrs-read-model]] — Kafka trasporta gli eventi che alimentano i read models
- [[patterns/request-reply-correlation-id]] — Kafka per request/response async con correlation ID
- [[concepts/domain-event]] — Kafka come transport per i domain events
