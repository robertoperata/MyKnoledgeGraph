---
tags:
  - ai-agents
  - model-context-protocol
  - context-engineering
  - developer-productivity
  - llm-tooling
feature:
type: article
author: Ajay Prakash
source: https://www.infoq.com/presentations/linkedin-context-engineering/
date: 2026-09-26
---

# Context Engineering at LinkedIn: How We Built an Organizational Context Layer for AI Agents with MCP

## Sunto

Ajay Prakash (Software Engineer di LinkedIn) descrive come LinkedIn abbia affrontato un problema fondamentale nell'adozione degli agenti AI di coding: i grandi codebase aziendali sono incomprensibili per i modelli senza contesto organizzativo. La soluzione sviluppata, denominata "Contextual Agent Playbooks and Tools", utilizza il Model Context Protocol (MCP) di Anthropic come standard aperto per connettere strumenti e agenti AI, permettendo agli agenti di accedere a conoscenza codificata su come operare all'interno dell'infrastruttura LinkedIn.

Il concetto centrale è la *procedural memory*: playbook step-by-step che catturano conoscenza tribale, best practice e runbook operativi che gli agenti possono consultare ripetutamente. Invece di iniettare tutte le istruzioni nel prompt, i playbook vengono recuperati dinamicamente riducendo il consumo di token e migliorando la coerenza dei risultati. Ogni playbook è progettato per essere autocontenuto (esegue esattamente un task) e componibile (i playbook più grandi referenziano quelli più piccoli per evitare context overflow).

Per risolvere il problema della scalabilità degli strumenti — i modelli degradano significativamente oltre una soglia di circa 30 strumenti disponibili — LinkedIn ha introdotto tre meta-tool: ricerca di strumenti/playbook, recupero degli schema degli strumenti, ed esecuzione dello strumento selezionato. Questo pattern consente di scalare a centinaia o migliaia di strumenti mantenendo le performance del modello. L'MCP server locale è installato su tutti i laptop LinkedIn con aggiornamenti automatici ogni ora, e conta 8.000 utenti giornalieri tra ruoli tecnici e non.

I risultati ottenuti mostrano un incremento di produttività del 20% senza degradazione della qualità del codice né della affidabilità dei sistemi. Un aspetto notevole è il comportamento di improvvisazione degli agenti: quando le informazioni di un playbook sono obsolete, gli agenti tentano comandi alternativi, consultano la documentazione disponibile ed escalano all'utente solo quando necessario. Questo suggerisce che la qualità del contesto organizzativo è più determinante della capacità del modello stesso.

Le direzioni future includono la generazione automatica di playbook a partire dai pattern delle pull request e dalla telemetria delle sessioni, e agenti in background che identificano autonomamente i gap di conoscenza e aggiornano i playbook. L'insegnamento principale è che l'infrastruttura persistente per accedere alla conoscenza tribale organizzativa rimane essenziale indipendentemente dagli aggiornamenti del modello.
