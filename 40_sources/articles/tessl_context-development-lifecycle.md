---
tags:
  - ai
  - agents
  - context
  - devops
  - ai-coding
feature:
type: article
author: Patrick Debois (tessl.io)
source: https://tessl.io/blog/context-development-lifecycle-better-context-for-ai-coding-agents/
---

# Context Development Lifecycle (CDLC)

## Il problema

Man mano che gli agenti AI diventano capaci di scrivere codice vero, il collo di bottiglia si sposta: non è più la *generazione del codice*, ma la *qualità del contesto*. Oggi il contesto è sparso ovunque — `.cursorrules`, `CLAUDE.md`, Slack, wiki — senza versioning, test o conflict resolution. È esattamente come veniva gestito il codice prima del version control e del CI/CD.

## Il framework CDLC in 4 fasi

### 1. Generate — Rendere esplicita la conoscenza implicita
- Articolare standard tecnici, pattern architetturali, timeline, requisiti di compliance
- L'AI può fare una prima bozza, ma l'umano deve validare l'accuratezza

### 2. Evaluate — Testare il contesto come si testa il codice
- Trattare il contesto come artefatto misurabile con test (evals)
- TDD applicato al contesto: definisci il comportamento atteso, testa, migliora iterativamente
- Le evals permettono confronti tra modelli e rivelano ambiguità nelle specifiche

### 3. Distribute — Contesto come pacchetti versionati
- Distribuzione strutturata (stile npm/pip/cargo) per la condivisione a livello organizzativo
- Chi migliora il contesto vede benefici immediati → incentivi allineati
- Richiede controlli di sicurezza e versioning per prevenire il decadimento silenzioso

### 4. Observe — Imparare dall'uso reale
- Gli agenti in produzione rivelano gap che i test sintetici non intercettano
- Le scelte inattese dell'agente espongono ambiguità nelle specifiche
- Il feedback loop guida il raffinamento continuo

## Il parallelo con DevOps

DevOps (2009) ha unificato Development e Operations che lavoravano separati. Il CDLC fa lo stesso per la creazione e il consumo di contesto. Vantaggio strutturale rispetto al DevOps: chi scrive contesto migliore vede *immediatamente* output migliori dall'agente — l'interesse personale è allineato con quello organizzativo. Questo cambia tutto sul piano dell'adozione.

## Take Away

> "I team che vinceranno nell'era degli agenti non saranno quelli col modello migliore. Saranno quelli col contesto migliore."

Il contesto è il nuovo codice: deve essere versionato, testato, distribuito e osservato. Investire in capacità agentiche senza un'infrastruttura di contesto equivale a deployare un'app senza CI/CD — funziona finché è piccola, poi crolla sotto il proprio peso.
