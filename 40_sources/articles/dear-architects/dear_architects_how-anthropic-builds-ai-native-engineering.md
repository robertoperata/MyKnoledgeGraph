---
tags:
  - ai-native
  - team-structure
  - engineering-leadership
  - platform-engineering
  - ai-agents
feature:
type: article
author: Gregor Ojstersek
source: https://newsletter.eng-leadership.com/p/how-anthropic-builds-ai-native-engineering
date: 2026-08-23
---

# How Anthropic Builds AI-Native Engineering Teams

## Sunto

Questo articolo, basato su un'intervista con Katelyn Lesse, Head of Platform Engineering di Anthropic, esplora come una delle aziende AI più avanzate struttura internamente i propri team di ingegneria in un ambiente che definisce "AI-native". La prospettiva è particolarmente preziosa perché Anthropic non solo usa l'AI come strumento, ma ne è anche il costruttore, rendendo la loro esperienza un caso studio estremo di integrazione AI nello sviluppo software.

La definizione di "AI-native" di Anthropic è precisa e operativa: l'AI è integrata nel workflow al punto da non sembrare uno strumento aggiuntivo. Non si tratta di usare un assistente AI occasionalmente, ma di costruire processi in cui ogni fase del lavoro di ingegneria — progettazione, implementazione, testing, revisione — contempla l'AI come parte naturale dell'esecuzione. Questa integrazione profonda cambia radicalmente come i team possono operare.

La struttura dei team mantiene il formato tradizionale a "two-pizza" (5-8 ingegneri, un engineering manager, un PM e un designer), ma la distribuzione del lavoro è cambiata. In passato, un tech lead coordinava mentre gli altri implementavano; ora, grazie alla leva dell'AI, quasi ogni ingegnere può assumere responsabilità da tech lead quando necessario. Il risultato è che i team gestiscono 4-5 progetti contemporaneamente invece di 1-2, senza aumentare il numero di persone. Questo moltiplica la capacità produttiva ma richiede una ridefinizione di chi prende le decisioni.

Contrariamente alla tendenza di molte aziende tech a ridurre i PM, Anthropic sta valutando di aumentare il rapporto PM/team. La logica è che quando i bottleneck implementativi scompaiono, il vero vincolo diventa la qualità delle decisioni: capire cosa costruire, perché, e per chi. I PM diventano quindi più critici, non meno. Parallelamente, Anthropic ha eliminato i ruoli QA dedicati, distribuendo la responsabilità del testing su tutti gli ingegneri e investendo in sistemi di AI evaluation per valutare le performance di modelli e prodotti, seguendo la piramide dei test classica con enfasi sui test unitari.

Un elemento distintivo è come Anthropic affronta il codice generato dall'AI: tutto il codice è AI-generated, ma gli ingegneri mantengono responsabilità critiche — progettare l'architettura di sistema, prendere decisioni strutturali, guidare continuamente gli agenti AI con feedback, iterare sull'output fino agli standard richiesti, e revisionare il codice prodotto. La revisione del codice AI è diventata una competenza ingegneristica core. Lesse sottolinea che descrivere outcomes anziché passi procedurali è il fattore critico di successo: "costruisci un dashboard" è meno efficace che descrivere cosa deve fare il dashboard e come si misura il suo successo.
