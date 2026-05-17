---
tags:
  - ai-agents
  - multi-agent
  - harness-design
  - evaluator-pattern
  - context-management
  - llm-engineering
feature:
type: article
author: Prithvi Rajasekaran
source: https://www.anthropic.com/engineering/harness-design-long-running-apps
date: 2026-05-17
---

# Harness Design for Long-Running Application Development

## Sunto

Questo articolo di Anthropic esplora come l'architettura degli harness (sistemi di scaffolding multi-agente) influenzi radicalmente la qualità del codice prodotto autonomamente da Claude in sessioni di sviluppo di lunga durata. La premessa centrale è che un singolo agente lasciato a sé stesso tende a degradare progressivamente la qualità del lavoro man mano che il contesto si riempie, mentre un'architettura separata di generazione e valutazione produce risultati sostanzialmente superiori.

Sono stati identificati due pattern di degrado nei task agentici lunghi: la "context anxiety" (il modello inizia a concludere prematuramente il lavoro quando il context window si riempie) e il "self-evaluation bias" (il modello valuta il proprio lavoro come eccellente anche quando è mediocre). La soluzione ai problemi di contesto è il "context reset": invece di fare in-place summarization, si svuota la finestra di contesto e si usa un passaggio strutturato tra sessioni. Questa tecnica era essenziale per Sonnet 4.5, ma è diventata meno necessaria con Opus 4.6 che pianifica meglio e sostiene task agentici più lunghi.

Per il design frontend, l'autore ha creato un sistema Generator-Evaluator ispirato alle GAN. Il generatore produce UI, l'evaluator — che usa Playwright per interagire con le pagine live — valuta il risultato su quattro criteri: qualità del design (identità visiva coesa), originalità (decisioni custom vs template generici), artigianato (tipografia, spaziatura, colore), e funzionalità. Pesare maggiormente design e originalità spinge il modello verso output meno generici. Il ciclo viene iterato 5-15 volte per generazione.

Per lo sviluppo full-stack (V1), l'architettura a tre agenti (Planner, Generator, Evaluator) ha prodotto risultati nettamente superiori a un singolo agente ma a costo e durata molto maggiori: 6 ore e $200 contro 20 minuti e $9, ma con funzionalità core effettivamente funzionanti contro un'applicazione rotta. Il concetto chiave è il "sprint contract": prima dell'implementazione, Generatore ed Evaluator negoziano un contratto esplicito che definisce i criteri di successo testabili, colmando il gap tra user story e verifica concreta.

Nella versione ottimizzata (V2), il miglioramento di Opus 4.6 ha permesso di eliminare la decomposizione in sprint, mantenendo solo Planner ed Evaluator. L'insight fondamentale è che ogni componente dell'harness codifica un'assunzione su cosa il modello non sa fare da solo — queste assunzioni vanno stress-testate continuamente perché scadono con ogni nuova versione del modello. Il valore dell'evaluator diventa dipendente dal task: critico per compiti al limite della capacità del modello, overhead non necessario per compiti nel range baseline.

## Codice

Struttura comparativa dei risultati: approccio singolo agente vs harness completo.

```
| Approccio      | Durata   | Costo | Esito                              |
|----------------|----------|-------|------------------------------------|
| Singolo agente | 20 min   | $9    | Funzionalità core rotta            |
| Harness V1     | 6 ore    | $200  | Applicazione multi-feature funz.   |
| Harness V2 DAW | 3h 50min | $124  | DAW browser-based con AI integrata |
```
