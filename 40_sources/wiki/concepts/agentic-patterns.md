---
title: Agentic Patterns
type: concept
tags: [thread4-ai]
sources:
  - "[[vercel_agent-responsibly]]"
  - "[[AI Talks - Lada Kesseler Augmented Coding]]"
  - "[[Claude Code and Large-Context Reasoning]]"
  - "[[tessl_context-development-lifecycle]]"
updated: 2026-04-17
related:
  - "[[concepts/context-management]]"
  - "[[concepts/architectural-governance]]"
  - "[[concepts/harness-engineering]]"
  - "[[concepts/spec-driven-development]]"
  - "[[concepts/autonomous-systems-architecture]]"
---

# Agentic Patterns

## Definizione

Pattern ricorrenti nell'uso efficace di agenti AI per il coding e la produzione di software. La distinzione fondamentale è tra **relying** (delegare il giudizio all'agente) e **leveraging** (usare l'agente mantenendo piena ownership).

## Pattern fondamentali (Lada Kesseler)

### Anti-pattern: Context Rot
Il contesto si degrada nel tempo. Soluzione: creare `knowledge.md` con le risposte buone, ricominciare conversazioni nuove con il contesto rilevante.

### Anti-pattern: Distracted Agent
L'agente si disperde su troppi task contemporaneamente.

### Pattern: Focused Agent
Scope più piccolo = risultati migliori. **Una cosa alla volta.** Documenti separati per stile (`code_style.md`), processo (`tdd_process.md`).

### Pattern: Knowledge Composition
Comporre contesti diversi: `CLAUDE.md` a livello utente o progetto per le ground rules.

### Pattern: Semantic Zoom
- **Zoom in**: "tell me more about", "how do you write", "explain better"
- **Zoom out**: "make it shorter", "describe architecture at high level", "give me a TLDR"

### Pattern: Noise Cancellation
Combattere la verbosità default degli LLM: dare istruzioni esplicite per la brevità, eliminare senza pietà (`delete mercilessly`).

### Pattern: Knowledge Checkpoint
`save → commit → implement`. Il commit git è il checkpoint: permette di tornare a uno stato noto.

### Pattern: Parallel Implementations
Esplorare più implementazioni in parallelo prima di scegliere.

## Relying vs Leveraging (Vercel)

> "Il futuro appartiene agli ingegneri che mantengono un giudizio spietato su ciò che rilasciano, non a quelli che generano più codice."

| Approccio | Comportamento |
|---|---|
| **Relying** | Ship se i test passano; PR enormi che i reviewer non capiscono davvero |
| **Leveraging** | Piena ownership, comprensione dell'impatto in produzione, prontezza per un incident |

Test pratico: *saresti a tuo agio nell'essere responsabile di un incident legato a questo codice?*

## Tre linee di difesa per deployment sicuro

1. **Self-driving deployments**: canary release + rollback automatico
2. **Continuous validation**: load test e chaos experiment come pratica continua
3. **Executable guardrails**: la conoscenza operativa codificata come strumenti (non documentazione)

## Context Window e Large-Context Reasoning

La context window non è FIFO: i LLM mantengono più strettamente il materiale iniziale e quello più recente. Il materiale in mezzo è più soggetto all'eviction.

Implicazione pratica: le istruzioni critiche (`CLAUDE.md`, system prompt) vanno all'inizio; i dettagli rilevanti al task corrente vanno verso la fine della conversazione.

## Multi-Agent Architecture (Spotify Ads)

Pattern da Spotify per sistemi multi-agent:

**Tool Grounding**: i modelli LLM hanno accesso strutturato a dati reali tramite function calling — gli agenti ragionano su *cosa* fare, i tool forniscono i *fatti*. Previene le allucinazioni.

**Separazione intent/esecuzione**: gli agenti interpretano l'intenzione di business e orchestrano dinamicamente le chiamate — coerenza su superfici diverse (UI, Salesforce, Slack) senza duplicare la logica.

**Prompt engineering come software engineering**: i prompt sono versioned, testati e iterati con la stessa disciplina del codice. Piccole variazioni di formulazione impattano significativamente la consistenza dell'output.

**Regola per i confini degli agenti**: un agente per skill distinta o sorgente di dati. Troppi agenti → overhead di coordinazione; troppo pochi → prompt monolitici.

Risultati Spotify: tempo creazione media plan 15-30min → 5-10sec; 20+ campi di form → 1-3 messaggi in linguaggio naturale.

## Connessioni

- [[concepts/context-management]] — la gestione del contesto è il tema centrale dell'era agentica
- [[concepts/architectural-governance]] — governance dichiarativa applicabile anche agli agenti
- [[concepts/harness-engineering]] — i sistemi intorno agli agenti per garantire la qualità del codice generato
- [[concepts/spec-driven-development]] — SDD come paradigma per governare i coding agent
- [[concepts/autonomous-systems-architecture]] — architettura dei confini per sistemi multi-agent autonomi
- [[technologies/kubernetes]] — deployment sicuro con canary release è infrastruttura K8s
