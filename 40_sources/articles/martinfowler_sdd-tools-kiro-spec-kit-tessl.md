---
tags:
  - ai
  - agentic
  - developer-experience
  - architecture
type: article
author: Birgitta Böckeler
source: https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html
date: 2025-10-15
---

# Understanding Spec-Driven Development: Kiro, spec-kit, and Tessl

**Serie:** Exploring Gen AI (martinfowler.com)

## Sunto

Birgitta Böckeler analizza tre tool che si dichiarano implementazioni di *Spec-Driven Development* (SDD): **Kiro** (IDE di Amazon), **spec-kit** (approccio GitHub), e **Tessl Framework** (private beta). Il punto di partenza è che "la definizione di SDD è ancora in evoluzione" — e questo è il problema centrale del pezzo.

L'autrice propone una tassonomia a tre livelli crescenti di ambizione: (1) *spec-first* — le specifiche vengono scritte prima che l'AI generi codice; (2) *spec-anchored* — le specifiche persistono dopo il completamento del task e guidano l'evoluzione futura della feature; (3) *spec-as-source* — le specifiche diventano l'artefatto primario e gli umani non toccano mai il codice generato, solo le spec. I tre tool si posizionano diversamente su questo spettro.

**Kiro** segue un workflow leggero in tre step (Requirements → Design → Tasks, tre documenti markdown) e introduce il concetto di "steering" come memory bank per il contesto di progetto. **Spec-kit** opera tramite slash command nei coding assistant, enfatizza una "constitution" (principi architetturali immutabili), e produce più file markdown interdipendenti nelle fasi specify, plan e tasks. **Tessl Framework** spinge verso lo spec-as-source con sincronizzazione bidirezionale tra spec e codice, ma è ancora in private beta.

Le preoccupazioni sollevate da Böckeler sono concrete e praticamente rilevanti: i workflow attuali potrebbero non scalare bene su problemi di dimensioni variabili; la produzione di estesa documentazione markdown rischia di diventare più onerosa della tradizionale code review; gli agenti ignorano frequentemente le specifiche o le over-interpretano; il confine tra specifica funzionale e tecnica è spesso poco chiaro; e non è chiaro se SDD sia pensato per developer-led requirements analysis o per lavoro specializzato di product.

Il parallelo storico più interessante è con il **Model-Driven Development** (MDD): l'autrice nota che gli LLM riducono i vincoli che hanno affossato MDD (rigidità dei metamodelli, limitata espressività dei linguaggi di modellazione) ma introducono nuove sfide attraverso la non-determinismo. La conclusione è cauta: l'approccio spec-first ha valore reale, ma "spec-driven development" rimane un termine senza definizione precisa. Il rischio è che l'elaborata tooling amplifichi i problemi esistenti invece di risolverli — il tedesco ha un termine per questo: *Verschlimmbesserung* (peggiorare tentando di migliorare).

---

## I tre livelli di SDD

| Livello | Definizione | Relazione umano/AI |
|---|---|---|
| **Spec-first** | Le spec precedono la generazione del codice | Umano scrive spec → AI genera codice |
| **Spec-anchored** | Le spec persistono e guidano l'evoluzione futura | Umano mantiene spec + codice; spec come punto di ancoraggio |
| **Spec-as-source** | Le spec sono l'unico artefatto umano; il codice è tutto generato | Umano mantiene solo spec; AI sincronizza codice |

## Confronto dei tre tool

| Tool | Produttore | Livello SDD | Struttura | Elemento chiave |
|---|---|---|---|---|
| **Kiro** | Amazon | Spec-first / anchored | 3 markdown: Requirements, Design, Tasks | "Steering" come memory bank |
| **Spec-kit** | GitHub | Spec-first / anchored | Multi-file: specify, plan, tasks | "Constitution" — principi architetturali immutabili |
| **Tessl Framework** | Tessl | Spec-as-source (ambizioso) | Sync bidirezionale spec ↔ codice | Sincronizzazione automatica (private beta) |

## Cosa distingue le spec dai memory bank

Böckeler propone una distinzione importante:
- **Spec**: artefatti strutturati, orientati al comportamento, scritti in linguaggio naturale, che esprimono funzionalità software come guida agli agenti AI. Sono relativi a una feature/task specifica.
- **Memory bank** (es. steering in Kiro): documentazione contestuale di progetto (architecture guide, convenzioni, decisioni tecniche). Non descrive una feature, ma il contesto entro cui le feature vanno sviluppate.

## Preoccupazioni principali

1. **Scalabilità del workflow**: workflow pensati per task di media complessità potrebbero non adattarsi bene a problemi molto piccoli (troppo overhead) o molto grandi (spec insufficienti)
2. **Review burden**: produrre estesa documentazione markdown può superare il costo tradizionale della code review, specialmente in team abituati a lavorare direttamente sul codice
3. **Instruction adherence**: gli agenti ignorano frequentemente le spec o le interpretano in modo eccessivamente letterale o eccessivamente creativo
4. **Confine funzionale/tecnico**: non è chiaro dove finisce la specifica funzionale e inizia quella tecnica — e chi è responsabile di ciascuna
5. **Target audience ambiguo**: SDD è per developer che vogliono fare requirements analysis assistita da AI, o per specialist di product/BA che vogliono generare codice da spec di business?

## Parallelo con Model-Driven Development (MDD)

> "LLMs reduce MDD's constraints while introducing new challenges through non-determinism."

MDD ha storicamente fallito per rigidità dei metamodelli e limitata espressività dei linguaggi di modellazione. Gli LLM rimuovono questi vincoli (il linguaggio naturale è espressivo per definizione) ma introducono non-determinismo: due esecuzioni della stessa spec possono produrre codice diverso, e gli agenti possono non rispettare le istruzioni.

La sfida tecnica si sposta da "il modello è abbastanza espressivo?" a "l'agente segue le istruzioni in modo affidabile?"

---

## Link esterni

- [Kiro IDE](https://kiro.dev) — IDE di Amazon con workflow SDD integrato (Requirements → Design → Tasks)
- [spec-kit (GitHub)](https://github.com/github/spec-kit) — tool GitHub per SDD tramite slash command nei coding assistant
- [Tessl Framework](https://tessl.io) — framework spec-as-source con sync bidirezionale spec/codice (private beta)
- [Exploring Gen AI series](https://martinfowler.com/articles/exploring-gen-ai/) — serie di articoli di Böckeler su Martin Fowler's site sul tema GenAI per sviluppatori

---

## Immagini

Nessuna immagine presente
