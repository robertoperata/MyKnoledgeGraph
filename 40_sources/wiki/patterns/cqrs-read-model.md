---
title: CQRS + Read Model
type: pattern
tags:
  - thread1-microservices
  - thread5-ddd
sources:
  - "[[microservizi_pattern_summary]]"
  - "[[chris_richardson_not-just-events-async-microservices-goto2019]]"
updated: 2026-05-07
related:
  - "[[concepts/domain-event]]"
  - "[[patterns/event-driven]]"
  - "[[technologies/kafka]]"
  - "[[patterns/transactional-outbox]]"
---

# CQRS + Read Model

## Problema che risolve

Un microservizio ha bisogno di aggregare dati che appartengono ad altri domini. Le chiamate sincrone ai servizi owner ad ogni richiesta creano latenza e dipendenza temporale. Il CQRS + Read Model elimina le chiamate remote a runtime mantenendo una copia locale denormalizzata.

## Struttura

```
Write side (owner del dato)    │    Read side (consumer)
                               │
Service A ──pubblica──► Kafka ──consume──► Projector ◄── Client request
owner dato   evento           │            │
                               │            ▼
DB Service A                   │   Read Model DB
(normalizzato)                 │   (denormalizzato)
fonte di verità                │   latenza zero
```

## CQRS — la separazione fondamentale

Command Query Responsibility Segregation separa:
- **Commands (Write side)**: modificano lo stato, non restituiscono dati
- **Queries (Read side)**: restituiscono dati, non modificano lo stato

In un sistema a microservizi: il servizio **owner** del dato (write side) e il servizio **consumer** (read side) sono servizi diversi. La sincronizzazione avviene via eventi.

## Il Read Model: cosa significa denormalizzato

Il read model non è progettato per eliminare la ridondanza, ma per soddisfare esattamente la query che il microservizio deve eseguire:

```java
// Read model ottimizzato per business logic (pricing)
public class CustomerPurchaseProfile {
    String customerId;
    int totalOrdersLast90Days;
    BigDecimal totalSpentLast12Months;
    String customerTier; // BRONZE, SILVER, GOLD
}
```

Tutti i dati necessari già aggregati in un documento. La query di business diventa una lettura locale banale.

## Il Projector

Il projector è il componente che mantiene aggiornato il read model ascoltando gli eventi:

```java
@KafkaListener(topics = "orders.events")
public void onOrderEvent(OrderEvent event) {
    if (event.getType() == ORDER_COMPLETED) {
        CustomerPurchaseProfile profile = repo.findByCustomerId(event.getCustomerId())
            .orElse(new CustomerPurchaseProfile());
        profile.incrementOrderCount();
        profile.addToTotalSpent(event.getTotalAmount());
        profile.recalculateTier();
        repo.save(profile);
    }
}
```

## Trade-off

| Vantaggio | Svantaggio |
|---|---|
| Latenza zero a runtime — lettura locale | **Eventual consistency** — dato leggermente in ritardo |
| Isolamento da downtime degli altri servizi | Complessità del projector e bootstrap iniziale |
| Scalabilità elevatissima in lettura | Storage aggiuntivo per ogni read model |
| Adatto per UI e business logic | Richiede strategia di replay eventi in caso di bug |

## Quando usarlo

- Letture frequenti di dati che appartengono ad altri domini
- Business logic che non richiede dati freschi al millisecondo (pricing, raccomandazioni, fraud detection)
- Per isolare il microservizio da downtime temporanei dei servizi dipendenti

## Tensione con Request-Reply

> **Tensione:** [[patterns/request-reply-correlation-id]] garantisce strong consistency ma introduce latenza e fragilità; CQRS + Read Model offre latenza zero e resilienza ma introduce eventual consistency.
>
> La scelta dipende da: quanto devono essere freschi i dati, quanto può aspettare il chiamante. In sistemi reali coesistono: CQRS per i dati letti frequentemente, Request-Reply per i pochi dati che devono essere freschi al momento della computazione.

## API Composition come alternativa semplice

Per query semplici, un **API Composer** (es. API Gateway) invoca i servizi coinvolti e assembla i risultati in memoria (sincrono o async). Funziona bene per join su dataset piccoli.

**Limiti**: query complesse (es. "clienti recenti con ordini > $X già spediti") richiedono join tra dataset potenzialmente grandi → inefficiente o impossibile con API Composition.

## I view store sono eliminabili e ricostruibili

> "The views in CQRS are disposable. They're replicas. If you need to change the schema, just throw them away and rebuild from scratch." — Chris Richardson

Il view store può usare il database più adatto al pattern di query:
- MongoDB per documenti JSON
- Elasticsearch per full-text search
- Neo4j per grafi
- Redis per cache ad alta performance

## Replication Lag (trade-off principale)

C'è un ritardo (tipicamente decine di ms) tra l'aggiornamento del command side e il view side. Un client che aggiorna e poi subito rilegge potrebbe ottenere dati vecchi. Soluzione lato UI: aggiornare il modello client-side ottimisticamente dopo una command request riuscita.

## Connessioni

- [[concepts/domain-event]] — i domain events alimentano il projector del read model
- [[patterns/event-driven]] — il read model è costruito tramite event-driven architecture
- [[technologies/kafka]] — il broker che trasporta gli eventi dal write side al read side
- [[patterns/api-composition-bff]] — il BFF avanzato usa un read model locale invece delle chiamate REST
- [[patterns/transactional-outbox]] — gli eventi che alimentano il read model vengono pubblicati affidabilmente tramite outbox
- [[concepts/dual-write-model]] — pattern complementare: scrive sia nell'event log che nello stato corrente nella stessa operazione; è il meccanismo di sincronizzazione intra-servizio che alimenta il read side del CQRS
