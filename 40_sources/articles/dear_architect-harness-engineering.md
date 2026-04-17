---
tags:
  - ai
  - agentic
  - testing
  - architecture
  - coding-agents
type: article
author: Dear Architects
source: https://martinfowler.com/articles/harness-engineering.html
date: 2026-04-12
---

# Harness Engineering for Coding Agent Users

## Sunto

L'articolo di Birgitta Böckeler (Thoughtworks) introduce il concetto di *harness engineering* applicato agli agenti di coding AI: si tratta dell'insieme dei sistemi che i team costruiscono *intorno* agli agenti per aumentare la fiducia nel codice generato. L'autrice distingue tra controlli *feedforward* (guide che anticipano comportamenti indesiderati prima che l'agente agisca) e controlli *feedback* (sensori che osservano i risultati e abilitano la correzione). Entrambi sono necessari: un sistema con sole guide codifica regole senza verifica, mentre uno con soli sensori perpetua gli errori ripetuti.

I controlli si classificano in *computazionali* (deterministici, veloci — linter, test, type checker) e *inferenziali* (analisi semantica via LLM — più lenti e non deterministici, ma in grado di rilevare problemi di significato). Tre categorie principali di harness emergono: **Maintainability Harness** (qualità del codice, già matura), **Architecture Fitness Harness** (fitness function per caratteristiche architetturali come performance e osservabilità) e **Behaviour Harness** (correttezza funzionale — l'area più debole, che oggi si affida ancora troppo a test generati da AI).

Un concetto chiave è lo *steering loop*: l'harness non è una configurazione statica ma una pratica ingegneristica continua. Quando un problema si ripresenta, il team migliora guide o sensori invece di supervisionare manualmente ogni volta. Strategicamente, i controlli vanno distribuiti lungo il ciclo di sviluppo in base a costo e velocità ("keep quality left"): linter e revisioni base pre-commit, mutation testing e architettura review post-integrazione, scansione delle dipendenze in produzione.

L'autrice introduce il concetto di *harnessability* e di *ambient affordances*: ambienti con linguaggi fortemente tipizzati, confini di modulo chiari e framework opinati offrono agli agenti una base "leggibile" che rende più efficaci le guide e i sensori. I progetti greenfield possono incorporare queste proprietà fin dall'inizio; i sistemi legacy presentano sfide maggiori ma ne hanno più bisogno. Un'idea promettente sono i *harness template*: bundle di guide e sensori pre-costruiti per le topologie di servizio più comuni (API CRUD, event processor, dashboard).

Il messaggio finale è che gli esseri umani portano un harness implicito — convenzioni assorbite, giudizio estetico sulla complessità, responsabilità sociale — che gli agenti non possiedono. Un buon harness non deve eliminare l'input umano, ma reindirizzarlo verso ciò che conta davvero. L'engineering dell'harness è, in definitiva, l'arte di rendere il territorio degli agenti governabile senza rinunciare alla loro potenza.
