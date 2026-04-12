---
title: Bounded Context
type: concept
tags: [thread1-microservices, thread5-ddd]
sources:
  - "[[Microservices Fundamental]]"
  - "[[Domain-Driven Design Aggregates Domain Events and Value Objects]]"
updated: 2026-04-09
related:
  - "[[concepts/ubiquitous-language]]"
  - "[[concepts/aggregate]]"
  - "[[concepts/independent-deployability]]"
  - "[[concepts/information-hiding]]"
---

# Bounded Context

## Definizione

Un Bounded Context è un confine esplicito all'interno del dominio di business in cui un modello specifico è valido e consistente. All'interno del confine vige un [[concepts/ubiquitous-language]] condiviso; oltre il confine, gli stessi termini possono significare cose diverse.

## Come funziona

Un bounded context ha tre responsabilità:
1. **Raggruppamento organizzativo**: corrisponde a una responsabilità del business (e spesso a un team)
2. **Nascondere la complessità**: ciò che è interno al context non è visibile dall'esterno
3. **Lingua condivisa**: all'interno del confine tutti usano lo stesso linguaggio — il ubiquitous language

La struttura del bounded context tende a essere più **stabile** rispetto alla struttura tecnica, perché riflette il business e il business cambia più lentamente del codice.

## Bounded Context e Microservizi

Il bounded context è il criterio principale per trovare le boundary dei microservizi. Vantaggi:
- Allineamento con la struttura organizzativa (Conway's Law)
- Le boundary di dominio sono più stabili delle boundary tecniche
- Facilita la ricombinazione funzionale per nuove esperienze

> [!quote] "Organizations which design systems are constrained to produce designs which are copies of the communication structures of those organizations." — Conway's Law, citata da Sam Newman

Il bounded context non coincide sempre con un singolo microservizio. Un bounded context può essere suddiviso in più microservizi, ma un microservizio non dovrebbe mai attraversare il confine di due bounded context diversi.

## Esempi pratici dalle sorgenti

**Microservices Fundamental:**
- L'aggregate è un concetto interno al bounded context: un insieme di oggetti da gestire come unità transazionale singola.
- La feature team ownership rispecchia il bounded context: un team possiede un pezzo del dominio di business, non una layer tecnica.

**DDD (Vaughn Vernon):**
- Strategic modeling: come identificare i bounded context del sistema
- Tactical modeling: come modellare all'interno di ogni context (aggregates, value objects, domain events)

## Connessioni

- [[concepts/ubiquitous-language]] — lingua condivisa all'interno del bounded context
- [[concepts/aggregate]] — building block principale all'interno di un bounded context
- [[concepts/domain-event]] — il mezzo principale per comunicare tra bounded context
- [[concepts/independent-deployability]] — la boundary del bounded context è dove si mette il confine del microservizio
- [[concepts/information-hiding]] — il bounded context è il confine dello hiding
