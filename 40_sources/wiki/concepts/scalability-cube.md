---
title: Scale Cube (X/Y/Z Axis Scalability)
type: concept
tags:
  - thread1-microservices
sources:
  - "[[chris_richardson_cubes-hexagons-triangles-yow2019]]"
updated: 2026-05-07
related:
  - "[[concepts/independent-deployability]]"
  - "[[concepts/bounded-context]]"
---

# Scale Cube (X/Y/Z Axis Scalability)

## Definizione

Framework da *The Art of Scalability* (Abbott & Fisher) che descrive tre dimensioni ortogonali di scalabilità per un'applicazione. La Y-axis decomposition corrisponde esattamente alla microservice architecture.

## Come funziona

| Asse | Strategia | Descrizione |
|---|---|---|
| **X-axis** | Horizontal cloning | Multiple istanze identiche dell'applicazione dietro un load balancer. Semplice ma non risolve i problemi di volume dei dati. |
| **Z-axis** | Data partitioning | Il load balancer instrada le richieste in base ad un attributo (es. customer ID, shard key). Scalabilità dei dati ma ancora monolite funzionale. |
| **Y-axis** | **Functional decomposition** | Decomposizione dell'applicazione per capability/dominio. **Questa è la microservice architecture.** |

```
        ┌────────────────────────────────────────┐
       /│                                       /│
      / │    Y: funzionale                     / │
     /  │    (microservices)                  /  │
    ────│────────────────────────────────────    │
    │   │                                   │   │
    │   └────────────────────────────────── │───┘
    │  /  Z: partitioning (shard)           │  /
    │ /                                     │ /
    │/   X: cloning                         │/
    └────────────────────────────────────────┘
```

## Perché il monolite degrada nel tempo

```
All'inizio: architettura pulita a layer, team di 6 persone
↓ (crescita nel tempo)
Team di 100-200 persone tutti sullo stesso codebase
↓
Modularity breakdown: "big ball of mud"
↓
Technology lock-in: impossibile adottare nuovi stack
↓
Delivery slowdown: il ritmo di consegna precipita
```

Il monolite non è un anti-pattern — è la scelta giusta per applicazioni piccole. I problemi emergono con la crescita quando la modularità degrada.

## Il triangolo del successo (Richardson)

Per il fast flow (DevOps) con team autonomi e applicazioni longeve, servono tre elementi in sinergia:

| Dimensione | Approccio |
|---|---|
| **Processo** | Lean + DevOps (continuous delivery/deployment) |
| **Organizzazione** | Team cross-functional da 6-10 persone, loosely coupled |
| **Architettura** | Microservizi (testabilità, deployabilità, modularità) |

**Metriche DevOps** (DORA metrics):
- Lead time dal commit al deploy (minimizzare)
- Deployment frequency (massimizzare)
- Change failure rate (minimizzare)
- Mean Time to Recovery (minimizzare)

## Connessioni

- [[concepts/independent-deployability]] — la Y-axis decomposition richiede e garantisce l'independent deployability
- [[concepts/bounded-context]] — ogni slice Y corrisponde a un bounded context
- [[concepts/team-topologies]] — la struttura dei team deve rispecchiare la decomposizione (Conway's Law)
