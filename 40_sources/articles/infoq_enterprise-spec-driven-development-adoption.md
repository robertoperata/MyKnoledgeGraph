---
tags:
  - architecture
  - agentic
  - ai
  - developer-experience
  - platform-engineering
type: article
author: Hari Krishnan
source: https://www.infoq.com/articles/enterprise-spec-driven-development
date: 2026-02-19
---

# Spec-Driven Development – Adoption at Enterprise Scale

## Sunto

L'articolo affronta l'adozione di **Spec-Driven Development (SDD)** a scala enterprise, tracciando l'evoluzione dell'interazione con l'AI dallo sviluppo tradizionale al paradigma spec-driven. Il percorso evolutivo parte dal *vibe coding* (iterazione prompt-by-prompt senza pianificazione deliberata), passa per la *plan mode* (l'AI elabora un piano di esecuzione per revisione umana prima di scrivere codice), e arriva a SDD dove le specifiche diventano il mezzo di dialogo tra umani e agenti AI, non semplici istruzioni.

Il punto centrale è che SDD replica i pattern di comunicazione tra senior engineer: la comprensione condivisa emerge attraverso il dialogo, non attraverso documenti prescrittivi. Le specifiche si articolano in tre livelli di responsabilità — **What (Discover)** per il Product (contesto aziendale), **How (Design)** per l'Architecture (approccio tecnico), **Tasks** per l'Engineering (piano di esecuzione) — e servono sia come mezzo di conversazione tra stakeholder che come motore di contesto per gli agenti AI.

L'autore identifica otto sfide concrete nell'adozione enterprise del tooling SDD attuale: (1) tooling centrato sullo sviluppatore che crea attrito per PM e business analyst; (2) focus mono-repo inadeguato per architetture microservizi; (3) mancanza di separazione delle responsabilità tra decisioni architetturali e contesto aziendale; (4) punti di partenza poco chiari rispetto al backlog esistente; (5) modelli di collaborazione non definiti tra ruoli diversi; (6) ampia varianza negli stili di specifica e granularità; (7) due transizioni distinte da gestire (intent→spec e spec→implementation); (8) percorso brownfield non chiaro per sistemi legacy.

Le soluzioni pratiche proposte seguono un approccio **"non boiling the ocean"**: integrazione con backlog esistenti (Jira, Linear, Azure DevOps) tramite MCP server, orchestrazione multi-repository con separazione del contesto aziendale dai dettagli tecnici, contributi specifici per ruolo dove agenti specializzati (infrastruttura, performance, sicurezza) stratificano automaticamente le loro linee guida sulle storie in arrivo. Per i sistemi brownfield, l'approccio incrementale è preferito: la copertura spec cresce organicamente con ogni modifica, simile al TDD nei refactoring legacy.

> "The unit of delivery is no longer a service or a codebase. The unit of delivery becomes the specification itself."

La visione a lungo termine introduce il concetto di **Harness Governance**: le specifiche meritano le stesse pratiche del codice di produzione (quality control, version control, revisione, miglioramento continuo). I bug diventano metriche di qualità dell'*harness* di contesto: un *Spec-to-Implementation Gap* indica che la specifica era chiara ma l'implementazione è divergita (serve rafforzare i meccanismi di validazione); un *Intent-to-Spec Gap* indica che un dettaglio del caso d'uso è andato perso durante la deliberazione (serve migliorare il processo di estrazione della spec). Il ruolo del QA evolve dalla validazione delle implementazioni alla validazione degli harness che guidano l'esecuzione degli agenti.

| Fase SDD | Responsabile | Focus |
|---|---|---|
| **Discover** (What) | Product Owner + AI | Intento aziendale, criteri di accettazione |
| **Design** (How) | Architect + AI | Approccio tecnico, decomposizione multi-repo |
| **Tasks** | Developer + AI | Dettagli implementativi, schema, moduli |

---

## Link esterni

- [Spec Driven Development: When Architecture Becomes Executable](https://www.infoq.com/articles/spec-driven-development/) — articolo precedente di Leigh Griffin e Ray Carroll citato come fondamento teorico di SDD

---

## Immagini

- ![Vibe coding workflow](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/enterprise-spec-driven-development/en/resources/1Figure-1-1771319819572.jpg) — flusso di lavoro del vibe coding: iterazione istruttiva prompt-by-prompt
- ![Plan Mode Workflow](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/enterprise-spec-driven-development/en/resources/1Figure-2-1771319819572.jpg) — flusso di lavoro della plan mode con revisione umana del piano prima del codice
- ![SDD high-level workflow](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/enterprise-spec-driven-development/en/resources/1Figure-3-1771319819572.jpg) — flusso di lavoro SDD ad alto livello
- ![SDD detailed workflow](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/enterprise-spec-driven-development/en/resources/1Figure-4-1771319819572.jpg) — flusso SDD dettagliato con fasi Discover/Design/Tasks
- ![OpenSpec SDD workflow](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/enterprise-spec-driven-development/en/resources/1Figure-5-a-1771319819572.jpg) — flusso SDD di OpenSpec con fasi proposal/application/archival
- ![OpenSpec workflow integrato con backlog via MCP](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/enterprise-spec-driven-development/en/resources/1Figure-5-b-1771319819572.jpg) — integrazione del backlog prodotto nel flusso SDD tramite MCP
- ![Collaborazione product owner, architect e dev tramite SDD](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/enterprise-spec-driven-development/en/resources/1Figure-6-1771319819572.jpg) — modello di collaborazione multi-ruolo nel flusso SDD
- ![Generazione sub-issue frontend e backend da contesto architetturale](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/enterprise-spec-driven-development/en/resources/1Figure-7-1771319819572.jpg) — decomposizione automatica in sub-issue per repository distinti
- ![Harness feedback loop](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/enterprise-spec-driven-development/en/resources/1Figure-8-1771319819572.jpg) — ciclo di feedback per il miglioramento continuo dell'harness di contesto
