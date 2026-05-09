---
title: Spec-Driven Development
type: concept
tags: [thread4-ai]
sources:
  - "[[infoq_spec-driven-development-architecture-executable]]"
  - "[[infoq_enterprise-spec-driven-development-adoption]]"
  - "[[martinfowler_sdd-tools-kiro-spec-kit-tessl]]"
  - "[[youtube_kiro-spec-driven-development-al-harris]]"
updated: 2026-05-08
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

## Formato EARS per i requisiti (Kiro)

**EARS** (Easy Approach to Requirements Syntax) è una sintassi strutturata in linguaggio naturale per scrivere requisiti che siano sia leggibili da umani sia processabili da automated reasoning:

```
When <condizione>, the system shall <comportamento>.
```

La struttura permette operazioni **non-LLM** sui requisiti:
- Rilevamento di **ambiguità** e requisiti in conflitto
- Estrazione automatica di **correctness properties** (invarianti)
- Traduzione diretta in **property-based tests**

> "Our goal is to use the LLM for less and less over time. We want to use classic automated reasoning techniques to give you high quality results." — Al Harris (Kiro)

## Property-Based Testing dai requisiti (Kiro)

Il link EARS → property tests chiude il loop requisito → codice:

1. Kiro estrae invarianti dal documento requirements (formato EARS)
2. Gli invarianti diventano **property-based tests** (Hypothesis/Python, fast-check/Node, clojure.spec)
3. Un property test cerca il contro-esempio che falsifica l'invariante — se non lo trova, alta confidenza che il requisito sia soddisfatto
4. I task vengono eseguiti solo se le proprietà passano

**Vantaggio vs unit test classici:** i PBT sono generativi — producono automaticamente casi di test che un umano non scriverebbe. Non testano "questo input produce quell'output" ma "questa proprietà vale per tutti gli input possibili".

## Spec come living documentation (Kiro)

Le spec non si accumulano ma vengono **mutate** quando la feature evolve — come un diff sul documento originale, non come un nuovo documento. Questo permette:
- Review incrementali: vedere esattamente cosa è cambiato rispetto alla versione precedente
- Storia della feature attraverso i diff delle spec, non solo del codice
- Eliminazione delle spec di esperimenti/feature abbandonate

Il team Kiro usa internamente **spec review** invece di design session: la spec viene postata su wiki (via MCP) e il team la commenta, poi l'implementazione segue.

## Backend neurosimbolico (Kiro)

Kiro non è puramente LLM-driven ma usa **neurosymbolic reasoning**:

| Operazione | Tipo di reasoning |
|---|---|
| Generazione requisiti, design, codice | LLM |
| Validazione requisiti (ambiguità, conflitti) | Automated reasoning classico |
| Estrazione correctness properties | Automated reasoning classico |
| Chat libera | LLM |

Obiettivo a lungo termine: ridurre progressivamente la dipendenza dall'LLM per le operazioni che possono essere rese deterministiche.

## Tassonomia a tre livelli (Böckeler, 2025)

Birgitta Böckeler propone una gradazione utile per distinguere le ambizioni dei tool SDD:

| Livello | Definizione | Relazione umano/codice |
|---|---|---|
| **Spec-first** | Le spec precedono la generazione del codice per il task | Umano scrive spec → AI genera codice; umano poi mantiene entrambi |
| **Spec-anchored** | Le spec persistono dopo il task e guidano l'evoluzione futura della feature | Le spec sono punto di ancoraggio permanente per il contesto dell'agente |
| **Spec-as-source** | Le spec sono l'unico artefatto umano; il codice è interamente generato e sincronizzato | Umano tocca solo spec; AI mantiene il codice in sync |

## Spec vs Memory Bank: distinzione critica

- **Spec**: artefatto strutturato, orientato al comportamento, relativo a una specifica feature/task. Guida l'agente AI su *cosa fare*.
- **Memory bank / steering**: documentazione contestuale di progetto (architecture guide, convenzioni, decisioni tecniche). Fornisce il *contesto* entro cui le spec vengono eseguite.

I tool SDD tendono a sovrapporre i due concetti. La distinzione è importante perché hanno cicli di vita e audience diversi: le spec cambiano per ogni feature, il memory bank cambia con l'architettura.

## Tool attuali (2025)

| Tool | Produttore | Livello | Struttura | Caratteristica distintiva |
|---|---|---|---|---|
| **Kiro** | Amazon | Spec-first/anchored | Requirements → Design → Tasks (3 md) | "Steering" come memory bank |
| **Spec-kit** | GitHub | Spec-first/anchored | Specify → Plan → Tasks (multi-file) | "Constitution": principi architetturali immutabili |
| **Tessl** | Tessl | Spec-as-source | Sync bidirezionale spec ↔ codice | Ambizione massima; private beta |

## Parallelo con Model-Driven Development (MDD)

Gli LLM riducono i vincoli storici di MDD (rigidità dei metamodelli, scarsa espressività dei linguaggi di modellazione) ma introducono una nuova sfida: il **non-determinismo**. Due esecuzioni della stessa spec possono produrre codice diverso; gli agenti possono ignorare o over-interpretare le istruzioni.

> La sfida si sposta da "il modello è abbastanza espressivo?" a "l'agente segue le istruzioni in modo affidabile?"

## Preoccupazioni aperte (Böckeler)

- **Instruction adherence**: gli agenti ignorano frequentemente le spec o le interpretano in modo eccessivamente letterale/creativo — il problema fondamentale che tutti i tool cercano di risolvere ma nessuno ha risolto
- **Review burden**: produrre estesa documentazione markdown può costare più della code review tradizionale
- **Scalabilità del workflow**: workflow pensati per task medi non si adattano bene a problemi molto piccoli o molto grandi
- **Confine funzionale/tecnico**: dove finisce la specifica funzionale e inizia quella tecnica? Chi la scrive?
- **Verschlimmbesserung**: il rischio che la tooling elaborata amplifichi i problemi esistenti invece di risolverli

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
