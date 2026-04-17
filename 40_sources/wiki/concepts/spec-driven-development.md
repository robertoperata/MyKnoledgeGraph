---
title: Spec-Driven Development
type: concept
tags: [thread4-ai]
sources:
  - "[[infoq_spec-driven-development-architecture-executable]]"
  - "[[infoq_enterprise-spec-driven-development-adoption]]"
updated: 2026-04-17
related:
  - "[[concepts/agentic-patterns]]"
  - "[[concepts/harness-engineering]]"
  - "[[concepts/architectural-governance]]"
---

# Spec-Driven Development (SDD)

## Definizione

SDD è un paradigma architetturale che inverte il tradizionale source of truth: invece di far emergere la verità dal codice, la verità è definita nelle **specifiche eseguibili**, e il codice ne diventa la materializzazione. È la quinta generazione di astrazione del software, resa possibile dall'AI generativa.

> "The unit of delivery is no longer a service or a codebase. The unit of delivery becomes the specification itself."

## Evoluzione verso SDD

| Fase | Caratteristica |
|---|---|
| **Vibe coding** | Iterazione prompt-by-prompt, implementazione come contesto per successive correzioni |
| **Plan mode** | L'AI elabora un piano per revisione umana prima di scrivere codice |
| **SDD** | Le specifiche come dialogo tra umani e agenti AI, non semplici istruzioni |

## Architettura: i 5 strati

1. **Specification Layer** — definizione autoritativa del comportamento
2. **Generation Layer** — trasformazione dell'intento dichiarativo in forma eseguibile
3. **Artifact Layer** — output concreti (codice generato, modelli, adattatori)
4. **Validation Layer** — applicazione continua dell'allineamento intento/esecuzione
5. **Runtime Layer** — il sistema operativo

## Modello classico vs SDD

| Modello Classico | Modello SDD |
|---|---|
| Il codice definisce il comportamento | La specifica definisce il comportamento |
| L'architettura è consigliativa | L'architettura è eseguibile e applicabile |
| La deriva è scoperta dopo il fatto | La deriva è prevenuta pre-runtime |
| La validazione è retrospettiva | La validazione è continua |

## Drift Detection

Capacità architettonica di rilevare divergenze tra intento dichiarato e comportamento osservato. Include: validazione dello schema, ispezione payload, contract testing, integrazione CI pipeline.

## SpecOps — capacità fondamentali

1. Spec authoring come superficie di engineering di prima classe
2. Validazione formale e type enforcement
3. Generazione deterministica (spec identiche → artefatti identici)
4. Conformance continua a runtime
5. Evoluzione governata con controllo esplicito della compatibilità

## Responsabilità per ruolo (enterprise)

| Fase | Responsabile | Focus |
|---|---|---|
| **Discover** (What) | Product Owner + AI | Intento aziendale, criteri di accettazione |
| **Design** (How) | Architect + AI | Approccio tecnico, decomposizione multi-repo |
| **Tasks** | Developer + AI | Dettagli implementativi, schema, moduli |

## Harness Governance

I bug diventano metriche di qualità dell'harness:
- **Spec-to-Implementation Gap**: spec chiara, implementazione divergente → rafforzare validazione
- **Intent-to-Spec Gap**: dettaglio perso durante la deliberazione → migliorare estrazione della spec

## Trade-off

- Le spec ereditano complessità, debito tecnico e inerzia di compatibilità del codice
- La fiducia nel generatore diventa un problema di supply chain
- La validazione runtime ha costo computazionale
- Il cambio cognitivo per gli engineer è significativo
- Rischio "SpecFall": spec markdown obsolete senza cambio culturale

## Connessioni

- [[concepts/agentic-patterns]] — SDD è il paradigma di governance per i coding agent
- [[concepts/harness-engineering]] — l'harness garantisce la qualità degli artefatti generati dalla spec
- [[concepts/architectural-governance]] — SDD estende la governance dichiarativa al livello delle specifiche
