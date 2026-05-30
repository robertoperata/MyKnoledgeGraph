---
tags:
  - openai
  - websocket
  - agentic-workflows
  - latency-optimization
  - api-design
feature:
type: article
author: Leela Kumili
source: https://www.infoq.com/news/2026/05/openai-websocket-responses-api/
date: 2026-05-30
---

# OpenAI Introduces WebSocket-Based Execution Mode to Reduce Latency in Agentic Workflows

## Sunto

OpenAI ha introdotto una modalità di esecuzione basata su WebSocket per la sua Responses API, con l'obiettivo di migliorare le performance dei workflow agentici utilizzati in agenti di codice e sistemi IA in tempo reale. Il cambiamento sostituisce il tradizionale pattern HTTP request-response con una connessione persistente e bidirezionale tra client e server, eliminando il costo dei round-trip di rete ripetuti che caratterizzano i workflow multi-step.

Il problema che questa soluzione affronta è la latenza strutturale nei workflow agentici complessi: ogni passo di ragionamento, ogni chiamata a un tool e ogni query di follow-up con HTTP tradizionale richiede l'apertura di una nuova connessione, la negoziazione TLS, e la trasmissione completa degli header. In workflow con decine o centinaia di step, questo overhead si accumula significativamente. Con WebSocket, la connessione viene stabilita una volta sola e rimane aperta per l'intera durata del workflow.

I risultati in produzione sono significativi: fino al 40% di riduzione della latenza rispetto all'HTTP tradizionale, throughput sostenuto di circa 1.000 transazioni per secondo, e capacità di burst fino a 4.000 transazioni per secondo. Gli sviluppatori di Vercel, Cline e Cursor hanno riportato miglioramenti del 30-40% nei loro workflow di produzione. Il progetto Codex ha migrato la maggior parte del suo traffico alla modalità WebSocket durante il testing in alpha, segnalando la maturità della soluzione.

Dal punto di vista tecnico, il pattern consente di pre-caricare ("warm") la connessione con system prompt e definizioni dei tool prima dell'elaborazione effettiva, allineandosi con i pattern di design event-driven dei sistemi distribuiti. La soluzione mantiene la compatibilità con Zero Data Retention (ZDR) per i requisiti di sicurezza dei clienti enterprise. La gestione dello stato attraverso interazioni multiple — anziché resettare ad ogni richiesta HTTP — riduce la complessità di orchestrazione nei workflow che coinvolgono ragionamenti intermedi, chiamate a tool e query successive.

Questo cambiamento riflette una tendenza più ampia nell'architettura dei sistemi IA: la necessità di infrastrutture di comunicazione ottimizzate per le caratteristiche specifiche dei workflow agentici, che differiscono fondamentalmente dalle interazioni stateless per cui HTTP fu originariamente progettato. Adottare WebSocket per workflow agentici è un'evoluzione naturale che si applica anche a client framework non-OpenAI che interagiscono con modelli language su connessioni di lunga durata.
