---
tags:
  - eval-driven-development
  - genai
  - llm-evaluation
  - ai-engineering
  - testing
feature:
type: article
author: Rohit Girme / Airbnb Engineering
source: https://airbnb.tech/ai-ml/eval-driven-development-lessons-from-evaluating-genai-at-scale/
date: 2026-09-13
---

# Eval-Driven Development: Lessons from Evaluating GenAI at Scale

## Sunto

Airbnb ha sviluppato il concetto di **Eval-Driven Development (EDD)** come analogo del Test-Driven Development per i sistemi di AI generativa. L'approccio centrale è "when in doubt, look at your data": i team dovrebbero esaminare manualmente gli output di almeno 100 esempi prima di costruire qualsiasi infrastruttura di valutazione formale, per sviluppare intuizione su cosa significhi "successo" in quel contesto specifico.

I cinque principi fondamentali dell'EDD sono: definire gli obiettivi in anticipo (stabilire criteri di ottimizzazione e di rilascio), lasciare che gli errori guidino le metriche (sviluppare le dimensioni di valutazione a partire dai fallimenti osservati, non da assunzioni teoriche), mantenere gli evaluator affilati (3-5 evaluator ben calibrati battono 20-30 evaluator rumorosi), nominare un decision-maker (includere arbitraggio umano per i disaccordi su cosa sia output accettabile), e collaborare continuamente (mantenere un dialogo cross-funzionale costante sul comportamento del sistema).

La strategia di valutazione a tre livelli è il cuore del framework: il **Livello 1** usa controlli programmatici deterministici (validazione del formato, vincoli di lunghezza, filtraggio di parole chiave, metriche ML classiche come precisione/richiamo); il **Livello 2** usa un LLM-as-Judge — un modello più potente che valuta gli output contro rubric progettate con cura per qualità sfumate come tono, coerenza e fedeltà; il **Livello 3** usa la valutazione umana da parte di esperti del dominio per i casi ad alto rischio e per risolvere i disaccordi tra i livelli automatici.

Per i sistemi agentici multi-step, la valutazione richiede tre livelli aggiuntivi: **step-level** (singole chiamate a tool e passi di ragionamento), **trajectory-level** (ragionevolezza ed efficienza del percorso complessivo), e **session-level** (se l'interazione ha raggiunto l'obiettivo dell'utente). L'approccio usa trace analysis e tree traversal per esaminare gli stati intermedi, non solo gli output finali.

Un esempio pratico illustra l'intero processo: per un assistente di policy di supporto, la revisione manuale di 100 output ha identificato 15 allucinazioni, 8 risposte eccessivamente verbosi, 5 over-refusal e 3 fallimenti di formato. Sono stati sviluppati controlli programmatici per formato e lunghezza, virtual judge separati per fedeltà e concisione, e 60 esempi etichettati da esperti del dominio. L'iterazione ha migliorato l'agreement del giudice dal 78% all'88%. In produzione, il monitoring scala a 5.000 esempi con campionamento del 5% del traffico giornaliero.

## Codice

La strategia di calibrazione del virtual judge:

```
Processo di calibrazione:
  1. Creare golden dataset: 50-100 esempi (inclusi fallimenti)
  2. Eseguire il virtual judge contro il dataset
  3. Misurare l'agreement:
     - Target: high 80s-90s percentages
     - Metriche: Cohen's kappa o Krippendorff's alpha
  4. Analizzare i disaccordi e raffinare i prompt
  5. Ricalibrare periodicamente man mano che le failure mode evolvono

Regola critica: "Rubric design matters. Ambiguity is the enemy"
```

Regole fondamentali per l'LLM-as-Judge:

```
Regole per un LLM-as-Judge efficace:
  ✓ Un evaluator per dimensione (non "God evaluators" universali)
  ✓ Modello giudice diverso dal modello generatore
  ✓ Few-shot examples nei prompt di valutazione
  ✓ Schema di output esplicito e strutturato
  ✓ Definizioni chiare del punteggio
  ✗ NON costruire evaluator generici o multi-dimensionali
  ✗ NON scalare prima di raggiungere alta calibrazione
```

Framework di valutazione per sistemi agentici:

```
Livello 1 - Step-level:
  - Singole chiamate a tool
  - Passi di ragionamento individuali
  - Tool call correctness

Livello 2 - Trajectory-level:
  - Ragionevolezza del percorso complessivo
  - Efficienza della traiettoria
  - Tree traversal dell'execution trace

Livello 3 - Session-level:
  - L'utente ha raggiunto il suo obiettivo?
  - Task completion rate
  - Analisi delle sessioni end-to-end
```
