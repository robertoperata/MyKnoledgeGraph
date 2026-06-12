---
tags:
  - microservices
  - platform-engineering
  - team-topologies
  - organizational-design
  - developer-experience
feature:
type: article
author: Chris Richardson
source: https://microservices.io/post/architecture/2026/06/03/qconsf-microservices-platforms-part-8.html
date: 2026-06-12
---

# Microservices Platforms - part 8: Getting Started with Platforms

## Sunto

Questo è l'ottavo articolo della serie basata sul talk di Chris Richardson al QCon San Francisco 2025, "Microservices Platforms: When Team Topologies Meets Microservices Patterns". La serie ha esplorato le diverse tipologie di piattaforme (Service Foundation, Security, Infrastructure Services, Observability, Build, Deployment), e questo articolo conclusivo affronta la domanda più pragmatica: come iniziare con le piattaforme e perché tanti team di platform engineering falliscono nel consegnare valore.

Richardson cita una ricerca secondo cui fino al **70% dei team di platform engineering fallisce nel consegnare impatto**, un dato che definisce "deprimente ma non sorprendente" basandosi sulla propria esperienza professionale. Il problema è strutturale: i team di platform engineering spesso partono con un mandato vago ("costruisci una piattaforma"), scelgono le tecnologie prima di capire i problemi reali degli sviluppatori, e misurano il successo in termini di feature rilasciate invece che di riduzione dell'attrito per i team di prodotto.

Le cause principali del fallimento identificate nell'articolo includono: mancanza di focus sui problemi reali degli utenti (i developer team), adozione di un approccio "big bang" invece di consegnare valore incrementale, difficoltà nel bilanciare la standardizzazione con la flessibilità richiesta dai team, e la tentazione di costruire piattaforme troppo complesse che nessuno usa. Richardson suggerisce di applicare gli stessi principi del product thinking alle piattaforme interne: identificare i job-to-be-done degli sviluppatori, costruire il minimo necessario, raccogliere feedback frequenti.

Il concetto di **Team Topologies** (di Matthew Skelton e Manuel Pais) è centrale nell'approccio: una piattaforma efficace riduce il carico cognitivo dei team di prodotto fornendo "paved roads" — percorsi ottimizzati e opinionated per le esigenze più comuni. Il successo si misura non in termini di uptime della piattaforma, ma di quanto i team di prodotto riescono ad accelerare il loro delivery mantenendo qualità e compliance.

Per iniziare, Richardson raccomanda di partire da un problema specifico e ad alto impatto — tipicamente una pain point comune a molti team (onboarding, deployment, osservabilità), costruire una soluzione minima ma funzionante, e iterare basandosi sul feedback reale. L'obiettivo è costruire fiducia attraverso il valore consegnato, non attraverso la completezza architetturale.

## Immagini

![Slide 48 - Platform Engineering Failures](https://microservices.io/i/qconsf-microservices-platforms-part-8/new-stack-platform-failiures.png)
