---
tags:
  - ai-agents
  - developer-platform
  - coding-agents
  - enterprise-ai
  - ci-cd
feature:
type: article
author: Craig Risi
source: https://www.infoq.com/news/2026/06/dropbox-nova-ai-coding-agents/
date: 2026-06-27
---

# Dropbox Introduces Nova, an Internal Platform for Running AI Coding Agents at Scale

## Sunto

Dropbox ha presentato Nova, una piattaforma interna per orchestrare agenti AI di coding all'interno dei workflow di ingegneria enterprise. La filosofia di design di Nova si distingue dall'approccio degli strumenti AI standalone: invece di trattare gli agenti come assistenti isolati, Nova fornisce un "execution layer centralizzato" che permette agli agenti di operare all'interno del monorepo, dei sistemi CI, degli strumenti di osservabilità e dei workflow infrastrutturali di Dropbox.

Il flusso di lavoro centrale di Nova segue un pattern "propose, validate, iterate": gli agenti propongono modifiche al codice, le validano contro build reali e iterano finché le soluzioni non sono stabili. Questo pattern è fondamentalmente diverso dal semplice completamento del codice — richiede che l'agente abbia accesso alla stessa infrastruttura che userebbe un ingegnere umano: compilatori, test runner, log di sistema e strumenti di osservabilità. La separazione tra pubblicazione del codice ed esecuzione mantiene la prevedibilità deterministica dei workflow.

Un'applicazione concreta di Nova è "Deflaker", uno strumento interno che indaga e ripara automaticamente i test flaky. Il processo è completamente automatizzato: Deflaker analizza i log di fallimento, propone fix al codice del test, itera eseguendo i test fino a quando la soluzione si stabilizza, e solo allora presenta il risultato per la revisione umana. Questo trasforma un'attività altamente time-consuming (debugging dei test intermittenti) in un processo quasi completamente automatico.

La consapevolezza contestuale è un differenziatore chiave: Nova fornisce agli agenti accesso ai sistemi di osservabilità e ai tool interni di Dropbox, permettendo agli agenti di comprendere il comportamento dei sistemi in produzione e non solo il codice sorgente in isolamento. Questo "context-aware coding" è più efficace perché allinea il reasoning dell'agente con i vincoli reali del sistema.

Il caso di Dropbox Nova riflette una tendenza più ampia nell'industria: le aziende stanno costruendo piattaforme agentiche interne piuttosto che affidarsi solo ad assistenti AI standalone. L'enfasi è sull'integrazione con gli ecosistemi di ingegneria esistenti, sulla safety guardrail e sulla workflow integration, più che sulla qualità del modello sottostante. Il messaggio è chiaro: la qualità dell'agente dipende più dall'infrastruttura di integrazione che dalla potenza del modello.
