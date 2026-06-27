---
tags:
  - ai-agents
  - rag
  - mcp
  - context-engineering
  - data-platform
feature:
type: article
author: Bonnie Xu
source: https://www.infoq.com/presentations/data-aware-ai-agents/
date: 2026-06-27
---

# AI Agents to Make Sense of Data at OpenAI

## Sunto

Bonnie Xu, tech lead del team Data Productivity di OpenAI, presenta Kepler: un agente AI interno progettato per interrogare oltre 600 petabyte di dati distribuiti su 70.000 dataset. Il problema fondamentale che Kepler risolve è la complessità delle piattaforme dati su larga scala, dove anche una piccola imprecisione semantica può portare a risposte errate di un intero ordine di grandezza — come confondere 5 milioni con 800 milioni di utenti.

L'architettura di Kepler si basa su quattro componenti fondamentali. Il primo è un sistema di gestione del contesto che incorpora metadati delle tabelle (schemi e cronologia delle query) arricchiti tramite crawling automatico del codice via Codex, producendo descrizioni semantiche ricche per ciascun dataset. Il secondo è un servizio di conoscenza interna che ingerisce thread Slack, documenti Notion e file Google Drive, resi disponibili tramite RAG per un recupero efficiente del contesto a runtime. Il terzo è il Model Context Protocol (MCP), che abilita l'orchestrazione degli strumenti permettendo al modello di raffinare iterativamente le query quando i risultati iniziali non sono soddisfacenti. Il quarto è una memoria strutturata su tre livelli di scope (utente, canale e globale) che consente un apprendimento continuo attraverso correzioni e feedback.

La gestione della memoria è organizzata gerarchicamente: embedding memorizzati per il recupero a runtime, con un meccanismo di potatura per eliminare le voci obsolete, garantiscono che l'agente impari dagli errori passati senza accumulare rumore semantico. Questo approccio si differenzia da una semplice cronologia delle conversazioni perché è strutturato e indicizzabile.

La sicurezza è implementata tramite un pattern di "autenticazione passthrough" — nessun grant aggiuntivo viene concesso all'agente oltre a quelli già posseduti dall'utente. Le query vengono pre-sanitizzate per prevenire fughe di PII, e l'output viene redatto tramite un servizio di anonimizzazione interno prima di essere mostrato all'utente. Il controllo delle autorizzazioni avviene prima di esporre i risultati raw.

La pipeline di valutazione utilizza la normalizzazione AST (Abstract Syntax Tree) per confrontare l'SQL generato con l'SQL di riferimento in modo strutturale, e il framework OpenAI Evals per il grading basato su modello con chain-of-thought per il debugging. Questa infrastruttura si è rivelata così efficace da essere considerata il principale enabler della qualità del sistema, confermando il principio: "Le eval sono sorprendentemente spesso tutto ciò di cui hai bisogno" (Greg Brockman).
