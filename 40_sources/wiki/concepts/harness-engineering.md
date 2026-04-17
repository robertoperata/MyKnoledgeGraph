---
title: Harness Engineering
type: concept
tags: [thread4-ai]
sources:
  - "[[dear_architect-harness-engineering]]"
  - "[[infoq_enterprise-spec-driven-development-adoption]]"
updated: 2026-04-17
related:
  - "[[concepts/agentic-patterns]]"
  - "[[concepts/spec-driven-development]]"
  - "[[concepts/architectural-governance]]"
---

# Harness Engineering

## Definizione

L'insieme dei sistemi che i team costruiscono *intorno* agli agenti di coding AI per aumentare la fiducia nel codice generato. L'harness non è una configurazione statica ma una pratica ingegneristica continua.

## Struttura dei controlli

### Feedforward vs Feedback

| Tipo | Descrizione | Esempio |
|---|---|---|
| **Feedforward** | Guide che anticipano comportamenti indesiderati prima che l'agente agisca | Linter rules, CLAUDE.md, pattern template |
| **Feedback** | Sensori che osservano i risultati e abilitano la correzione | Test suite, mutation testing, architettura review |

Entrambi sono necessari: solo guide = regole senza verifica; solo sensori = errori ripetuti.

### Computazionale vs Inferenziale

| Tipo | Caratteristica | Uso |
|---|---|---|
| **Computazionale** | Deterministico, veloce | Linter, type checker, test |
| **Inferenziale** | Semantico via LLM, più lento | Rilevare problemi di significato, violazioni architetturali |

## Tre categorie di harness

1. **Maintainability Harness** — qualità del codice (già matura: linter, code review bot)
2. **Architecture Fitness Harness** — fitness function per caratteristiche architetturali (performance, osservabilità)
3. **Behaviour Harness** — correttezza funzionale; l'area più debole, ancora troppo affidata a test generati da AI

## Steering Loop

Quando un problema si ripresenta, il team migliora guide o sensori invece di supervisionare manualmente. Il loop:

```
Problema identificato
    → Classificazione (Spec-to-Impl gap o Intent-to-Spec gap)
    → Raffinamento dell'harness (guide o sensori)
    → Riduzione futura degli stessi problemi
```

## Distribuzione temporale ("keep quality left")

| Fase | Controlli |
|---|---|
| Pre-commit | Linter, revisioni base |
| Post-integrazione | Mutation testing, architettura review |
| Produzione | Scansione dipendenze, monitoring continuo |

## Harnessability e Ambient Affordances

Ambienti con linguaggi fortemente tipizzati, confini di modulo chiari e framework opinati offrono agli agenti una base "leggibile". I **harness template** pre-costruiti per topologie comuni (API CRUD, event processor) abbassano il costo di adozione.

> "Gli esseri umani portano un harness implicito — convenzioni assorbite, giudizio estetico sulla complessità — che gli agenti non possiedono."

## Connessioni

- [[concepts/agentic-patterns]] — l'harness è il sistema di governance intorno ai pattern agentici
- [[concepts/spec-driven-development]] — in SDD, l'harness valida gli artefatti generati dalla spec e misura i gap
- [[concepts/architectural-governance]] — fitness function come forma di harness architetturale
