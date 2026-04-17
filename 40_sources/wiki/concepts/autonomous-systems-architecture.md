---
title: Autonomous Systems Architecture
type: concept
tags: [thread4-ai, thread1-microservices]
sources:
  - "[[dear_architect-redefining-architecture-boundaries-matter-most]]"
updated: 2026-04-17
related:
  - "[[concepts/agentic-patterns]]"
  - "[[concepts/harness-engineering]]"
  - "[[concepts/architectural-governance]]"
  - "[[concepts/bounded-context]]"
---

# Autonomous Systems Architecture

## Definizione

Framework architetturale per sistemi in cui agenti AI operano con autonomia. La distinzione centrale è tra **automazione** (esecuzione di passi predeterminati) e **autonomia** (comportamento emergente e non deterministico). Inserire l'AI nei flussi procedurali è l'errore più comune: "autonomia e logica procedurale sono olio e acqua".

## Il cambio di paradigma: da passi a confini

Un architetto di sistemi autonomi non definisce i passi del processo — definisce i **confini** entro cui gli agenti operano. Le sette dimensioni dei confini:

1. **Scope** — cosa rientra nel dominio dell'agente
2. **Obiettivi** — cosa l'agente deve ottimizzare
3. **Autorità** — quali azioni può intraprendere
4. **Diritti decisionali** — quali decisioni può prendere autonomamente
5. **Policy** — regole che non può violare
6. **Rischio** — envelope di rischio accettabile
7. **Ontologia/Semantica** — significato condiviso dei termini

## Sicurezza ontologica

Se più agenti interpretano diversamente il termine "completato", il sistema "deriva e allucinera immediatamente". Le ontologie condivise e i meccanismi di context-sharing diventano misure di sicurezza strutturali, non mere convenzioni.

## Livelli di maturità AI (5 livelli)

| Livello | Descrizione |
|---|---|
| Level 1 | Assistenti AI ad hoc |
| Level 2 | AI con workflow definiti |
| Level 3 | Multi-agent con autonomia piena |

Il salto al Level 3 è un "passo davvero grande" che richiede nuovi linguaggi di design e un nuovo operating model. La maggior parte delle organizzazioni opera ai livelli 1-2.

## Pattern: LLM-Assisted Design Process

Invece di lavagne e post-it:
1. Il team definisce i confini in modo completo
2. Un LLM progetta i processi *all'interno* di quei confini
3. I processi generati vengono validati tramite testing adversariale

In un caso reale questo approccio ha generato automaticamente 27-33 agenti.

## Principio di governance

Governance e design devono essere "uniti all'anca" fin dall'inizio. Non si può aggiungere la governance a posteriori su sistemi autonomi in produzione.

> **Tensione con SDD:** Lowgren descrive la governance come definizione di confini; Griffin/Carroll (SDD) la descrivono come specifiche eseguibili. I due approcci sono complementari: i confini definiscono l'envelope, le specifiche definiscono il comportamento all'interno.

## Connessioni

- [[concepts/agentic-patterns]] — i pattern agentici operano all'interno dei confini definiti da questa architettura
- [[concepts/harness-engineering]] — l'harness implementa i confini come controlli tecnici
- [[concepts/architectural-governance]] — governance come definizione di confini, non regole procedurali
- [[concepts/bounded-context]] — i bounded context DDD sono un analogo per sistemi autonomi AI
