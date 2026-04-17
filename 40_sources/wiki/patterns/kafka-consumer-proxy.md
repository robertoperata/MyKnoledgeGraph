---
title: Kafka Consumer Proxy
type: pattern
tags: [thread1-microservices]
sources:
  - "[[infoq_uber-uforwarder-kafka-push-proxy]]"
updated: 2026-04-17
related:
  - "[[technologies/kafka]]"
  - "[[patterns/event-driven]]"
  - "[[concepts/independent-deployability]]"
---

# Kafka Consumer Proxy

## Problema che risolve

A scala (1.000+ servizi consumer, trilioni di messaggi/giorno), i consumer group Kafka tradizionali presentano: gestione complessa delle partizioni, supporto linguistico inconsistente, overhead operativo per ogni servizio (offset handling, retry, delay). Ogni team reimplementa la stessa logica infrastrutturale.

## Struttura

```
Kafka Topics
    │
    ▼
Consumer Proxy (uForwarder)
    │  - offset management centralizzato
    │  - retry logic
    │  - dead letter queue
    │  - backpressure
    ▼
gRPC push → Service A
         → Service B
         → Service C
```

## Implementazione: uForwarder (Uber)

| Feature | Descrizione |
|---|---|
| **Context-Aware Routing** | Header Kafka propagano metadati di routing nelle chiamate gRPC — decisioni infrastrutturali senza filtering applicativo |
| **Out-of-Order Commit Tracker** | Messaggi problematici → dead letter queue, commit pointer avanza → nessuno stallo della partizione |
| **Consumer Auto Rebalancer** | Ridistribuzione continua delle partizioni in base a CPU, memoria, throughput |
| **DelayProcessManager** | Pausa/ripresa a livello di partizione — solo le partizioni bloccate vanno in buffer |

## Quando preferirlo

- Molti servizi consumer con logica infrastrutturale comune da centralizzare
- Multi-linguaggio: i servizi espongono gRPC, non devono conoscere Kafka
- Backpressure granulare a livello di partizione richiesta
- Necessità di isolamento del workload tra consumer

## Trade-off

| Vantaggio | Svantaggio |
|---|---|
| Logica infrastrutturale centralizzata | Strato aggiuntivo nella pipeline |
| Supporto multi-linguaggio via gRPC | Latenza aggiuntiva del proxy |
| Backpressure granulare | Single point of failure se non replicato |
| Dead letter queue integrata | Complessità operativa del proxy stesso |

## Connessioni

- [[technologies/kafka]] — il proxy si interpone tra Kafka e i servizi consumer
- [[patterns/event-driven]] — il consumer proxy è un'implementazione infrastrutturale dell'EDA
- [[concepts/independent-deployability]] — i servizi sono disaccoppiati da Kafka tramite interfaccia gRPC stabile
