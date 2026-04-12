---
title: Architectural Governance
type: concept
tags: [thread1-microservices, thread4-ai]
sources:
  - "[[infoq_architectural-governance-ai-speed]]"
updated: 2026-04-09
related:
  - "[[concepts/agentic-patterns]]"
  - "[[concepts/context-management]]"
  - "[[concepts/information-hiding]]"
---

# Architectural Governance

## Il problema

L'AI generativa ha accelerato la produzione di codice, creando un disallineamento: il codice è diventato una commodity, ma l'allineamento architetturale no. I processi tradizionali (architecture review board, esperti umani) non riescono a scalare alla velocità del codice prodotto da agenti AI.

## La soluzione: Architettura Dichiarativa

Tradurre le decisioni architetturali in **dichiarazioni eseguibili dalle macchine**, integrate direttamente negli editor, nelle pipeline CI/CD e negli strumenti di code review.

> "A declaration that can only be understood by a human still depends on that human being in the loop."

## Tre implementazioni pratiche

### 1. Event Models come specifiche eseguibili
File `eventmodel.json` con schema formale. Permettono:
- Generazione di codice deterministica da template
- Coordinamento multi-team
- Automazione tramite agenti AI su singole "slice" di dominio (Ralph Loop)

Connessione diretta con il [[concepts/domain-event]] e la [[concepts/ubiquitous-language]] del DDD.

### 2. Validatori OpenAPI per la coerenza delle API
Linter automatici per convenzioni (path, versioning, paginazione). La validazione passa da "guardiano" a "collaboratore". Connessione con [[concepts/information-hiding]]: lo schema esplicito è il contratto.

### 3. Architecture.md come manifesto eseguibile
Le ADR (Architecture Decision Records) distillate in direttive brevi e leggibili dalle macchine:
```
[ADR-088] [Warn] Require asynchronous Kafka events for inter-service communication
[ADR-004] [Block] Services must own their data
```
Gli agenti di governance sincronizzano il manifesto e lo applicano a runtime.

## Principi chiave

1. **Scope minimale**: dichiarazioni limitate abilitano comprensione umana e AI
2. **Governance embedded**: strumenti nelle pipeline → review board esterni bypassati
3. **Feedback loop**: le violazioni osservate guidano l'evoluzione architetturale
4. **Raffinamento continuo**: l'architettura si adatta con i requisiti tramite telemetria automatizzata

> "The conformant path should be the easiest path."

## Connessione con il CDLC

L'Architecture.md come manifesto eseguibile è una forma di [[concepts/context-management]] strutturato: le decisioni architetturali diventano contesto che gli agenti AI possono leggere e applicare, seguendo esattamente il CDLC di Patrick Debois (Distribute → Observe → Evaluate).

## Connessioni

- [[concepts/context-management]] — l'architettura dichiarativa è contesto strutturato per gli agenti
- [[concepts/agentic-patterns]] — gli executable guardrails di Vercel sono la stessa idea
- [[concepts/domain-event]] — event models come specifiche eseguibili si basano sui domain events
- [[concepts/information-hiding]] — lo schema esplicito (OpenAPI) è la forma di information hiding dell'architettura
