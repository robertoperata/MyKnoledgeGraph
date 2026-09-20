---
tags:
  - context-engineering
  - model-context-protocol
  - ai-agents
  - linkedin
  - produttivita
feature:
type: article
author: Ajay Prakash
source: https://www.infoq.com/presentations/linkedin-context-engineering/
date: 2026-09-20
---

# Context Engineering at LinkedIn: AI Agents with Model Context Protocol

## Sunto

LinkedIn ha affrontato un problema fondamentale nell'adozione degli agenti di coding AI: i modelli erano efficaci su codice open source, ma fallivan ripetutamente nell'ambiente interno di LinkedIn. La causa era strutturale: gli agenti non conoscevano i sistemi interni, i framework proprietari, i database personalizzati e il "tribal knowledge" accumulato in anni di sviluppo. Il codebase copre migliaia di repository e microservizi con infrastrutture custom che richiedono settimane di onboarding anche per ingegneri esperti.

La soluzione architettata da Ajay Prakash si basa sul **Model Context Protocol (MCP)** di Anthropic come standard aperto per connettere gli agenti AI agli strumenti e alla conoscenza organizzativa. L'innovazione centrale sono i **playbook**: strutture di memoria procedurale che catturano come eseguire task specifici con nome, descrizione, istruzioni step-by-step e riferimenti componibili ad altri playbook più piccoli. Ogni playbook affronta esattamente un task specifico (self-containment) e i playbook complessi referenziano quelli più piccoli (composability), evitando il sovraccarico della context window.

Per scalare oltre i limiti degli strumenti (oltre 30 strumenti degradano le performance), LinkedIn implementa un approccio a tre livelli: uno strumento di ricerca per scoprire tool e playbook tramite keyword, un layer per recuperare gli schemi dei parametri, e un layer di esecuzione. Questa architettura scala a migliaia di tool e playbook mantenendo qualità delle risposte.

I risultati operativi sono significativi: 8.000 utenti attivi giornalieri (non solo ingegneri, ma anche PM, designer e TPM), oltre 600 playbook operativi, **+20% di produttività con zero degradazione dell'affidabilità**. I cinque casi d'uso principali sono: debugging e investigation (runbook automatizzati), generazione di boilerplate, cleanup e migrazioni del codice, gestione infrastruttura, e setup dell'ambiente per nuovi ingegneri.

La governance è un elemento chiave del sistema: tutti i tool passano per revisione InfoSec, il server MCP è pre-installato su tutti i laptop LinkedIn con aggiornamenti orari automatici, ogni contribuzione ai playbook richiede code review. La roadmap include agenti in background che analizzano pattern di PR e telemetria per identificare workflow automatizzabili e generare playbook in modo automatico, riducendo il carico di documentazione manuale.

## Codice

Il pattern di composabilità dei playbook prevede che ogni playbook referenzi playbook più piccoli:

```yaml
# Esempio struttura di un playbook LinkedIn
name: "debug-service-incident"
description: "Investigate and debug a service incident"
instructions:
  - "Step 1: Check monitoring dashboards"
  - "Step 2: See playbook: check-recent-deployments"
  - "Step 3: See playbook: query-internal-logs"
  - "Step 4: See playbook: create-incident-report"
references:
  - check-recent-deployments
  - query-internal-logs
  - create-incident-report
```

Architettura a tre livelli per scalare oltre i limiti dei tool:

```
Layer 1: Search Tool
  └─ Scopre tool e playbook via keyword/tag

Layer 2: Schema Retrieval
  └─ Carica i parametri del tool selezionato

Layer 3: Execution
  └─ Esegue il tool con i parametri corretti
```
