---
title: Context Management (AI)
type: concept
tags: [thread4-ai]
sources:
  - "[[tessl_context-development-lifecycle]]"
  - "[[AI Talks - Lada Kesseler Augmented Coding]]"
  - "[[Claude Code and Large-Context Reasoning]]"
updated: 2026-04-09
related:
  - "[[concepts/agentic-patterns]]"
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

## Connessioni

- [[concepts/agentic-patterns]] — i pattern agentic si basano su una buona gestione del contesto
- [[concepts/architectural-governance]] — l'architettura dichiarativa (ADR, eventmodel.json) è una forma di contesto strutturato per gli agenti
