---
title: Async Workflow Patterns
type: concept
tags: [thread1-microservices]
sources:
  - "[[dear_architect-50k-user-journeys-async-workflows]]"
updated: 2026-04-17
related:
  - "[[patterns/event-driven]]"
  - "[[technologies/kafka]]"
  - "[[patterns/saga-pattern]]"
---

# Async Workflow Patterns

## Definizione

Pattern per la progettazione di sistemi di workflow asincrono scalabili. Basato sull'analisi di 50.000 utenti attivi, emerge un modello di maturità a tre livelli che mappa *quando* specifiche funzionalità diventano necessarie.

## Perché le code tradizionali non bastano

Le code tradizionali disaccoppiano l'accettazione del lavoro dall'esecuzione, ma non rispondono a:
- Fallimenti parziali
- Dati stantii durante il processing
- Rifiuti dell'API a valle
- Burst di eventi

I **durable execution engine** (es. Inngest, Temporal) offrono progresso checkpointato, retry automatici e controllo dell'esecuzione relativo alle altre funzioni.

## Modello di maturità a 3 livelli

### Tier 1 — Affidabilità Fondamentale
Presente in quasi tutti i deployment in produzione:

| Feature | Descrizione |
|---|---|
| Retry con backoff | Configurabili per funzione, non solo globali |
| Controllo concorrenza | Chiavi per evitare corruzione dello stato condiviso |
| Cron scheduling | Integrato con sistemi event-driven |
| Invocazione con attesa | Comporre workflow con dipendenze |

### Tier 2 — Flow Control (Guardrail)
Adottato inizialmente dal 12-20%, sale al 45-55% a scala:

| Feature | Descrizione |
|---|---|
| **Multi-trigger** | Una funzione risponde a più tipi di evento |
| **Throttle/Rate limit** | Distingue tra accodare o scartare il lavoro in eccesso |
| **Cancel** | Termina run stantii quando arrivano eventi più recenti |
| **Debounce** | Coalescenza burst in una singola esecuzione — riduce invocazioni dell'80-90% |
| **Replay** | Recuperare da deploy errati |
| **Timeout** | Tetto massimo alla durata di una run |

### Tier 3 — Ottimizzazione
Adottato da <3% inizialmente, ma dal 40% ai livelli di maturità più alti:

| Feature | Descrizione |
|---|---|
| **Batching** | Riduce overhead per-invocazione ad alto volume |
| **Singleton** | Garantisce esecuzione esclusiva di job critici |
| **Priorità** | Espressioni valutate a runtime per ordinamento guidato dalla business logic |

## Insight sull'adozione

Le funzionalità avanzate compaiono quasi sempre insieme a quelle fondamentali: è la complessità del problema — non la scopribilità dello strumento — a guidare l'adozione. Non si aggiunge il debounce perché si è letto un blog post, lo si aggiunge perché i burst di eventi hanno causato un incidente.

## Connessioni

- [[patterns/event-driven]] — i workflow asincroni si basano su architetture event-driven
- [[technologies/kafka]] — Kafka come substrate per il routing e il buffering dei workflow
- [[patterns/saga-pattern]] — le saga sono un caso speciale di workflow distribuito con compensazione
