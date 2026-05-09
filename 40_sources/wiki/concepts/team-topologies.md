---
title: Team Topologies
type: concept
tags:
  - thread1-microservices
sources:
  - "[[chris_richardson_microservices-platforms-part-1-overview]]"
  - "[[chris_richardson_microservices-platforms-part-2-service-foundation-platform]]"
  - "[[chris_richardson_microservices-platforms-part-3-security-platform]]"
  - "[[chris_richardson_microservices-platforms-part-4-infrastructure-services-platform]]"
  - "[[chris_richardson_microservices-platforms-part-5-observability-platform]]"
  - "[[chris_richardson_microservices-platforms-part-6-build-platform]]"
  - "[[chris_richardson_microservices-platforms-part-7-deployment-platform]]"
updated: 2026-05-07
related:
  - "[[concepts/microservices-platform]]"
  - "[[concepts/microservice-chassis]]"
  - "[[concepts/independent-deployability]]"
  - "[[concepts/scalability-cube]]"
---

# Team Topologies

## Definizione

Framework di Manuel Pais e Matthew Skelton per strutturare le organizzazioni IT in modo da abilitare il **fast flow** — la capacità di portare valore agli utenti finali in modo rapido, sicuro e ripetibile. Applicato ai microservizi da Chris Richardson come modello organizzativo per le piattaforme interne.

## Come funziona

### Due tipologie fondamentali nel contesto microservizi

**Stream-aligned team (team applicativo)**:
- Responsabile di un flusso di valore business end-to-end
- Sviluppa e opera la logica di dominio
- Vuole consegnare feature, non diventare esperto di infrastruttura
- "Full cognitive load" sulla logica di business

**Platform team**:
- Responsabile di una piattaforma interna
- Offre capacità self-service al team applicativo
- Riduce il carico cognitivo del team applicativo
- Tratta la piattaforma come un **prodotto interno**

### Il contratto tra i team

```
Platform Team → [piattaforma come prodotto] → Stream-aligned Team
                    ↑
           Self-service, API stabili,
           documentazione, supporto
```

Il team applicativo non collabora in modo ad-hoc con il platform team ogni volta che ha bisogno di infrastruttura — consuma la piattaforma in modo autonomo tramite interfacce self-service.

## Applicazione alle Microservices Platforms (Richardson)

Richardson mappa i sei layer della piattaforma su team dedicati:

| Piattaforma | Team responsabile | Capacità offerta |
|---|---|---|
| Service Foundation | Platform Team | Microservice chassis, service template |
| Security | Platform Team | Identity provider, mTLS, gestione segreti |
| Infrastructure Services | Platform Team | Kubernetes, service mesh, service discovery |
| Observability | Platform Team | Prometheus, ELK, Jaeger, dashboard |
| Build | Build Platform Group | CI/CD pipeline, reusable workflows, artifact registry |
| Deployment | Platform Team | Ambienti prod/pre-prod, GitOps, IaC |

## Il problema del "modern legacy system"

Adottare l'architettura a microservizi senza costruire le piattaforme di supporto genera il **modern legacy system**: una nuova architettura con gli stessi vecchi problemi organizzativi e di velocità. I team applicativi spendono la maggior parte del tempo su problemi infrastrutturali anziché sulla logica di dominio.

## Conway's Law

> L'architettura di un sistema tende a rispecchiare la struttura di comunicazione dell'organizzazione che lo costruisce.

Implicazione pratica: per avere microservizi loosely coupled, i team devono essere loosely coupled. Non è possibile avere team autonomi con un'architettura monolitica tightly coupled. Le Team Topologies sono la risposta operativa a Conway's Law.

## Quando usarlo

- Organizzazioni che adottano microservizi in scala (decine di team)
- Quando il carico cognitivo infrastrutturale rallenta il delivery dei team applicativi
- Come framework per decidere chi possiede cosa in un'organizzazione engineering

## Connessioni

- [[concepts/microservices-platform]] — le piattaforme sono il prodotto che il Platform Team costruisce
- [[concepts/microservice-chassis]] — il chassis è il deliverable principale della Service Foundation Platform
- [[concepts/independent-deployability]] — il fast flow richiede independent deployability, che richiede piattaforme di supporto
- [[concepts/scalability-cube]] — la decomposizione funzionale (Y-axis) genera il bisogno di Platform Teams
