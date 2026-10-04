---
tags:
  - domain-driven-design
  - software-architecture
  - bounded-context
  - microservices
  - modular-monolith
feature:
type: article
author: Alireza Rahmani Khalili
source: https://nidly.substack.com/p/where-does-software-architect-fit
date: 2026-10-04
---

# Where Does Software Architect Fit in Domain-Driven Design?

## Sunto

L'articolo di Alireza Rahmani Khalili affronta un malinteso diffuso nei team di sviluppo: la confusione tra i confini semantici identificati dal Domain-Driven Design e i confini fisici di deployment dell'architettura software. L'autore argomenta che DDD e architettura software servono scopi distinti e che conflare i due approcci porta a decisioni di design costose e spesso non giustificate.

Il **DDD** offre strumenti per identificare confini semantici significativi attraverso il linguaggio condiviso (Ubiquitous Language), invarianti coerenti e ownership chiara. Un bounded context emerge dall'analisi del dominio e rappresenta un confine di significato: parole diverse per lo stesso concetto in parti diverse del business, o lo stesso termine con significati diversi. Tuttavia, l'autore sottolinea che "Domain-Driven Design gives you evidence for boundaries. It does not hand you a deployment topology." Scoprire un bounded context non giustifica automaticamente la creazione di un servizio separato.

I confini fisici — network boundary, deployment separato, database indipendente — richiedono giustificazioni operative concrete: bisogni di scaling indipendente, domini di failure separati, cicli di deployment distinti, requisiti di sicurezza differenti, modelli di consistenza diversi. Senza almeno uno di questi driver operativi, "a network boundary represents nothing except an architectural habit": un'abitudine architettonica senza valore reale, ma con costi concreti in complessità distribuita.

L'articolo identifica una serie di equivalenze errate comuni nei team: bounded context ≠ microservizio, aggregate ≠ servizio, domain event ≠ messaggio su broker, context map ≠ topologia di rete, business process ≠ workflow distribuito. Ciascuna di queste confusioni porta a over-engineering sistematico: teams che decompongono il sistema in decine di microservizi perché "ogni bounded context deve essere un servizio", aumentando la complessità operativa senza ottenere benefici reali.

La raccomandazione pratica è di iniziare con un **monolite modulare** che preserva i confini logici identificati dal DDD, ma li implementa come moduli del codice piuttosto che come servizi separati. Questo approccio mantiene la flessibilità: la separazione fisica può avvenire in seguito quando l'evidenza operativa la giustifica. La tesi centrale si condensa in una frase emblematica: "A bounded context is a hypothesis about meaning; a service boundary is a bet about cost."
