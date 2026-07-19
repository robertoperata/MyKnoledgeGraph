---
tags:
  - modularità
  - architettura-software
  - domain-driven-design
  - team-topologies
  - debito-tecnico
feature:
type: article
author: Andrew Harmel-Law, Richard Gall
source: https://www.thoughtworks.com/insights/blog/architecture/modularity-mortality-importance-skin-game
date: 2026-07-19
---

# Modularity & Mortality: The Importance of Skin in the Game

## Sunto

L'articolo di Andrew Harmel-Law e Richard Gall (Thoughtworks, luglio 2026) affronta un paradosso fondamentale del software engineering: nonostante decenni di discussione su astrazioni pulite, microservizi e Domain-Driven Design, i sistemi software tendono inevitabilmente a degenerare nel "Big Ball of Mud". La tesi centrale è che il problema non è la mancanza di conoscenza tecnica, ma l'assenza di accountability: gli ingegneri non sentono le conseguenze del fallimento nel mantenere la modularità.

Gli autori osservano che gli ecosistemi di successo come WordPress, Drupal, Linux e Android condividono un pattern comune: architetture a plugin con pressione evolutiva incorporata. I componenti che non si adattano o perdono utilità "muoiono" per disuso. I codebase aziendali soffrono invece di "immortalità assoluta": niente muore mai, il debito tecnico si accumula come strati sedimentari. Il punto di svolta è che lo sviluppo esterno di plugin impone confini rigorosi perché i developer non conoscono i loro consumer; i codebase interni mancano di questo meccanismo coercitivo.

Un'altra trappola analizzata è il mal uso del Domain-Driven Design. I professionisti tipicamente sbagliano in due modi: trattarlo come guida tattica (dibattendo se qualcosa è Entity o Value Object) oppure discutere i "bounded context" senza tradurre le intuizioni in codice reale. Il breakthrough critico è che la modularità fallisce quando i tecnologi tracciano confini attorno a concetti che non comprendono abbastanza. Un esempio illuminante: le compagnie aeree low-cost trattano i posti come value object volatili, mentre i vettori luxury li trattano come entity stateful — questa differenza di modello di business richiede architetture radicalmente diverse per lo stesso concetto di "posto".

Un contributo originale dell'articolo è l'identificazione dei "Swamp Guides" (o "Dungeon Masters"): developer che prosperano nei codebase caotici. Conoscono dove si nascondono i problemi e come bypassare l'encapsulation per velocità. Le organizzazioni li premiano involontariamente, celebrando la loro efficienza nel breve termine ma incentivando la degradazione architetturale nel lungo. La ricerca Microsoft dimostra che l'alta rotazione dei team correla con i bug più dell'ownership del codice: la perdita di modelli mentali condivisi porta a vittorie tattiche di breve termine che erodono i confini architetturali.

Con l'avvento dei LLM e degli agenti AI che scrivono codice, i confini modulari diventano "da preferenza estetica a prerequisito cognitivo". Questi sistemi trovano il percorso matematicamente più efficiente indipendentemente dall'encapsulation, potenzialmente obliterando l'integrità architetturale. La raccomandazione pratica degli autori è progettare per la cancellabilità anziché per la riusabilità: costruire componenti autonomi con contratti puliti e testarne la pulizia verificando se il componente può essere strappato via e riscritto rapidamente.

## Immagini

![Modularity and Skin in the Game - Thoughtworks](https://www.thoughtworks.com/content/dam/thoughtworks/images/photography/meta-image/tw_meta_image.png)
