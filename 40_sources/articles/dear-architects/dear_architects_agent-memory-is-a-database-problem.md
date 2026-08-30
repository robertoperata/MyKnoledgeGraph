---
tags:
  - agent-memory
  - database
  - ai-agents
  - oracle
  - llm-engineering
  - vector-search
feature:
type: article
author: Richmond Alake
source: https://blogs.oracle.com/developers/agent-memory-is-a-database-problem-oracle-research-makes-the-case
date: 2026-08-30
---

# Agent Memory is a Database Problem: Oracle Research Makes the Case

## Sunto

Il team di ricerca Oracle ha pubblicato un articolo tecnico su arXiv sostenendo che la gestione della memoria degli agenti AI dovrebbe essere trattata come un problema di database, non come una sfida di finestra di contesto o di vector store isolato. Il report introduce Oracle Agent Memory, un substrato nativo per database costruito su Oracle AI Database. La tesi di fondo è che tutto ciò che la memoria degli agenti richiede — persistenza, indicizzazione, recupero con scope, transazioni, governance — è esattamente l'elenco dei requisiti di un database.

Le soluzioni tradizionali per la memoria degli agenti disperdono le funzionalità tra più servizi: buffer di riflessione, componenti aggiuntivi di framework, sistemi di memoria autonomi. Questo crea complicazioni di governance e sicurezza negli ambienti enterprise. Oracle propone invece di centralizzare la memoria dell'agente nel database, dove il dato aziendale già risiede e dove i controlli di accesso esistenti possono essere ereditati, eliminando la necessità di creare "shadow data store" con modelli di autorizzazione duplicati.

Il sistema classifica la memoria degli agenti in tre categorie fondamentali: Working Memory (riassunti di thread e schede di contesto ottimizzate per l'iniezione nel prompt), Long-Term Factual Memory (fatti duraturi, preferenze e attributi che persistono tra le sessioni con scope per utente/agente), e Procedural Memory (lezioni apprese, strategie e linee guida derivate da risultati precedenti). Questa strutturazione permette al sistema di ridurre drasticamente l'uso di token mantenendo alta l'accuratezza nel recupero delle informazioni.

I risultati empirici riportati sono significativi: 93,8% di accuratezza su LongMemEval (469/500 test), con recall perfetta in sessione singola e 96,2% nel ragionamento temporale. La riduzione di token rispetto alla storia conversazionale piatta è di 10,7x (circa 1.300 token vs circa 13.900 al turno 80), con un win ratio di 3,7x nei confronti pairwise rispetto agli approcci baseline. L'integrazione per gli sviluppatori Python segue un workflow a quattro fasi: inizializzazione con connessione database ed embedder, creazione/riapertura di thread con scope per utente e agente, aggiunta di messaggi per persistenza ed estrazione, e recupero del contesto tramite ricerca con scope o riassunti.

## Codice

Il client Python `oracleagentmemory` segue un workflow a quattro fasi:

```python
# 1. Inizializzazione con connessione database, embedder e LLM opzionale
from oracleagentmemory import AgentMemory

memory = AgentMemory(
    connection=db_connection,
    embedder=embedder,
    llm=llm  # opzionale
)

# 2. Creazione/riapertura di thread con scope per utente e agente
thread = memory.get_or_create_thread(
    user_id="user_123",
    agent_id="support_agent"
)

# 3. Aggiunta di messaggi per persistenza, estrazione e riepilogo
thread.add_message(role="user", content="What's my order status?")
thread.add_message(role="assistant", content="Your order #456 ships tomorrow.")

# 4. Recupero del contesto via ricerca con scope o riassunti
context = thread.get_context(query="order status", top_k=5)
summary = thread.get_summary()
```
