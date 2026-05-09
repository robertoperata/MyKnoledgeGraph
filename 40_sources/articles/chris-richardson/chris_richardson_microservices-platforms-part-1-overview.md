---
tags:
  - microservices
  - team-topologies
  - platform-engineering
  - socio-technical-architecture
  - fast-flow
feature:
type: article
author: Chris Richardson
source: https://microservices.io/post/architecture/2025/12/10/qconsf-microservices-platforms-part-1.html
date: 2026-05-03
---

# Microservices Platforms - part 1: Overview

## Sunto

Questo articolo è il primo di una serie basata sul talk tenuto da Chris Richardson al QCon San Francisco 2025, intitolato "Microservices Platforms: When Team Topologies Meets Microservices Patterns". Introduce il framework concettuale che verrà approfondito nei successivi sei articoli della serie.

Il punto di partenza è la domanda: perché le **piattaforme interne** sono indispensabili per il fast flow? Richardson argomenta che adottare l'architettura a microservizi non basta: senza una serie di piattaforme dedicate, i team applicativi (stream-aligned team, nella terminologia di Team Topologies) spendono la maggior parte del tempo su problemi infrastrutturali anziché sulla logica di dominio. Il risultato è il cosiddetto **"modern legacy system"** — una nuova architettura con gli stessi vecchi problemi organizzativi e di velocità.

La serie identifica **sei piattaforme essenziali** che ogni organizzazione che adotta i microservizi dovrebbe costruire:

1. **Service Foundation Platform** — semplifica la creazione e manutenzione dei microservizi
2. **Security Platform** — semplifica lo sviluppo di microservizi sicuri
3. **Infrastructure Services Platform** — permette ai team di consegnare valore senza diventare esperti di infrastruttura
4. **Observability Platform** — rende i microservizi osservabili out of the box
5. **Build Platform** — fornisce infrastruttura e template per i deployment pipeline
6. **Deployment Platform** — fornisce gli ambienti di produzione e pre-produzione

L'approccio è fondamentalmente **socio-tecnico**: la struttura organizzativa (chi possiede quale piattaforma, come i team collaborano) è inscindibile dalle scelte architetturali. Applicando i principi delle **Team Topologies**, la piattaforma deve essere trattata come un prodotto interno offerto da un Platform Team al team applicativo, riducendo il carico cognitivo e abilitando il fast flow — la capacità di portare valore agli utenti finali in modo rapido, sicuro e ripetibile.
