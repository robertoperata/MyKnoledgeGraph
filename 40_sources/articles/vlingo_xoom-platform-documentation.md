---
tags:
  - architecture
  - ddd
  - microservices
  - event-driven
  - java
  - distributed-systems
  - concurrency
type: article
author: Vaughn Vernon / VLINGO Team
source: https://docs.vlingo.io/
date: 2026-05-05
---

# VLINGO XOOM Platform — Official Documentation Overview

**Collegato a:** `kalele_xoom-open-source-reactive-platform.md` (annuncio della piattaforma) e `kalele_architecture-vs-model.md` (articolo DDD di Vaughn Vernon)

---

## Sunto

La documentazione ufficiale di VLINGO XOOM presenta la piattaforma come risposta alla lentezza delle iniziative di trasformazione digitale e ai budget sforati. Il claim centrale: le implementazioni di architetture event-driven e Domain-Driven Design falliscono troppo spesso perché mancano di strumenti appropriati che guidino dall'Event Storming alla produzione in modo rapido e corretto.

XOOM si posiziona come **"crossroads where business strategy and modern technology meet"**: una piattaforma open-source (JVM e .NET) che implementa il *Reactive Manifesto* attraverso l'Actor Model, offrendo reactive HTTP, distributed grid, CQRS, event sourcing, schema registry e streaming — tutto in un ecosistema integrato orientato alla semplicità.

> "There is far too much complexity in the software industry. The overarching vision for the VLINGO XOOM platform puts extreme emphasis on simplicity, efficiency, rapid delivery."

Il modello di concorrenza è inusuale: invece del multi-threading tradizionale (complesso e error-prone), XOOM usa un **modello basato su oggetti single-threaded** dove ogni oggetto è eseguito su un singolo thread, ma il processo complessivo è massivamente concorrente e parallelo — tramite il sistema di Actor Model (XOOM Actors).

---

## Componenti della Piattaforma

### Architettura a layer

```
┌─────────────────────────────────────────────────┐
│              XOOM Designer                      │  ← Visual design, EventStorming, code gen
├─────────────────────────────────────────────────┤
│   XOOM HTTP  │  XOOM GraphQL  │  XOOM Streams   │  ← I/O layer
├─────────────────────────────────────────────────┤
│         XOOM Lattice (Aggregates, Sagas)        │  ← Domain model distribution
├─────────────────────────────────────────────────┤
│   XOOM Symbio (Storage)  │  XOOM Schemata       │  ← Persistence + Schema Registry
├─────────────────────────────────────────────────┤
│              XOOM Actors (Runtime)              │  ← Reactive foundation
└─────────────────────────────────────────────────┘
```

---

### XOOM Actors — Il Foundation Runtime

- Implementazione dell'Actor Model
- Messaggistica asincrona type-safe ad alta scalabilità
- Underlying architecture di tutti gli altri componenti XOOM
- Ogni attore è single-threaded, il sistema è massivamente concorrente

### XOOM HTTP

- Reactive HTTP server non-bloccante con massive concurrent request scaling
- Fluent API + configuration-based request mapping
- **XOOM annotations** per auto-dispatch ai domain model
- **Server-Sent Events (SSE)** per feed event-driven continui via REST log resources

### XOOM Lattice — Distributed Domain Model

Il componente più importante per DDD e microservizi:

| Elemento supportato | Descrizione |
|---|---|
| **Aggregates / Entities** | Domain model distribuito across locations |
| **Processes / Sagas** | Long-running business processes |
| **Data Tuples** | Dati key-value distribuiti |
| **Message Exchanges** | Comunicazione inter-service |
| **Stateless Services** | Servizi senza stato |
| **State Projections** | Read model (CQRS) |

- *Opaque message routing*: i messaggi vengono instradati agli oggetti specifici senza che il mittente sappia dove si trovano
- Persistence multipla: Key-Value, Event Sourcing, Object Mapped
- Resilient cluster management

### XOOM Symbio — Storage

| Storage Type | Backing Databases |
|---|---|
| Key-Value State Storage | DynamoDB, Postgres, MySQL, MariaDB, YugaByte |
| Event Sourcing | DynamoDB, Postgres, MySQL, MariaDB, YugaByte |
| Object Mapped | DynamoDB, Postgres, MySQL, MariaDB, YugaByte |

- Non-blocking, async throughput
- Configurabile via XOOM Designer

### XOOM Schemata — Schema Registry

- Registry degli schemi per sistemi e servizi distribuiti
- Pubblica **Published Languages** (tipi standard condivisi) tra Bounded Context
- **Previene il forte accoppiamento tra microservizi** tramite versioning degli schemi
- Chiave per implementare sistemi event-driven correttamente

### XOOM Streams

- Implementazione del **Reactive Streams standard**
- Streaming di collezioni dati in-memory e persistite
- Streaming di risultati di query database
- **Back-pressure** mechanism per bilanciare publisher e subscriber
- Stream composition

### XOOM GraphQL

- HTTP-based GraphQL query server
- Async + parallel query execution
- Non-blocking client-side interface
- Ottimizzato per query aggreganti massicce

### XOOM Designer

- Visual design tool per architetture DOMA e DDD
- Supporta **EventStorming** collaborativo → genera codice funzionante in < 30 minuti
- Genera: infrastruttura, domain model, microservizi
- Include **XOOM Turbo** (quick-booting container) per deployment immediato
- Non richiede esperienza previa con l'SDK

---

## Reactive Architecture

XOOM si basa sui principi del **Reactive Manifesto**:

```
Responsive Value
  in the form of
Elasticity + Resiliency
  by means of
Message-Driven Foundation
```

La piattaforma fornisce:
- **Distributed process choreography and orchestration**
- **Type- and version-safe event broadcasting**
- **Peer-to-peer messaging**
- **CQRS** con database separati per read e write
- **Data grid distribution** across compute clusters

---

## Modello di Concorrenza (XOOM Actors)

Il multi-threading tradizionale è complesso e error-prone, specialmente con moderne CPU con ottimizzazioni (out-of-order execution, branch prediction, ecc.).

**Soluzione XOOM**: ogni oggetto è eseguito su un singolo thread. L'esecuzione parallela avviene a livello di sistema, non di oggetto. Questo elimina la necessità di synchronized, lock, volatile, ecc. nel codice di dominio.

Questo è lo stesso principio descritto nel blog post `kalele_xoom-open-source-reactive-platform.md` (Actor Model implementation in XOOM/ACTORS).

---

## Workflow: da EventStorming a Produzione

```
1. EventStorming collaborativo con XOOM Designer
        │
        ▼
2. Visual modeling dell'architettura
        │
        ▼
3. Code generation automatica (microservizi, infrastruttura, domain model)
        │
        ▼
4. Deployment con XOOM Turbo (container quick-boot)
        │
        ▼
< 30 minuti da EventStorming a microservizi funzionanti >
```

---

## Querying the Docs via API

La documentazione supporta query in linguaggio naturale:

```
GET https://docs.vlingo.io/master.md?ask=<domanda>
```

Es: `GET https://docs.vlingo.io/master.md?ask=How do I implement event sourcing?`

---

## Link esterni

- [Reactive Manifesto](https://www.reactivemanifesto.org/) — fondamento teorico della reactive architecture su cui si basa XOOM
- [VLINGO GitHub](https://github.com/vlingo) — tutti i repository open-source della piattaforma
- [Kalele Books — DDD](https://kalele.io/books/) — libri DDD di Vaughn Vernon (IDDD, DDD Distilled, Strategic Monoliths & Microservices)
- [Domain-Oriented Microservices Architecture (Uber)](https://eng.uber.com/microservice-architecture/) — DOMA, framework architetturale di riferimento supportato da XOOM

---

## Immagini

Immagini salvate localmente in `vlingo_xoom-docs_images/`.

- ![XOOM Platform Ecosystem](vlingo_xoom-docs_images/xoom-ecosystem.png) — L'ecosistema completo della piattaforma XOOM per la digital transformation

- ![XOOM DDD + EventStorming](vlingo_xoom-docs_images/xoom-ddd-eventstorming.png) — Come XOOM facilita DDD con architetture event-driven usando choreography e orchestration

- ![Reactive: Value, Form, Means](vlingo_xoom-docs_images/reactive-value-form-means.png) — Il triangolo reactive: valore (responsiveness), forma (elasticity + resiliency), mezzo (message-driven)

- ![EventStorming to Microservices in < 30min](vlingo_xoom-docs_images/eventstorming-to-microservices.png) — Pipeline da EventStorming a visual modeling a microservizi funzionanti in meno di 30 minuti

- ![XOOM Components in Layers](vlingo_xoom-docs_images/xoom-components-layers.png) — I componenti primari della piattaforma XOOM mostrati come layer di container leggeri
