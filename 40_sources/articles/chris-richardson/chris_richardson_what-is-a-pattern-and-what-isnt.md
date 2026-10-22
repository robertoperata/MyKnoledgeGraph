---
tags:
  - software-patterns
  - pattern-language
  - software-architecture
  - microservices
  - design-patterns
feature:
type: article
author: Chris Richardson
source: https://microservices.io/post/architecture/2026/10/12/what-is-a-pattern.html
date: 2026-10-22
---

# What is a pattern, and what isn't?

## Sunto

Il termine "pattern" ha subito una forte **diffusione semantica** nell'ingegneria del software: molte conoscenze riutilizzabili vengono etichettate come pattern anche quando non rispettano la definizione originale. Chris Richardson, autore di *Microservices Patterns*, richiama alla precisione terminologica prendendo spunto dal Domain-Driven Design e dalla tradizione alexandriana.

Un pattern è definito come una **soluzione ricorrente a un problema comune in un contesto specifico**. I suoi elementi essenziali sono: *context* (la situazione in cui sorge il problema), *problem* (espresso come domanda difficile con *forces* — le considerazioni contrastanti da bilanciare), *solution* (generativa: spiega come costruire la soluzione, non solo come appare), *consequences* (benefici, svantaggi e *issues* — sotto-problemi che emergono applicando il pattern) e *related patterns* (alternative, successori, predecessori, generalizzazioni, specializzazioni). Affinché la soluzione sia un pattern, deve essere stata applicata in contesti reali **più di una volta**.

Richardson distingue chiaramente cosa **non è** un pattern: una *tecnica* è una soluzione priva di contesto e forze; un *design* è una soluzione applicata una volta sola; una *pratica* è un buon modo di lavorare senza un problema specifico; un *principio* è un enunciato di buona soluzione senza problema; una *regola* è un'istruzione il cui problema si può solo inferire. Il libro GoF è, nel senso stretto, una *pattern collection* (raccolta), non un *pattern language*, perché le relazioni tra i pattern sono deboli.

Un **pattern language** si distingue da una collection per la ricchezza delle relazioni: alternative, predecessori, successori, generalizzazioni e specializzazioni. È *generativo*: seguire le relazioni produce un design completo. L'esempio di Richardson è il grafo di microservices.io: il pattern *Database per Service* è predecessore di *Saga*, che ha come alternative *Transactional Outbox* ed *Event Sourcing* come successori. Questa rete consente di navigare lo spazio delle decisioni architetturali.

L'articolo confronta cinque forme per scrivere pattern: la forma **Alexandriana** (prosa continua con pochi titoli, asterischi per la confidenza), la forma **GoF** (12 sezioni su classi e oggetti), la forma **POSA** (13 sezioni con esempio iniziale e risolto), la forma **Narrative** (*Fearless Change*, storie di persone con linguaggio colloquiale) e la forma **Microservices Patterns** (8 sezioni con Issues distinte dai Drawbacks). Richardson conclude con checklist pratiche per autori e lettori: prima di chiamare qualcosa "pattern", verificare che siano presenti problema, forze, conseguenze e che la soluzione sia stata applicata più volte in contesti reali.

## Immagini

![Elementi di un pattern: context, problem, forces, solution, consequences, related patterns](https://microservices.io/i/what-is-a-pattern/pattern-elements.svg)

## Codice

Non sono presenti snippet di codice significativi nell'articolo. Gli esempi del pattern Saga vengono illustrati in prosa attraverso le cinque forme di scrittura dei pattern, con riferimenti ai nomi delle classi (`CreateOrderSaga`, `Customer Service`) ma senza blocchi di codice eseguibile.
