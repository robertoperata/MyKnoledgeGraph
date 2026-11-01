---
tags:
  - ddd
  - bdd
  - domain-driven-design
  - event-sourcing
  - software-architecture
feature:
type: article
author: Anita Kvamme
source: https://www.youtube.com/watch?v=Qk9UCQB5x8w
date: 2026-11-01
---

# DDD & BDD Then and Now: Unpacking Similarities and Differences — DDD Europe 2025

## Sunto

In questo talk presentato a DDD Europe 2025, Anita Kvamme esplora le similarità e le differenze tra Domain-Driven Design (DDD) e Behavior-Driven Development (BDD) attraverso il racconto di due progetti reali con contesti molto diversi: il primo incentrato sul breaking up di un monolite, il secondo su un progetto greenfield che ha adottato event sourcing fin dall'inizio. La prospettiva "then and now" suggerisce un'analisi di come queste metodologie si sono evolute e come la loro applicazione pratica cambia a seconda del contesto di progetto.

DDD e BDD condividono un focus fondamentale sul linguaggio condiviso tra business e team tecnico. Entrambe le metodologie enfatizzano la collaborazione tra domain expert e sviluppatori, l'uso di un linguaggio ubiquitous che riduce la distanza semantica tra requisiti e implementazione, e la modellazione del comportamento del sistema come elemento centrale del design. Tuttavia, partono da angolazioni diverse: DDD si focalizza sulla strutturazione del dominio (aggregati, bounded context, domain event), mentre BDD si concentra sulla specifica del comportamento atteso attraverso scenari eseguibili (Given-When-Then).

Nel contesto del progetto di breaking up del monolite, le due metodologie hanno probabilmente mostrato le loro differenze in modo più netto: DDD ha guidato l'identificazione dei bounded context e la strategia di decomposizione, mentre BDD ha aiutato a specificare e verificare il comportamento atteso di ogni pezzo estratto. Nel progetto greenfield con event sourcing, la combinazione delle due metodologie diventa particolarmente interessante: event sourcing nasce direttamente da concetti DDD (domain event come first-class citizen), mentre BDD fornisce il framework per specificare le sequenze di eventi e le trasformazioni di stato in modo verificabile.

Le differenze emerse nelle due esperienze illuminano anche i limiti di ciascun approccio quando applicato da solo: DDD senza BDD può portare a una modellazione del dominio ricca ma difficile da verificare in modo sistematico; BDD senza DDD può produrre scenari ben specificati ma ancorati a un modello di dominio superficiale. La combinazione dei due approcci — usando BDD per articolare i comportamenti che emergono da un modello DDD solido — rappresenta una pratica sempre più adottata nei team che lavorano su domini complessi.

*Nota: il video YouTube non era accessibile al momento del fetch; il sunto è basato sul titolo, sulla descrizione dell'email e sul contesto del talk.*
