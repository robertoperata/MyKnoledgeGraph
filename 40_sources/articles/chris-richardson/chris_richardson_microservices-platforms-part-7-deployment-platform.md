---
tags:
  - microservices
  - deployment
  - kubernetes
  - gitops
  - team-topologies
  - platform-engineering
feature:
type: article
author: Chris Richardson
source: https://microservices.io/post/architecture/2026/04/23/qconsf-microservices-platforms-part-7.html
date: 2026-04-29
---

# Microservices Platforms - part 7: Deployment platform

## Sunto

Questo articolo è il settimo di una serie basata sul talk tenuto da Chris Richardson al QCon San Francisco 2025, intitolato "Microservices Platforms: When Team Topologies Meets Microservices Patterns". La serie esplora come strutturare le piattaforme interne a supporto dei team di sviluppo che adottano un'architettura a microservizi, applicando i principi delle Team Topologies per abilitare il fast flow.

Il **Deployment platform** è il componente della piattaforma che si occupa di fornire gli ambienti di produzione e pre-produzione nei quali i servizi dell'applicazione vengono eseguiti. Insieme al Build platform (trattato nella parte 6), il Deployment platform definisce il percorso completo che una modifica al codice compie dal laptop dello sviluppatore fino alla produzione — il cosiddetto "path to production".

Il Deployment platform integra tipicamente strumenti e pratiche come **Kubernetes** per l'orchestrazione dei container, **GitOps** per la gestione dichiarativa dello stato infrastrutturale (con strumenti come ArgoCD o Flux), e **Infrastructure as Code** per la definizione riproducibile degli ambienti. Questi elementi, combinati con template di servizio e pattern di deployability, permettono ai team applicativi di rilasciare in autonomia senza doversi occupare della complessità infrastrutturale sottostante.

L'approccio alle Team Topologies impone che la piattaforma sia trattata come un prodotto interno: il team di piattaforma (Platform Team) offre capacità self-service al team applicativo (Stream-aligned Team), riducendo il carico cognitivo e aumentando la frequenza di deploy. Il Deployment platform è quindi un abilitatore fondamentale per raggiungere l'obiettivo del fast flow — ovvero la capacità di portare valore agli utenti finali in modo rapido, sicuro e ripetibile.

Questa serie di articoli fornisce un framework strutturato per progettare piattaforme interne a microservizi, affrontando in modo sistematico tutti i layer coinvolti: service foundation, sicurezza, servizi infrastrutturali, osservabilità, build e infine deployment. L'approccio è particolarmente rilevante per le organizzazioni che vogliono modernizzare la propria architettura evitando di creare un "modern legacy system" — una nuova architettura con gli stessi vecchi problemi organizzativi.
