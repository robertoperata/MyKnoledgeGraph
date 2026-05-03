---
title: AI Governance Stack — SDD, Harness Engineering e Architectural Governance
type: synthesis
tags:
  - thread4-ai
  - thread1-microservices
sources:
  - "[[spec-driven-development]]"
  - "[[harness-engineering]]"
  - "[[architectural-governance]]"
  - "[[agentic-patterns]]"
  - "[[context-management]]"
  - "[[autonomous-systems-architecture]]"
updated: 2026-04-18
related:
  - "[[concepts/spec-driven-development]]"
  - "[[concepts/harness-engineering]]"
  - "[[concepts/architectural-governance]]"
  - "[[concepts/agentic-patterns]]"
  - "[[concepts/context-management]]"
  - "[[concepts/autonomous-systems-architecture]]"
---

# Domanda originale

**Come si combinano Spec-Driven Development, Harness Engineering e Architectural Governance per governare lo sviluppo AI-assisted in un team che produce microservizi?**

---

# Risposta sintetica

I tre concetti non sono alternativi: operano a livelli diversi dello stesso problema. La governance architetturale definisce i confini, le specifiche eseguibili (SDD) definiscono il comportamento all'interno di quei confini, l'harness verifica che il codice generato rispetti entrambi. Senza tutti e tre, uno qualsiasi degli altri è insufficiente.

---

## La tensione di fondo

> **Tensione documentata:** Lowgren (*Autonomous Systems Architecture*) descrive la governance come **definizione di confini** entro cui gli agenti operano. Griffin/Carroll (*Spec-Driven Development*) la descrivono come **specifiche eseguibili** che definiscono il comportamento in modo misurabile. I due approcci sono complementari: i confini definiscono l'envelope, le specifiche definiscono il comportamento all'interno.

Questa tensione riflette due domande diverse:
- *Cosa può fare l'agente?* → governance dei confini (Architectural Governance)
- *Cosa deve produrre l'agente?* → specifiche eseguibili (SDD)
- *Come verifico che lo abbia fatto?* → harness (Harness Engineering)

---

## Livello 1 — Architectural Governance: i confini

L'Architectural Governance definisce le regole che valgono per tutto il sistema, espresse in forma leggibile dalle macchine:

```
[ADR-088] [Warn] Require async Kafka events for inter-service communication
[ADR-004] [Block] Services must own their data
```

Queste dichiarazioni vengono integrate in editor, pipeline CI/CD e code review. L'obiettivo: *"the conformant path should be the easiest path"* — non un gate che blocca, ma un percorso che guida.

**Nel contesto dei microservizi:** le ADR sui bounded context, sull'ownership dei dati e sui pattern di comunicazione diventano regole che gli agenti AI devono rispettare quando generano codice. Un agente che genera un servizio che accede al database di un altro servizio viene bloccato dalla pipeline, non da un reviewer umano.

---

## Livello 2 — Spec-Driven Development: il comportamento

SDD inverte il source of truth: la specifica eseguibile è autorevole, il codice è la sua materializzazione. I cinque strati:

1. **Specification Layer** — definizione autoritativa del comportamento
2. **Generation Layer** — trasformazione in forma eseguibile
3. **Artifact Layer** — codice, modelli, adattatori generati
4. **Validation Layer** — allineamento continuo intento/esecuzione
5. **Runtime Layer** — il sistema operante

**Nel contesto dei microservizi:** ogni "slice" del dominio (un aggregate, un evento, un endpoint) ha una specifica. Gli agenti AI generano il codice da quella specifica. Il **Drift Detection** verifica continuamente che il comportamento osservato non diverga dall'intento dichiarato.

**Responsabilità per ruolo:**
- Product Owner + AI: *What* (intento aziendale, criteri di accettazione)
- Architect + AI: *How* (approccio tecnico, decomposizione multi-repo)
- Developer + AI: dettagli implementativi, schema, moduli

---

## Livello 3 — Harness Engineering: la verifica

L'harness è il sistema che verifica che il codice generato sia corretto — sia rispetto alle specifiche (SDD) sia rispetto ai confini architetturali (Architectural Governance).

**Struttura dei controlli:**
- **Feedforward** (anticipano): linter, CLAUDE.md, template per topologie comuni (API CRUD, event processor)
- **Feedback** (osservano): test suite, mutation testing, fitness function architetturali

**Tre categorie di harness:**
1. **Maintainability** — qualità del codice (matura)
2. **Architecture Fitness** — caratteristiche architetturali (es. servizi che non accedono a DB altrui)
3. **Behaviour** — correttezza funzionale (ancora debole)

Quando un bug appare in produzione, il team non supervisiona manualmente: migliora l'harness. I bug sono metriche della qualità dell'harness, non solo errori da correggere.

---

## Come i tre livelli si collegano al CDLC

Il Context Development Lifecycle di Patrick Debois attraversa tutti e tre i livelli:

```
Generate  →  Architectural Governance: rendere esplicite le ADR come contesto
Evaluate  →  Harness: testare il contesto con evals e fitness function
Distribute →  SDD: distribuire le specifiche come artefatti versionati
Observe   →  Harness: imparare dal comportamento reale degli agenti
```

Il Context Management è il substrato comune: le ADR, le specifiche e le regole dell'harness sono tutte forme di contesto strutturato che gli agenti AI leggono per operare correttamente.

---

## La connessione trasversale

```
Architectural Governance  →  confini (cosa non si può fare)
Spec-Driven Development   →  specifiche (cosa si deve produrre)
Harness Engineering       →  verifica (come si misura la qualità)
Context Management        →  substrato (come il contesto arriva agli agenti)
Agentic Patterns          →  operatività (come gli agenti usano tutto questo)
```

---

## Sorgenti utilizzate

- [[concepts/spec-driven-development]] — SDD come paradigma di governance
- [[concepts/harness-engineering]] — verifica del codice generato
- [[concepts/architectural-governance]] — governance dichiarativa e eseguibile
- [[concepts/agentic-patterns]] — pattern operativi degli agenti
- [[concepts/context-management]] — CDLC come substrato comune
- [[concepts/autonomous-systems-architecture]] — tensione confini vs specifiche

---

## Lacune / Argomenti da approfondire

- Evals framework per LLM: come costruire test sistematici per il contesto e gli output degli agenti
- Prompt versioning tools (es. PromptLayer, LangSmith)
- CI/CD pipeline per sistemi AI-assisted: come integrare harness inferenziali nella pipeline
- Contract testing per agenti AI: come verificare che l'output di un agente rispetti un contratto formale
- Observability per agenti: tracing distribuito di workflow multi-agent
- Security dell'harness: prompt injection, data exfiltration, jailbreak nei contesti aziendali
- Metriche per misurare la maturità dell'harness (Behaviour Harness Index)
