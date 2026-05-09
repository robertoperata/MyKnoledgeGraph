---
title: Transactional Outbox Pattern
type: pattern
tags:
  - thread1-microservices
sources:
  - "[[chris_richardson_not-just-events-async-microservices-goto2019]]"
  - "[[spring_transactional-declarative-annotations]]"
updated: 2026-05-09
related:
  - "[[patterns/saga-pattern]]"
  - "[[patterns/event-driven]]"
  - "[[concepts/domain-event]]"
  - "[[technologies/kafka]]"
  - "[[concepts/spring-transactional]]"
---

# Transactional Outbox Pattern

## Problema che risolve

In un'architettura a microservizi basata su messaggistica, ogni passo di una Saga (o ogni evento domain) deve:
1. Aggiornare il database locale
2. Pubblicare un messaggio/evento al message broker

Questi due passi devono essere **atomici** — o entrambi avvengono, o nessuno. Non si può usare il 2-Phase Commit (esattamente quello che si cerca di evitare nei microservizi). Le soluzioni naive falliscono:
- **Publish first, then update DB**: il servizio non può leggere le proprie scritture nella stessa transazione
- **Update DB, then publish**: se il processo crasha tra i due passi, il messaggio non viene mai pubblicato → inconsistenza permanente

## Struttura

```sql
-- Il servizio scrive nella stessa transazione locale:
BEGIN TRANSACTION;
  UPDATE orders SET status = 'APPROVED' WHERE id = ?;
  INSERT INTO outbox (aggregate_type, aggregate_id, type, payload)
         VALUES ('Order', ?, 'OrderApproved', ?);
COMMIT;
```

Un processo separato legge la tabella `outbox` e pubblica i messaggi al broker. I messaggi già pubblicati vengono marcati come tali (o eliminati).

```
Business Service                    Message Relay
      │                                  │
      │ BEGIN TX                         │
      │ UPDATE entity                    │
      │ INSERT INTO outbox               │
      │ COMMIT TX                        │
      │                                  │
      │                          SELECT FROM outbox
      │                          → pubblica su Kafka
      │                          → mark as published
```

**Risultato**: l'atomicità è garantita dalla transazione ACID locale. Il servizio può leggere e scrivere nella stessa transazione.

## Due approcci per leggere l'outbox

| Approccio | Come funziona | Pro | Contro |
|---|---|---|---|
| **Transaction Log Tailing** | Legge il WAL/binlog del DB (Postgres WAL, MySQL binlog, DynamoDB Streams) tramite tool come Debezium | Zero latency, efficiente, nessun polling | Database-specific; richiede accesso al WAL |
| **Polling Publisher** | `SELECT * FROM outbox WHERE published = false` ogni N ms | Universale (qualsiasi JDBC DB) | Polling frequency trade-off; overhead query |

Entrambi possono pubblicare lo stesso messaggio due volte in caso di crash del relay → i message handler devono essere **idempotenti**.

## Trade-off

| Vantaggio | Svantaggio |
|---|---|
| Atomicità garantita senza 2PC/XA | Aggiunge complessità: tabella outbox + processo relay |
| Funziona con qualsiasi RDBMS | Overhead di storage (tabella outbox) |
| Idempotency gestibile | I consumer devono essere idempotenti (comunque necessario in sistemi distribuiti) |
| Supporta at-least-once delivery naturalmente | Latenza aggiuntiva (il relay introduce un delay rispetto alla scrittura) |

## Quando preferirlo ad alternative

- **Sempre**, quando si usa una Saga o si emettono domain events in un microservizio con database relazionale/SQL
- **Event Sourcing** è un'alternativa che risolve lo stesso problema in modo diverso: persiste gli eventi direttamente nell'event store — non serve l'outbox perché gli eventi sono sia la fonte di verità sia il meccanismo di pubblicazione

## Relazione con Event Sourcing

> **Tensione:** Event Sourcing risolve lo stesso problema (atomicità DB + pubblicazione) in modo elegante: l'event store è sia la fonte di verità sia la tabella outbox. Vantaggio: nessuna duplicazione. Svantaggio: approccio radicalmente diverso di programmazione, schema evolution complessa, forza CQRS.

## Connessioni

- [[patterns/saga-pattern]] — ogni passo di una Saga usa l'outbox per garantire l'invio del messaggio successivo
- [[concepts/domain-event]] — l'outbox è il meccanismo per pubblicare affidabilmente i domain events
- [[technologies/kafka]] — Kafka è il broker target dell'outbox relay
- [[patterns/event-driven]] — il Transactional Outbox è la fondazione tecnica dell'event-driven nei microservizi
- [[concepts/spring-transactional]] — l'outbox richiede che l'INSERT nella tabella outbox sia nella stessa transazione @Transactional della scrittura dell'entity; attenzione: checked exception non causano rollback di default
