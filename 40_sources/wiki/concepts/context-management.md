---
title: Context Management (AI)
type: concept
tags:
  - thread4-ai
sources:
  - "[[tessl_context-development-lifecycle]]"
  - "[[AI Talks - Lada Kesseler Augmented Coding]]"
  - "[[Claude Code and Large-Context Reasoning]]"
  - "[[youtube_ai-coding-workflow-matt-pocock]]"
updated: 2026-05-09
related:
  - "[[concepts/agentic-patterns]]"
  - "[[concepts/deep-modules]]"
---

# Context Management

## Definizione

Il contesto è l'insieme di informazioni che un agente AI riceve per svolgere un task: prompt, istruzioni, history della conversazione, documenti di riferimento. La qualità del contesto è il collo di bottiglia dell'era agentica.

> "I team che vinceranno nell'era degli agenti non saranno quelli col modello migliore. Saranno quelli col contesto migliore." — Patrick Debois

## Context Development Lifecycle (CDLC)

Proposto da Patrick Debois (tessl.io), il CDLC tratta il contesto come codice:

### 1. Generate — Rendere esplicita la conoscenza implicita
- Articolare standard tecnici, pattern architetturali, timeline, compliance
- L'AI può fare una bozza, ma l'umano valida

### 2. Evaluate — Testare il contesto come si testa il codice
- Trattare il contesto come artefatto misurabile con **evals**
- TDD applicato al contesto: definisci comportamento atteso, testa, migliora
- Le evals rivelano ambiguità nelle specifiche e permettono confronti tra modelli

### 3. Distribute — Contesto come pacchetti versionati
- Distribuzione strutturata (stile npm/pip) per condivisione organizzativa
- Versioning per prevenire il **decadimento silenzioso del contesto**
- Controlli di sicurezza

### 4. Observe — Imparare dall'uso reale
- Gli agenti in produzione rivelano gap che i test sintetici non intercettano
- Le scelte inattese dell'agente espongono ambiguità nelle specifiche
- Feedback loop che guida il raffinamento continuo

## Il parallelo con DevOps

DevOps (2009) ha unificato Development e Operations che lavoravano separati. Il CDLC fa lo stesso per la creazione e il consumo di contesto. Vantaggio rispetto al DevOps classico: chi migliora il contesto vede **immediatamente** output migliori — l'incentivo personale è allineato con quello organizzativo.

## Dove vive il contesto oggi (e i problemi)

Il contesto è sparso ovunque: `.cursorrules`, `CLAUDE.md`, Slack, wiki, documentazione, PR. Senza versioning, test o conflict resolution — esattamente come veniva gestito il codice prima del version control.

## Tipi di contesto

| Tipo | Esempi | Stabilità |
|---|---|---|
| Project-level rules | `CLAUDE.md`, `code_style.md` | Alta |
| Feature context | `user_login_feature.md`, `scratchpad.md` | Bassa (usa e getta) |
| Domain knowledge | `knowledge.md` | Media |
| Conversation history | Chat con l'agente | Molto bassa (eviction) |

## Incremental Disclosure (Kiro)

Invece di caricare tutto il contesto disponibile all'inizio, l'agente riceve il **minimo necessario** e usa strumenti (codebase search, MCP) per auto-scoprire il contesto del task man mano che serve.

> "The agent does better when given less context but given the tools to understand where to go find things." — Al Harris (Kiro)

Questo si contrappone all'approccio "stuff everything into context": un contesto enorme non migliora necessariamente la qualità — anzi, può degradarla (l'agente viene distratto da informazioni irrilevanti). L'indice del codebase viene usato per **UI tools** (ricerca semantica, navigazione) piuttosto che per alimentare direttamente il prompt dell'agente.

## Prompt Caching come strategia di contesto

Negli IDE agentici come Kiro, ottimizzare per il **prompt cache hit rate** è più efficace che comprimere il contesto:

- Kiro raggiunge 90-95% di cache token reuse per turno
- Questo mantiene le interazioni veloci nonostante sessioni lunghe (200k token limit)
- La compressione (context summarization) viene evitata finché possibile perché distrugge la cache

Trade-off: quando si aggiungono/cambiano MCP o tool mid-session è una **cache-busting operation** → evitare se si è profondi in una sessione lunga.

## Smart Zone e Dumb Zone (Matt Pocock / Dex Hardy)

Ogni LLM ha una **smart zone** — l'inizio della conversazione, dove le attention relationships sono poco stressate — e una **dumb zone** — dove il modello degrada progressivamente.

L'attenzione scala quadraticamente: ogni token aggiunto crea nuove relazioni con tutti i token precedenti. Il risultato pratico: intorno ai **~100k token** — indipendentemente dalla context window dichiarata (200k, 1M) — il modello inizia a fare scelte sbagliate.

> "La 1M context window di Claude Code non ha ampliato la smart zone: ha semplicemente aggiunto più dumb zone. È utile per il retrieval, non per il coding." — Matt Pocock

Implicazione pratica: **le task vanno dimensionate per stare nella smart zone**. Questo è il vincolo strutturale che giustifica tutto il resto del workflow agentic: vertical slices piccole, issue indipendenti, context window pulite per ogni task.

## Clear vs Compact

Due strategie per gestire una context window piena:

| Strategia | Meccanismo | Problema |
|---|---|---|
| **Clear** | Azzera tutti i token, torna al system prompt | Nessuno — stato identico ogni volta |
| **Compact** | Comprime la conversazione in un sommario e continua | Il sommario porta "sedimento" — informazioni imprecise o obsolete |

**Clear è preferibile a compact**: lo stato iniziale pulito è sempre lo stesso, sempre predictable. Se il workflow è ottimizzato per ricominciare pulito (backlog esterno, issue ben definite), il clear ha un costo vicino a zero.

> **Tensione con l'approccio Kiro:** Kiro ottimizza per il **prompt cache hit rate** (90-95% di cache token reuse) e quindi evita il context summarization perché distrugge la cache. Pocock preferisce clear per predictability. I due approcci rispondono a contesti diversi: Kiro ottimizza sessioni lunghe con caching aggressivo; Pocock ottimizza loop di task brevi e ripetibili.

## Connessioni

- [[concepts/agentic-patterns]] — i pattern agentic si basano su una buona gestione del contesto; smart zone come vincolo strutturale del workflow
- [[concepts/architectural-governance]] — l'architettura dichiarativa (ADR, eventmodel.json) è una forma di contesto strutturato per gli agenti
- [[concepts/spec-driven-development]] — la spec (requirements + design + tasks) è la struttura di contesto per task agentic complessi; lo steering è il contesto permanente del progetto
- [[concepts/deep-modules]] — deep modules riducono i token necessari all'AI per capire il sistema, preservando la smart zone
