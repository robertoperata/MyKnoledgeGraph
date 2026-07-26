---
tags:
  - ai-agents
  - llm
  - model-context-protocol
  - multi-agent
  - rag
feature:
type: article
author: Micheal Lanham
source: https://www.manning.com/books/ai-agents-in-action-second-edition
date: 2026-07-26
---

# AI Agents in Action (Second Edition)

## Sunto

*AI Agents in Action* (seconda edizione, Manning, giugno 2026) di Micheal Lanham è una revisione sostanziale della prima edizione e si propone come guida pratica e completa per progettare, implementare, valutare e mettere in produzione agenti AI. A differenza di molte risorse che descrivono cosa sono gli agenti, questo libro si concentra sul *come* costruirli davvero, con codice Python funzionante che guida il lettore da agenti minimali a sistemi multi-agente complessi e pronti per la produzione.

Il framework concettuale centrale del libro è un modello funzionale a cinque livelli: **persona**, **azioni e strumenti**, **ragionamento e pianificazione**, **conoscenza e memoria**, **valutazione e feedback**. Questo schema non è solo una tassonomia accademica: è uno strumento diagnostico pratico. Un agente che fallisce può essere analizzato sistematicamente per identificare quale dei cinque livelli è carente, e le soluzioni proposte nel libro sono mappate direttamente su questi layer.

Un capitolo chiave è dedicato al **Model Context Protocol (MCP)** come connettore standard per le capacità degli agenti. MCP emerge come il meccanismo che permette agli agenti di accedere a strumenti esterni in modo standardizzato e componibile, riducendo il costo di integrazione di nuove capacità. Il libro illustra come usare l'OpenAI Agents SDK insieme a MCP per costruire sistemi modulari dove le capacità si aggiungono senza riscrivere la logica core.

I pattern di ragionamento coperti includono **ReAct**, **Reflexion**, **Tree-of-Thought** e **Sequential Thinking** — ognuno con esempi concreti su quando applicarli e come scegliere tra alternative. La sezione su RAG e memoria approfondisce come combinare retrieval semantico con memoria episodica e procedurale, e come strutturare la knowledge base perché l'agente mantenga coerenza attraverso sessioni multiple.

Il libro chiude con osservabilità e deployment in produzione: come monitorare il comportamento non deterministico degli agenti, come costruire pipeline di valutazione automatizzata, e come gestire il feedback loop tra valutazione umana e miglioramento continuo del sistema.
