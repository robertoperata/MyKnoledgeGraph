---
title: Event-Driven Architecture
type: pattern
tags: [thread1-microservices, thread5-ddd]
sources:
  - "[[Microservices Fundamental]]"
  - "[[Microservices Collaboration]]"
  - "[[microservizi_pattern_summary]]"
updated: 2026-04-09
related:
  - "[[concepts/domain-event]]"
  - "[[concepts/independent-deployability]]"
  - "[[patterns/cqrs-read-model]]"
  - "[[patterns/saga-pattern]]"
  - "[[technologies/kafka]]"
---

# Event-Driven Architecture

## Problema che risolve

In un sistema a microservizi, i servizi devono comunicare senza diventare accoppiati. La comunicazione sincrona (request/response) crea **temporal coupling**: entrambi i servizi devono essere up contemporaneamente. L'event-driven rimuove questo accoppiamento temporale.

## Struttura

```
Service A (Publisher)
    │
    ▼
Message Broker (Kafka, RabbitMQ, SQS)
    │
    ▼
Service B, C, D... (Subscribers, indipendenti)
```

La caratteristica fondamentale: **l'intento della collaborazione è nel destinatario**, non nel mittente. Service A pubblica "è successo X"; ciascun subscriber decide autonomamente come reagire.

## Trade-off

| Vantaggio | Svantaggio |
|---|---|
| Basso accoppiamento spaziale e temporale | Difficile tracciare il flusso del business process |
| Scalabilità naturale (broker come buffer) | Debugging complesso (no stack trace lineare) |
| Resilienza: il publisher non sa degli subscriber | Eventual consistency — i dati sono "freschi" con un ritardo |
| Evoluzione indipendente dei servizi | Gestione del versionamento degli eventi |

## Request/Response vs Event-Driven

Sam Newman (Microservices Collaboration) distingue chiaramente:

**Request/Response**:
- L'intento è nel **mittente** ("dammi X", "fai Y")
- Stili: RPC, REST, gRPC
- Sincrono (o asincrono con broker-based request/response)

**Event-Driven**:
- L'intento è nel **destinatario** ("è successo X, tu decidi cosa fare")
- Decentralizza l'intelligenza del sistema
- Favorisce la choreography rispetto all'orchestration

## Quando preferirlo ad alternative

- Quando più servizi devono reagire allo stesso evento (evita multiple chiamate sincrone)
- Quando il publisher non deve conoscere chi consuma il suo output
- Quando la latenza può essere eventuale (no strong consistency richiesta)
- Per integrare sistemi legacy o third-party senza modificarli

## Pattern correlati

- **[[patterns/cqrs-read-model]]**: i Domain Events alimentano i Read Model (proiezioni locali denormalizzate)
- **[[patterns/saga-pattern]]**: le saga usano eventi per coordinare transazioni distribuite
- **[[patterns/request-reply-correlation-id]]**: event-driven applicato al pattern request/response (via Kafka) per evitare il temporal coupling

## Event-driven monolith come step intermedio

L'event-driven non richiede necessariamente servizi separati. Un **event-driven monolith** (es. Revolut 2016-2017) ha moduli interni disaccoppiati tramite eventi ma rimane un unico deployment. Vantaggi:
- Elimina le dipendenze cicliche tra service class (service spaghetti)
- Prepara la codebase alla separazione in servizi: i moduli sono già isolati
- Mantiene la semplicità operativa del monolite

È lo step naturale prima di separare i moduli in servizi indipendenti. Vedere anche [[patterns/modular-monolith]].

## Queue vs Event Stream

Distinzione importante spesso confusa (Sookocheff):

| Aspetto | Coda | Event Stream |
|---|---|---|
| Consumer | Singolo per messaggio | Multipli (tutti ricevono tutti gli eventi) |
| Replay | No | Sì (log durevole) — a differenza del pub-sub classico |
| Ownership | Consumer locale | Infrastruttura condivisa |
| Uso ideale | Load leveling, task queue | Fan-out, event sourcing, CEP |

La coda appartiene al **servizio consumatore** (non è un bus globale): questo evita single point of failure e permette backpressure indipendente per ogni servizio.

## Connessioni

- [[concepts/domain-event]] — gli eventi di dominio sono l'unità fondamentale dell'event-driven
- [[technologies/kafka]] — message broker spesso usato per l'event-driven in ambienti microservizi
- [[concepts/independent-deployability]] — l'event-driven favorisce il disaccoppiamento necessario per i deploy indipendenti
- [[concepts/sync-async-hybrid-architecture]] — principio di usare REST ai confini e async internamente; le code e gli event stream hanno ruoli distinti
- [[concepts/dual-write-model]] — pattern che combina event log (per event-driven) e stato corrente (per query veloci) nella stessa operazione di write
