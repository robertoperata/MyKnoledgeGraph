---
tags:
  - knowledge-graphs
  - agentic-ai
  - rag
  - llm-architecture
  - ai-engineering
feature:
type: article
author: Cassie Shum
source: https://r.mail.infoq.com/mk/cl/f/sh/7nVU1aA2nfwFRoMo2nayfjBKouRPhk1/S2R5Zpjpbk99
date: 2026-09-26
---

# From Retrieval to Reasoning: Building Production-Ready Agentic AI Systems with Knowledge Graphs

## Sunto

Cassie Shum argomenta che i knowledge graph rappresentano una fondamenta critica per i sistemi agentici in produzione, andando ben oltre le limitazioni del RAG (Retrieval-Augmented Generation) tradizionale. Mentre il RAG standard si limita al recupero di frammenti testuali simili, un knowledge graph permette all'agente di navigare relazioni strutturate, ragionare su catene causali e mantenere provenance delle decisioni — capacità essenziali per sistemi affidabili in contesti enterprise.

L'articolo illustra quattro pattern architetturali pratici costruiti su knowledge graph. Il primo è il *context bundling*: invece di recuperare documenti isolati, il grafo consente di aggregare contesto relazionale coerente attorno a un'entità, riducendo l'allucinazione e migliorando la pertinenza delle risposte. Il secondo pattern è la *decision provenance*: ogni passo ragionativo dell'agente viene tracciato come nodo nel grafo, creando un audit trail che rende il comportamento dell'agente spiegabile e verificabile.

Il terzo pattern, *code as truth*, utilizza il knowledge graph per mantenere una rappresentazione aggiornata del codebase come grafo di dipendenze — più accurato di qualsiasi documentazione manuale. Il quarto pattern, *agent visibility*, espone metriche operative del grafo (utilizzo dei nodi, percorsi più frequenti, nodi orfani) per ottimizzare il comportamento dell'agente nel tempo. Shum presenta anche un "engineering harness" costruito su knowledge graph per semplificare i feedback loop tra agenti, ottimizzare il consumo di token e mantenere l'affidabilità del sistema in produzione.

Il punto centrale dell'articolo è che il passaggio da sistemi RAG reattivi a sistemi agentici proattivi richiede una rappresentazione della conoscenza strutturata e navigabile, non solo un vettore di embedding. I knowledge graph forniscono la struttura semantica necessaria per supportare ragionamento multi-step, risoluzione di ambiguità, e coordinamento tra agenti specializzati in architetture multi-agent.
