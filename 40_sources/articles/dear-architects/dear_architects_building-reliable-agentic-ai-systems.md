---
tags:
  - agentic-ai
  - rag
  - llm
  - multi-agent
  - langgraph
  - enterprise-ai
feature:
type: article
author: Sarang Sanjay Kulkarni
source: https://martinfowler.com/articles/reliable-llm-bayer.html
date: 2026-06-21
---

# Building Reliable Agentic AI Systems

## Sunto

L'articolo descrive PRINCE (Preclinical Information Center), una piattaforma agentica sviluppata da Bayer per la ricerca preclinica nel settore farmaceutico. Il problema centrale era che i ricercatori dovevano accedere a migliaia di report di studi in formato PDF (spesso decenni di documenti scansionati), dati strutturati in database disparati e metadati frammentati in sistemi isolati. Le tradizionali ricerche per parole chiave con logica booleana si dimostravano inadeguate per rispondere a domande scientifiche complesse.

La soluzione adottata combina due tecniche di recupero: RAG (Retrieval-Augmented Generation) per i dati non strutturati (PDF) e Text-to-SQL per i dati strutturati in Amazon Athena. L'orchestrazione del flusso di lavoro è implementata con LangGraph, che gestisce una pipeline multi-agente composta da un agente Researcher, un agente Reflection e un agente Writer. Ogni agente riceve solo il contesto necessario per il proprio compito specifico, evitando il fenomeno di "context pollution" che degradava le performance nelle versioni iniziali.

Un elemento chiave dell'architettura è la distinzione tra tre tipi di riflessione: la riflessione sul processo (Think & Plan), che verifica che il workflow stia progredendo nella giusta direzione; la riflessione sui dati (Reflection Agent), che valuta se le informazioni recuperate siano sufficienti per rispondere alla domanda; e la riflessione sulla bozza (Writer Agent), che controlla la completezza e la coerenza della risposta finale. Questi tre livelli di riflessione complementari formano un pattern di "context engineering" che assegna il contesto giusto all'agente giusto al momento giusto.

Il sistema RAG utilizza un approccio di ricerca ibrida che combina ricerca semantica vettoriale (peso 0.7) e ricerca per parole chiave (peso 0.3), con query expansion (n=5 query semanticamente simili), metadata filtering su OpenSearch e un cross-encoder (bge-reranker-large) per il re-ranking dei top 20 chunk verso i top 7 finali. Per il Text-to-SQL, il sistema usa few-shot prompting dinamico con esempi recuperati da un vector store e la validazione delle query per bloccare operazioni non-SELECT.

L'affidabilità del sistema è garantita da meccanismi di fallback a livello di LLM (con switch automatico su provider alternativi), persistenza dello stato in PostgreSQL tramite LangGraph checkpointer, retry automatici a più livelli e possibilità di riprendere i workflow dal punto di fallimento senza ricominciare da capo. Il monitoraggio utilizza Langfuse per il tracing dettagliato del traffico di produzione e il framework RAGAS per la valutazione delle metriche di qualità (Faithfulness, Answer Relevancy, Context Relevancy, Answer Accuracy).

## Immagini

![System context and container view of PRINCE platform](https://martinfowler.com/articles/reliable-llm-bayer/system-container-view.png)

![The multi-agent research workflow in PRINCE](https://martinfowler.com/articles/reliable-llm-bayer/new-research-workflow.png)

![Text-to-SQL tool architecture](https://martinfowler.com/articles/reliable-llm-bayer/text-to-sql.png)

## Codice

Pipeline RAG con ricerca ibrida, query expansion e reranking:

```
Query utente: "Were any clinical findings observed in study T123456-2:
piloerection, ataxia, eyes partially closed, loose faeces?"

1. Estrazione keywords:
   ["piloerection", "ataxia", "eyes partially closed", "loose faeces"]

2. Generazione filtro metadata:
   eq(study_id, T123456-2)

3. Query expansion (n=5 varianti semantiche):
   - "Can you provide details on clinical symptoms in research T123456-2..."
   - "In results of experiment T123456-2, were there recorded observations..."
   - ...

4. Ricerca ibrida parallela su OpenSearch:
   - Semantic search weight: 0.7
   - Keyword search weight: 0.3
   - Risultati iniziali: ~20 chunk

5. Reranking con cross-encoder (bge-reranker-large):
   - Top 7 chunk selezionati

6. Generazione risposta con citazioni
```
