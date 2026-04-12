---
title: Domain Event
type: concept
tags: [thread1-microservices, thread5-ddd]
sources:
  - "[[Domain-Driven Design Aggregates Domain Events and Value Objects]]"
  - "[[Microservices Collaboration]]"
  - "[[microservizi_pattern_summary]]"
updated: 2026-04-09
related:
  - "[[concepts/aggregate]]"
  - "[[concepts/bounded-context]]"
  - "[[patterns/event-driven]]"
  - "[[patterns/cqrs-read-model]]"
  - "[[patterns/saga-pattern]]"
---

# Domain Event

## Definizione

Un Domain Event rappresenta qualcosa di significativo che è **accaduto** nel dominio. È sempre nominato al passato ("OrderPlaced", "CustomerActivated", "PaymentReceived"). Descrive un fatto, non un'intenzione.

## Come funziona

Gli eventi di dominio:
1. Vengono **pubblicati dall'Aggregate Root** quando il suo stato cambia in modo significativo
2. Sono **immutabili**: descrivono qualcosa che è già accaduto, non può essere cambiato
3. Trasportano i **dati** necessari ai consumer per reagire, senza richiede chiamate aggiuntive
4. Permettono la **comunicazione tra Bounded Context** senza accoppiamento diretto

## Struttura tipica

```java
public record OrderPlaced(
    String orderId,
    String customerId,
    List<OrderItem> items,
    BigDecimal totalAmount,
    Instant occurredAt
) {}
```

## Comunicazione tra Bounded Context

In un'architettura a microservizi, i Domain Event sono il meccanismo principale per comunicare tra bounded context. L'owner del dato pubblica gli eventi; gli altri servizi consumano e reagiscono.

Questo pattern è alla base del **[[patterns/cqrs-read-model|CQRS + Read Model]]** descritto nel blog post sui microservizi: il Consumer ascolta gli eventi Kafka pubblicati dai servizi owner per costruire una proiezione locale denormalizzata.

## Intent vs Event

**Microservices Collaboration** fa una distinzione fondamentale tra stili di comunicazione:
- **Request/Response**: l'intento della collaborazione è nel *mittente*. "Fai questo."
- **Event-Driven**: l'intento della collaborazione è nel *destinatario*. "È successo questo, fai quello che ritieni opportuno."

Questa differenza è cruciale: gli eventi decentralizzano l'intelligenza del sistema.

## Versionamento

A differenza delle API REST (dove si versiona l'endpoint nel suo insieme), gli eventi permettono di evolvere il contratto in modo più granulare. Il **Tolerant Reader Pattern** (citato in Microservices Collaboration) descrive come i consumer degli eventi dovrebbero tollerare cambiamenti che non li riguardano.

## Connessioni

- [[concepts/aggregate]] — l'aggregate root pubblica i domain events
- [[concepts/bounded-context]] — i domain events sono il ponte tra bounded context diversi
- [[patterns/event-driven]] — l'architettura event-driven si basa sui domain events
- [[patterns/cqrs-read-model]] — i domain events alimentano i read models dei servizi consumer
- [[patterns/saga-pattern]] — le saga usano gli eventi come trigger per i passi del workflow distribuito
- [[technologies/kafka]] — spesso usato come transport per i domain events
