---
title: Hexagonal Architecture (Ports & Adapters)
type: concept
tags:
  - thread1-microservices
sources:
  - "[[chris_richardson_cubes-hexagons-triangles-yow2019]]"
updated: 2026-05-07
related:
  - "[[concepts/information-hiding]]"
  - "[[concepts/independent-deployability]]"
  - "[[concepts/bounded-context]]"
  - "[[patterns/event-driven]]"
---

# Hexagonal Architecture (Ports & Adapters)

## Definizione

Stile architetturale proposto da Alistair Cockburn per strutturare il singolo servizio: la **business logic** sta al centro, separata dal mondo esterno tramite **porte** (interfacce) e **adapter** (implementazioni). Nota anche come Ports & Adapters pattern.

## Come funziona

Alternativa all'architettura a 3 layer classica (presentation → business logic → persistence), che mette erroneamente la business logic in dipendenza dalla persistence e non supporta bene multiple presentation/persistence layer.

```
         ┌──────────────────────────────────────┐
         │           Hexagon                    │
         │                                      │
[REST    │  Inbound Port  ──► Business  ──► Outbound Port  │  [DB
Adapter] │  (Java interface)   Logic       (Java interface)  │  Adapter]
         │                                      │
[gRPC    │  Inbound Port  ──►             ──► Outbound Port  │  [Kafka
Adapter] │                                      │  Adapter]
         └──────────────────────────────────────┘
```

**Inbound ports**: interfacce Java invocate dagli adapter in entrata (es. controller REST, consumer Kafka). Definiscono cosa il servizio può fare.

**Outbound ports**: interfacce Java implementate dagli adapter in uscita (es. repository DB, client HTTP verso altri servizi). Definiscono di cosa il servizio ha bisogno.

**Adapter**: classi che bridgiano tra il mondo esterno e le porte. Si possono sostituire senza toccare la business logic.

## L'API del servizio

L'API è la ragione d'essere del servizio. Composta da:

| Tipo | Esempi | Invocazione |
|---|---|---|
| **Commands** | `createOrder`, `reviseOrder`, `cancelOrder` | REST/gRPC (sync) o messaging (async) |
| **Queries** | `findOrder`, `findOrderHistory` | REST/gRPC |
| **Events** | `OrderCreated`, `OrderRevised`, `OrderCancelled` | Consumabili da altri servizi |

## Il principio Iceberg

> "You want to design services that look like icebergs — a small stable API encapsulating a complex implementation." — Chris Richardson

Un buon servizio ha un'API piccola e stabile (sopra la superficie) che nasconde una grande implementazione complessa (sotto la superficie). L'anti-pattern opposto — servizi che wrappano direttamente uno schema database — genera fortissimo design-time coupling.

## Quando usarlo

- Struttura standard per ogni microservizio
- Specialmente quando il servizio ha multiple fonti di input (REST + Kafka) o multiple implementazioni di storage
- Per garantire testabilità: i test possono invocare i port direttamente, senza passare dagli adapter

## Connessioni con altri concetti

- [[concepts/information-hiding]] — le porte sono il meccanismo per nascondere l'implementazione
- [[concepts/independent-deployability]] — sostituire un adapter non richiede cambi ad altri servizi
- [[concepts/bounded-context]] — ogni microservizio è un bounded context; l'esagono ne definisce i confini tecnici interni
- [[patterns/event-driven]] — gli eventi emessi dal servizio escono tramite outbound adapter Kafka
- [[concepts/microservice-chassis]] — il chassis implementa le preoccupazioni cross-cutting (tracing, health check, config) che circondano l'esagono
- [[patterns/anti-corruption-layer]] — l'ACL è la stessa filosofia dell'adapter ma applicata a livello inter-sistema: protegge il modello interno dalle strutture di un sistema esterno, come un outbound adapter che traduce tra due modelli di dominio incompatibili
