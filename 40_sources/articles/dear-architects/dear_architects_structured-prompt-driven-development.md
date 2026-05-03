---
tags:
  - prompt-engineering
  - ai-assisted-development
  - software-methodology
  - thoughtworks
  - llm
feature:
type: article
author: Wei Zhang, Jessie Jie Xia
source: https://martinfowler.com/articles/structured-prompt-driven/
date: 2026-05-03
---

# Structured Prompt-Driven Development (SPDD)

## Sunto

La Structured Prompt-Driven Development (SPDD) è una metodologia ingegneristica proposta da Wei Zhang e Jessie Jie Xia di Thoughtworks che tratta i prompt come artefatti di consegna di prima classe, conservati nel controllo di versione. L'approccio nasce dall'esigenza di rendere il codice generato dall'AI governabile e manutenibile su scala organizzativa, superando il limite delle interazioni ad hoc con i modelli linguistici. Invece di affidarsi a conversazioni episodiche con gli agenti di coding, SPDD sistematizza la creazione, la revisione e il riuso dei prompt per costruire una base di conoscenza organizzativa duratura.

Il cuore della metodologia è il framework **REASONS Canvas**, un template strutturato in sette parti: **R**equirements (definizione del problema e definition of done), **E**ntities (modello di dominio e relazioni), **A**pproach (strategia della soluzione), **S**tructure (architettura del sistema e componenti), **O**perations (passi di implementazione testabili a livello di metodo), **N**orms (standard trasversali: naming conventions, osservabilità, defensive coding), e **S**afeguards (vincoli non negoziabili: invarianti, soglie di performance, regole di sicurezza). Le prime quattro componenti coprono l'intent e il design (aspetti astratti), mentre le ultime tre gestiscono l'esecuzione e la governance.

Il workflow SPDD è supportato dallo strumento CLI `openspdd` che implementa comandi come `/spdd-story` per decomporre i requisiti in user story conformi ai criteri INVEST, `/spdd-reasons-canvas` per generare il template di specifica completo, `/spdd-generate` per produrre codice seguendo rigorosamente le specifiche del canvas, e `/spdd-sync` per sincronizzare le ottimizzazioni del codice di ritorno nella documentazione dei prompt. Il ciclo si chiude in entrambe le direzioni: le modifiche al codice aggiornano i prompt, e i requisiti modificati rigenerano il codice.

L'articolo distingue nettamente tra **correzioni logiche** (che cambiano il comportamento osservabile) e **refactoring** (ottimizzazioni interne senza cambio di comportamento). La regola aurea per le correzioni logiche è: aggiornare prima il prompt, poi rigenerare il codice. Per il refactoring invece si modifica prima il codice e poi si usa `/spdd-sync` per aggiornare la documentazione. Questo garantisce che il canvas rimanga sempre una specifica accurata e non un artefatto storico obsoleto — il principio fondamentale è: *"When reality diverges, fix the prompt first — then update the code."*

SPDD è altamente raccomandato per sistemi di delivery standardizzati e scalabili, ambienti ad alta conformità con vincoli rigidi, e team multi-persona che richiedono piena tracciabilità e auditabilità. Non è adatto per hotfix in produzione, prototipi esplorativi, o domini con regole di business mal definite. La metodologia richiede competenze senior per la progettazione delle astrazioni, imponendo un cambio mentale fondamentale da *"code-first"* a *"design-first"*: le gerarchie di oggetti, i pattern di collaborazione e i confini del sistema devono essere progettati prima di avviare la generazione del codice.

## Codice

Il ciclo completo di sviluppo SPDD per una funzionalità di billing engine con supporto a prezzi per modello e tier di abbonamento:

```bash
# Step 1: generare user story dal requirements document
/spdd-story enhancement-description.md

# Step 2: analisi del dominio e identificazione dei rischi
/spdd-analysis story.md

# Step 3: generare il canvas REASONS completo
/spdd-reasons-canvas analysis.md

# Step 4: generare l'implementazione dal canvas
/spdd-generate reasons-canvas.md

# Step 5: generare test unitari sincronizzati con l'implementazione
/spdd-api-test reasons-canvas.md implementation/

# Step 6: sincronizzare refactoring del codice nel canvas
/spdd-sync implementation/ reasons-canvas.md
```

I comandi principali del tool `openspdd` e il loro scopo:

```
/spdd-story          → Decompone requisiti in user story INVEST-compliant
/spdd-analysis       → Estrae keyword di dominio, analisi strategica, risk assessment
/spdd-reasons-canvas → Genera il template a sette parti REASONS
/spdd-generate       → Produce codice seguendo rigorosamente le specifiche del canvas
/spdd-prompt-update  → Raffina iterativamente il canvas al cambio dei requisiti
/spdd-sync           → Sincronizza refactoring del codice → documentazione prompt
/spdd-api-test       → Genera test strutturati (normal, boundary, error scenarios)
```
