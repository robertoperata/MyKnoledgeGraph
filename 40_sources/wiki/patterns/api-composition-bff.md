---
title: API Composition / BFF
type: pattern
tags: [thread1-microservices]
sources:
  - "[[microservizi_pattern_summary]]"
updated: 2026-04-09
related:
  - "[[concepts/completable-future]]"
  - "[[patterns/cqrs-read-model]]"
  - "[[concepts/independent-deployability]]"
---

# API Composition / Backend For Frontend (BFF)

## Problema che risolve

Un client (web, mobile, third-party) ha bisogno di dati aggregati da più microservizi. Invece di far fare al client N chiamate separate, un livello intermedio (BFF) aggrega e compone le risposte.

## Struttura

```
Client ──► BFF
              ├──► Service A
              ├──► Service B
              └──► Service C
              
         aggrega e compone
              │
              └──► risposta unificata al client
```

I microservizi non si conoscono tra loro. Il BFF puro è **stateless**: non ha un database proprio.

## BFF puro: implementazione

```java
public OrderDetailResponse getOrderDetail(String orderId) {
    // Chiamate parallele — tempo totale = max(A, B), non A + B
    CompletableFuture<Order> orderFuture =
        CompletableFuture.supplyAsync(() -> orderClient.getOrder(orderId));
    CompletableFuture<Customer> customerFuture =
        CompletableFuture.supplyAsync(() -> customerClient.getCustomer(orderId));

    CompletableFuture.allOf(orderFuture, customerFuture).join();
    return compose(orderFuture.get(), customerFuture.get());
}
```

## BFF + Read Model: la variante avanzata

Il BFF avanzato combina API composition con un [[patterns/cqrs-read-model|read model locale]]:
- Un Projector Kafka mantiene aggiornati i dati locali
- Le richieste leggono dal database locale (latenza zero)
- Resiliente ai downtime dei servizi dipendenti

```
BFF
├── API Layer (routing, auth)
├── Aggregator (CompletableFuture per chiamate REST)
└── Read Model (Projector Kafka + DB locale)
```

## Trade-off

| | BFF Puro | BFF + Read Model |
|---|---|---|
| Implementazione | Semplice | Complessa |
| Latenza | Dipende dai servizi | Zero (locale) |
| Consistenza | Forte | Eventuale |
| Resilienza ai downtime | Bassa | Alta |

## Quando usarlo

- Aggregazione di dati per client con esigenze diverse (mobile vs web vs third-party)
- Quando i microservizi hanno già API REST ben definite
- Per adattare il formato dei dati senza modificare i servizi di dominio

## Rischio principale

Il BFF tende ad accumulare logica di dominio nel tempo. Serve disciplina: il BFF orchestra, non decide. La logica di business deve rimanere nei servizi di dominio.

## Il BFF come confine REST del sistema

Il BFF non è solo un aggregatore: è il **punto di stabilità verso l'esterno** nell'architettura ibrida sync/async. Mentre internamente i microservizi comunicano in modo asincrono, il BFF espone un'API REST stabile e ben documentata verso client e sistemi terzi.

Varianti possibili per lo stesso sistema: BFF browser, BFF mobile, BFF third-party — ognuno adatta il contratto REST alle esigenze del proprio consumer senza modificare i servizi di dominio.

## Connessioni

- [[concepts/completable-future]] — implementazione Java dell'aggregazione parallela
- [[patterns/cqrs-read-model]] — variante avanzata con read model locale nel BFF
- [[concepts/independent-deployability]] — il BFF è un servizio separato deployabile indipendentemente
- [[concepts/sync-async-hybrid-architecture]] — il BFF è il layer REST ai confini del sistema; internamente usa comunicazione asincrona verso i microservizi
