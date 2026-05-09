---
tags:
  - microservices
  - team-topologies
  - platform-engineering
  - ci-cd
  - deployment-pipeline
  - devops
  - github-actions
  - build-platform
feature:
type: article
author: Chris Richardson
source: https://microservices.io/post/architecture/2026/03/20/qconsf-microservices-platforms-part-6.html
date: 2026-05-03
---

# Microservices Platforms - part 6: Build Platform

## Sunto

Questo articolo è il sesto di una serie basata sul talk tenuto da Chris Richardson al QCon San Francisco 2025, intitolato "Microservices Platforms: When Team Topologies Meets Microservices Patterns". Descrive il quinto layer della piattaforma: la **Build Platform**, che insieme al Deployment Platform (parte 7) definisce il percorso completo che una modifica al codice compie dal laptop dello sviluppatore fino alla produzione.

Il problema centrale è sintetizzabile in una domanda: se ogni microservizio ha bisogno del proprio deployment pipeline, chi si occupa di configurare e mantenere tutti questi pipeline? L'**independent deployability** è uno dei grandi vantaggi dell'architettura a microservizi, ma ha un costo: ogni servizio tipicamente richiede il proprio CI/CD pipeline, e la configurazione e la gestione di questi pipeline richiedono expertise specializzata. È una distrazione dalle responsabilità principali dei team applicativi, e avere molteplici implementazioni bespoke è un incubo di manutenzione.

La **Build Platform**, mantenuta da un **Build Platform Group** dedicato, risolve questo problema fornendo:

- **Shared pipeline infrastructure**: l'infrastruttura CI/CD centralizzata (es. server Jenkins, runner GitHub Actions, cluster CircleCI) che tutti i team possono usare senza doverla gestire autonomamente
- **Reusable pipeline components**: componenti riutilizzabili che incapsulano task comuni — build, test, security scanning, containerizzazione — realizzati con strumenti come **GitHub Actions reusable workflows** o **CircleCI Orbs**
- **Deployment pipeline templates**: template pre-costruiti che i team applicativi possono adottare con minime personalizzazioni per ottenere un pipeline completo dal commit al deploy, seguendo le best practice organizzative
- **Artifact registry**: un registro centralizzato per le immagini Docker e altri artefatti di build, che garantisce tracciabilità e reproducibilità dei rilasci

L'obiettivo finale è che ogni nuovo servizio sia **deployable out of the box**: un team che usa il service template (descritto nella parte 2) ottiene automaticamente anche un pipeline funzionante, senza dover partire da zero. Questo abbatte drasticamente il tempo tra la creazione di un nuovo servizio e il suo primo deploy in produzione, abilitando il fast flow promesso dall'architettura a microservizi.
