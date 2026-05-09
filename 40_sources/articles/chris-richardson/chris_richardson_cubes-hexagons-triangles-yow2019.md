---
tags:
  - microservices
  - architecture
  - event-driven
  - testing
  - devops
type: article
author: Chris Richardson
source: https://www.youtube.com/watch?v=JaXZ2DDS2G0
date: 2019-01-01
---

# Cubes, Hexagons, Triangles & More

**Speaker:** Chris Richardson (autore di *Microservices Patterns*, creatore di eventuate.io)  
**Evento:** YOW! 2019  
**Durata:** 50:01  
**Lingua originale:** inglese

> **Relazione con altri talk**: questo è il talk "overview" di Richardson sull'architettura microservizi. Per i dettagli su transazioni e messaging asincrono vedi `chris_richardson_not-just-events-async-microservices-goto2019.md`.

---

## Sintesi

Richardson costruisce una visione completa dell'architettura microservizi attraverso cinque metafore geometriche: il **triangolo del successo** (processo + organizzazione + architettura), il **cubo della scalabilità** (Y-axis = decomposizione funzionale), **l'esagono** (architettura hexagonal per i singoli servizi), **l'iceberg** (API piccola e stabile su implementazione complessa) e il **cilindro** (messaging channel per disaccoppiamento runtime), chiudendo con la **piramide del testing**.

Il filo conduttore è che la microservice architecture non è un fine ma uno strumento al servizio di tre obiettivi business: continuous delivery/deployment (DevOps), team autonomi e loosely coupled, applicazioni longeve che evolvono nel tempo. L'architettura deve supportare queste tre dimensioni simultaneamente — se non lo fa, è sbagliata per quel contesto.

Richardson introduce l'**iceberg principle**: un buon servizio ha un'API piccola e stabile che nasconde una grande implementazione complessa, non il contrario. Il monolith non è un anti-pattern — può essere la scelta giusta. I problemi emergono quando cresce e la modularità degrada. La migrazione da monolite a microservizi è "hideous and painful" e può richiedere anni.

---

## Il Triangolo del Successo

Per consegnare software rapidamente e frequentemente (DevOps) con team autonomi e applicazioni longeve, servono tre elementi in sinergia:

| Dimensione | Approccio moderno |
|---|---|
| **Processo** | Lean + DevOps (continuous delivery/deployment) |
| **Organizzazione** | Team cross-functional da 6-10 persone (two-pizza teams), loosely coupled |
| **Architettura** | Microservizi (testabilità, deployabilità, modularità, evolvibilità) |

**Conway's Law** attraversa tutte e tre: l'architettura deve essere loosely coupled se i team devono essere loosely coupled. Non è possibile avere team autonomi con un'architettura monolitica tightly coupled.

**Metriche DevOps** per misurare la velocità di consegna:
- **Lead time**: dal commit al deployment in produzione (minimizzare)
- **Deployment frequency**: frequenza dei deployment (massimizzare)
- **Change failure rate**: percentuale di deployment che causano outage (minimizzare)
- **Mean Time to Recovery (MTTR)**: tempo per rilevare, diagnosticare e risolvere un problema (minimizzare)

---

## Il Cubo della Scalabilità (X, Y, Z Axis)

Da *The Art of Scalability* (Abbott & Fisher):

| Asse | Strategia | Descrizione |
|---|---|---|
| **X** | Cloning | Multiple istanze identiche dietro un load balancer |
| **Z** | Partitioning | Load balancer che instrada per attributo della request (es. customer ID) |
| **Y** | **Functional decomposition** | **= Microservices** — split dell'applicazione per funzione/capability |

La Y-axis decomposition è la microservice architecture. Richardson l'ha scoperta leggendo questo libro e capendo che il monolite di Cloud Foundry avrebbe beneficiato enormemente da questo approccio.

### Perché il monolite degrada nel tempo

```
All'inizio: architettura pulita a layer, team di 6 persone
↓ (crescita nel tempo)
Applicazione enorme: team di 100-200 persone tutti sul stesso codebase
↓
Modularity breakdown: "big ball of mud"
↓
Technology lock-in: impossibile adottare nuovi stack
↓
Delivery slowdown: il ritmo di consegna precipita
```

**Il monolite non è un anti-pattern**. È una scelta valida per applicazioni piccole. I problemi emergono con la crescita.

---

## L'Esagono — Struttura del Singolo Servizio

### Architettura Hexagonal (Ports & Adapters)

Alternativa alla classica architettura a 3 layer (presentation → business logic → persistence) che:
- Mette erroneamente la business logic in dipendenza dalla persistence
- Non supporta bene multiple presentation layer o persistence layer

**Hexagonal architecture**:
- Business logic al centro
- **Inbound ports** (interfacce Java): invocati dagli adapter in entrata (es. controller REST)
- **Outbound ports** (interfacce Java): implementati dagli adapter in uscita (es. repository)
- **Adapters**: classi che bridgiano tra il mondo esterno e le porte

### API del Servizio

L'API è la ragione d'essere del servizio. È composta da:

**1. Operations (invocabili):**
- *Commands*: mutano dati (`createOrder`, `reviseOrder`, `cancelOrder`)
- *Queries*: recuperano dati (`findOrder`, `findOrderHistory`)
- Possono essere invocate via REST/gRPC (sincrono) o messaging (asincrono)

**2. Events (emessi):**
- Domain events DDD: `OrderCreated`, `OrderRevised`, `OrderCancelled`
- Consumabili da altri servizi

### Capacità Operative (service "production ready")

| Capability | Descrizione | Esempio |
|---|---|---|
| **Externalized Configuration** | Config dall'esterno, non hardcoded | Env vars, config store |
| **Logging** | Log su stdout in formato standard | Logback/SLF4J → aggregazione centralizzata |
| **Health Check** | Endpoint per runtime orchestration | `/actuator/health` (Kubernetes liveness/readiness probe) |
| **Telemetry/Metrics** | Raccolta e reporting metriche | Prometheus, CloudWatch |
| **Distributed Tracing** | Trace ID propagato tra servizi | Sleuth/Zipkin → visualizzazione del flusso di request |

---

## L'Iceberg — Design Time Coupling

### Il principio

Un buon servizio è un **iceberg**: piccola API stabile sopra la superficie, grande implementazione complessa sotto.

```
            ┌─────────────┐
    API      │  POST /orders│  ← piccola, stabile, ben progettata
            └─────────────┘
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ livello dell'acqua
    Impl     │ Saga, CQRS, │  ← grande, complessa, nascosta
            │ validazioni, │
            │ integr. esterne│
            └─────────────┘
```

**Anti-pattern**: servizi che "wrappano" uno schema database — API che riflette direttamente le tabelle, generata automaticamente da tool. Questi servizi sono "all API, no implementation" e creano fortissimo design-time coupling.

### Due tipi di coupling

| Tipo | Problema | Soluzione |
|---|---|---|
| **Design-time coupling** | Cambio in Service A richiede cambio in Service B (incompatibilità API) | API piccole, stabili, che encapsulano l'implementazione; no shared libraries con business logic; no condivisione di tabelle DB |
| **Runtime coupling** | Service A non può completare una request senza che Service B sia disponibile | Messaggistica asincrona |

**No shared database tables**: ogni servizio ha il suo schema privato. Se Order Service accedesse direttamente alle tabelle di Customer Service, qualsiasi cambio allo schema del Customer Service richiederebbe aggiornamenti in Order Service.

---

## Il Cilindro — Runtime Coupling e Messaggistica Asincrona

### Il problema della comunicazione sincrona

```
Client → POST /orders → Order Service → POST /customers/{id}/reserve → Customer Service
                                              (attende risposta)
```

**Availability = Availability(Order) × Availability(Customer)**

Con N servizi in catena sincrona: `A^N` → crolla rapidamente. Anti-pattern: convertire i moduli del monolite in servizi che comunicano nello stesso modo (sincrono) — si ottiene un "distributed monolith" con tutti i problemi di entrambi gli approcci.

### Soluzioni con messaggistica asincrona

**Saga pattern** (per transazioni):
```
POST /orders → Order Service → risponde subito 202 Accepted
Order Service ←→ Customer Service (messaggi asincroni)
→ ordine eventualmente approvato o rifiutato
```
*Svantaggio*: la risposta iniziale non contiene il risultato dell'approvazione.

**CQRS / Replication** (per query e validazione):
```
Credit limit replicato nel Order Service via eventi da Customer Service
→ Order Service valida ordine localmente senza chiamare Customer Service
→ risposta sincrona con risultato
```
*Svantaggio*: costo e complessità della replicazione.

**Regola pratica**: per gli *update* evitare sempre comunicazione sincrona tra servizi. Per le *query* può funzionare.

---

## La Piramide del Testing

### Il concetto

```
        ╱─────╲         End-to-end tests: lenti, fragili, costosi
       ╱───────╲        Integration/Contract tests
      ╱─────────╲       Component tests
     ╱───────────╲      Unit tests: veloci, affidabili, economici
```

Più scendiamo nella piramide, più i test sono:
- Veloci da eseguire
- Affidabili (non falliscono casualmente)
- Semplici da scrivere
- Economici

**Obiettivo**: eliminare o minimizzare i test end-to-end. La soluzione per il confine tra servizi: **consumer-driven contract testing** (Spring Cloud Contract).

### Testing senza automated test = disastro

Il 72% delle aziende non fa automated testing estensivo. Il 90% vuole fare DevOps/Agile (che richiedono testing). Questo gap è il motivo per cui molte iniziative microservizi falliscono.

### Canary deployment / Testing in Production

Separazione tra **deployment** (codice in produzione) e **release** (traffico instradato):

```
v1 ─────────────────────────────────── 100% traffico
v2 (deployed, no traffic) → test interni → 5% traffico → 20% → 100%
                                              ↕
                              monitoring automatico (latency, 500s)
                                              ↕
                             rollback automatico se soglie superate
```

Tool: service mesh (Istio), traffic routing intelligente, monitoring integrato.

---

## Citazioni notevoli

> "I travel around the world and I actually spend a whole bunch of time going 'no, just use a monolith, it's okay.' A monolith can be highly testable, highly deployable, highly maintainable. It can work just fine."  
> — Chris Richardson

> "You want to design services that look like icebergs — a small stable API encapsulating a complex implementation. You want to avoid services that are all API and no implementation."  
> — Chris Richardson

> "If you attempt to do microservices without automated testing, it is self-defeating and it is risky. 72% of companies do not extensively do automated testing, and ironically 90% want to do DevOps — there's a total disconnect."  
> — Chris Richardson

> "Taking your legacy system and breaking it apart into services is just this hideous painful process that can take many years. But many companies have gone down this route and have experienced microservice nirvana at the end of it."  
> — Chris Richardson

---

## Riferimenti e risorse

- **[Microservices Patterns](https://www.manning.com/books/microservices-patterns)** — libro di Chris Richardson (Manning); 2 capitoli dedicati al testing
- **[microservices.io](https://microservices.io)** — pattern language, assessment tool
- **[The Art of Scalability](https://theartofscalability.com/)** — Martin Abbott & Michael Fisher; fonte del Scale Cube (X/Y/Z axis)
- **[Enterprise Integration Patterns](https://www.enterpriseintegrationpatterns.com/)** — Gregor Hohpe & Bobby Woolf; libro citato per i concetti di messaging channel
- **[Spring Cloud Contract](https://spring.io/projects/spring-cloud-contract)** — framework per consumer-driven contract testing
- **Istio** — service mesh per canary deployment e traffic routing
- **Conway's Law** — legge che lega struttura organizzativa e architettura software (già citata nel libro di Constantine del 1979)
- **Beth (speaker)** — ha parlato di consumer-driven contract testing nella sessione precedente a YOW! 2019
- **Cindy Sridharan** — autrice di post su testing in production e canary deployment
