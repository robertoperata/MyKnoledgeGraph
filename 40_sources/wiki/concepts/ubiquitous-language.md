---
title: Ubiquitous Language
type: concept
tags: [thread1-microservices, thread5-ddd]
sources:
  - "[[Microservices Fundamental]]"
  - "[[Domain-Driven Design Aggregates Domain Events and Value Objects]]"
  - "[[como_preparation-collaborative-modeling-workshops]]"
updated: 2026-04-09
related:
  - "[[concepts/bounded-context]]"
  - "[[concepts/aggregate]]"
---

# Ubiquitous Language

## Definizione

Una lingua condivisa tra tecnici e persone di business, che si riflette direttamente nel codice. Verbi, sostantivi e concetti del dominio vengono usati in modo coerente sia nelle conversazioni che nel modello software.

## Come funziona

L'ubiquitous language non è un dizionario: è un linguaggio **vivente** che evolve con la comprensione del dominio. Si costruisce attraverso:
- Conversazioni continue tra sviluppatori e domain expert
- Tecniche come **event storming** per scoprire la lingua del dominio
- Riflessione del linguaggio nel codice: nomi di classi, metodi, variabili

> [!important] L'ubiquitous language è valido solo all'interno di un [[concepts/bounded-context]]. Il termine "Customer" può significare cose diverse nel bounded context degli ordini e in quello della fatturazione.

## Connessione con i Workshop di Collaborative Modeling

Il talk "Preparation for Collaborative Modeling Workshops" (Beja, CoMo community) descrive come costruire la comprensione condivisa che porta all'ubiquitous language:
- **Context Clarity**: chiarire il *perché* del workshop prima di tutto
- **People & Politics**: ogni partecipante porta una storia — il senior architect che difende lo status quo, il junior che vuole cambiare tutto. La lingua condivisa emerge dalla negoziazione di queste storie.
- **EventStorming**: tecnica citata da Sam Newman come strumento per scoprire l'ubiquitous language del dominio

> [!quote] "Se le persone non sanno perché sono in quella stanza, nessuna quantità di post-it le salverà." — Beja, CoMo

## Esempi pratici dalle sorgenti

**Microservices Fundamental (Sam Newman):**
- L'ubiquitous language è il collegamento tra il domain model e il business. Senza di esso il codice riflette strutture tecniche, non il business.

**Workshop Collaborative Modeling:**
- Il CoMo Prep Canvas è uno strumento per preparare le condizioni (cognitive, politiche, di design) necessarie perché l'ubiquitous language possa emergere in un workshop.

## Connessioni

- [[concepts/bounded-context]] — l'ubiquitous language è valido dentro un bounded context
- [[concepts/aggregate]] — gli aggregates sono modellati con i concetti dell'ubiquitous language
- [[concepts/domain-event]] — gli eventi di dominio usano il linguaggio del dominio ("OrderPlaced", "CustomerActivated")
