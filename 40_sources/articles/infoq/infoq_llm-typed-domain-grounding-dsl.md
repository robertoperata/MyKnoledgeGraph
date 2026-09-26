---
tags:
  - llm
  - domain-specific-languages
  - type-systems
  - ai-engineering
  - kotlin
feature:
type: article
author: Irakli Betchvaia
source: https://r.mail.infoq.com/mk/cl/f/sh/7nVU1aA2ng1uQNsiyVIKmo8lJzvlvZZ/N_uDTgxcgGI5
date: 2026-09-26
---

# Your Next DSL Author Is a Language Model

## Sunto

Irakli Betchvaia analizza una limitazione fondamentale dei modelli linguistici nel lavoro con linguaggi domain-specific (DSL): la loro sintassi è raramente rappresentata nei dati di training, portando i modelli a produrre sintassi allucinata o strutture non valide. La soluzione proposta, denominata *Typed Domain Grounding*, aggira questo problema incorporando i DSL in linguaggi host ampiamente noti — come Kotlin o TypeScript — dove il type system del linguaggio e il feedback del compilatore possono rilevare e guidare la correzione degli errori.

L'idea centrale è sfruttare ciò che i modelli già conoscono bene. Kotlin dispone di un DSL builder idiomatico basato su lambda con receiver che consente di costruire API fluenti con supporto completo di type safety. Quando un LLM genera codice Kotlin che usa un DSL built this way, il compilatore può segnalare immediatamente gli errori di tipo, e il modello può usare questi messaggi di errore come feedback per correggere iterativamente la propria output. Questo crea un loop di refinement controllato dove le allucinazioni vengono catturate meccanicamente prima di raggiungere l'esecuzione.

Il pattern *Typed Domain Grounding* si distingue dall'approccio alternativo di definire DSL completamente custom (grammatiche ANTLR, parser dedicati): invece di insegnare al modello una nuova sintassi ignota, si riutilizza la sua competenza su linguaggi mainstream e si delegano le garanzie di correttezza ai meccanismi del linguaggio host. I compilatori e i language server moderni (LSP) possono essere integrati direttamente nel loop di inferenza dell'agente per fornire feedback strutturato sugli errori.

Betchvaia argomenta che questo approccio è particolarmente rilevante per il tooling enterprise dove i DSL proliferano: pipeline di trasformazione dati, configurazioni infrastrutturali, query specializzate. Invece di richiedere ai modelli di imparare decine di mini-linguaggi, si adotta un unico linguaggio host come "lingua franca" della generazione automatica, con il sistema di tipi come barriera di qualità.
