---
tags:
  - claude-api
  - agent-patterns
  - tool-use
  - rag
  - ai-engineering
feature:
type: article
author: Dear Architects
source: https://platform.claude.com/cookbook/
date: 2026-05-10
---

# Claude Cookbook — Catalogo di Pattern per Agenti AI

## Sunto

Il Claude Cookbook è il catalogo ufficiale di Anthropic di guide pratiche ed esempi per l'utilizzo efficace di Claude nell'ingegneria AI. Il catalogo copre l'intero spettro dello sviluppo con Claude: dall'uso base dell'API fino a sistemi multi-agente complessi in produzione. È organizzato in categorie tematiche che riflettono i pattern architetturali più importanti nell'ingegneria AI moderna.

La categoria **Claude Managed Agents** (maggio 2026) rappresenta lo stato dell'arte nella costruzione di sistemi agentici: include pattern per coordinare team di agenti specializzati (ricercatori web, bibliotecari di file, classificatori), cicli di verifica autonoma del lavoro prodotto (grade-and-revise loop), gestione della memoria persistente per preferenze utente cross-sessione, e agenti SRE per incident response con generazione automatica di PR correttive. La categoria **Claude Agent SDK** (da febbraio 2026) fornisce pattern per vulnerability detection, site reliability, ricerca con un solo comando e sistemi multi-agente con subagenti e hooks.

La sezione **Tools & Tool Use** affronta problemi concreti di ingegneria: il Programmatic Tool Calling (PTC) riduce latenza e consumo di token lasciando che Claude scriva codice che chiama strumenti programmaticamente nell'ambiente di code execution; il Tool Search with Embeddings scala le applicazioni a migliaia di strumenti tramite embeddings semantici per il discovery dinamico; l'Automatic Context Compaction gestisce i limiti di contesto in workflow agentici di lunga durata comprimendo automaticamente la history della conversazione.

La sezione **RAG & Retrieval** copre le tecniche chiave per sistemi di retrieval-augmented generation: dal Contextual Retrieval (settembre 2024, che migliora l'accuratezza RAG con embedding contestuale) alla costruzione di knowledge graph con entity extraction, relation mining, deduplication e multi-hop querying (marzo 2026). Sono presenti ricette per Text-to-SQL con auto-miglioramento, integrazione con Pinecone e MongoDB, e tecniche di summarization avanzate con valutazione.

La sezione **Agent Patterns** cataloga i building block fondamentali per sistemi multi-agente: Basic Workflows (tre pattern semplici multi-LLM), Evaluator-Optimizer (un LLM per generazione, uno per feedback di valutazione), Orchestrator-Workers (LLM centrale che delega ai worker LLM), e ReAct agents con LlamaIndex per ragionamento basato su strumenti. Completano il catalogo sezioni per multimodale (vision e audio), Skills (Excel, PowerPoint, PDF), observability, integrazioni con LangChain e LlamaIndex, fine-tuning su Amazon Bedrock, e valutazione dei sistemi AI.
