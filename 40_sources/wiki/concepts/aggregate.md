---
title: Aggregate
type: concept
tags: [thread1-microservices, thread5-ddd]
sources:
  - "[[Microservices Fundamental]]"
  - "[[Domain-Driven Design Aggregates Domain Events and Value Objects]]"
updated: 2026-04-09
related:
  - "[[concepts/bounded-context]]"
  - "[[concepts/domain-event]]"
  - "[[concepts/value-object]]"
  - "[[concepts/ubiquitous-language]]"
---

# Aggregate

## Definizione

Un Aggregate è una collezione di oggetti di dominio che vengono sempre gestiti come una singola unità transazionale. Ogni Aggregate ha un **Aggregate Root**: l'unico punto di ingresso per modificare lo stato dell'aggregate.

## Come funziona

- Tutte le modifiche a un aggregate avvengono attraverso l'Aggregate Root
- Le invarianti di business dell'aggregate sono mantenute atomicamente (in una singola transazione)
- Gli aggregate si riferiscono ad altri aggregate solo tramite ID, mai per reference diretta
- I **[[concepts/domain-event|Domain Event]]** vengono pubblicati dall'aggregate root per notificare il resto del sistema di cambiamenti rilevanti

## Regole di design (Vaughn Vernon)

1. **Proteggere le invarianti di business dentro un singolo aggregate**: tutto ciò che deve essere consistente in modo forte va nello stesso aggregate.
2. **Progettare aggregate piccoli**: aggregate grandi portano a lock di lunga durata e a problemi di performance.
3. **Riferirsi ad altri aggregate solo per ID**: riduce il coupling e permette la consistency eventuale tra aggregate.
4. **Usare la consistency eventuale tra aggregate**: non tutto deve essere consistente nello stesso momento.

## Connessione con Microservizi

In Microservices Fundamental, Sam Newman introduce il concetto di aggregate nel contesto del DDD applicato ai microservizi: un aggregate è l'unità di transazionalità che aiuta a trovare le boundary del servizio. Se due concetti devono sempre essere consistenti insieme, appartengono allo stesso aggregate (e probabilmente allo stesso servizio).

## Connessioni

- [[concepts/bounded-context]] — ogni aggregate appartiene a un bounded context
- [[concepts/domain-event]] — l'aggregate pubblica domain events quando cambia stato
- [[concepts/value-object]] — gli aggregates contengono value objects per descrivere concetti di dominio
- [[patterns/saga-pattern]] — quando una transazione coinvolge più aggregate (o più servizi), si usa la saga per gestire la consistency eventuale
