---
tags:
  - hibernate
  - jpa
  - hierarchical-data
  - sql
  - java-persistence
feature:
type: article
author: Vlad Mihalcea
source: https://www.youtube.com/watch?v=nrNY0h4jHVo
date: 2026-05-15
---

# The Best Way to Fetch Hierarchical Data with Java

## Sunto

In questa presentazione al Virtual JUG, Vlad Mihalcea e Laurenţiu Spilcă esplorano le strategie ottimali per recuperare strutture dati gerarchiche quando si utilizzano Java e sistemi di database relazionali. La trattazione analizza comparativamente diversi framework di accesso ai dati, fornendo benchmark di performance che guidano le scelte architetturali in contesti reali.

Le strutture dati gerarchiche — come alberi di categorie, organizzazioni aziendali, thread di commenti o strutture di menu ricorsivi — rappresentano una delle sfide più comuni nella persistenza relazionale. I database relazionali tradizionali non modellano nativamente la ricorsione, rendendo necessario l'uso di pattern specifici come le Adjacency List, le Nested Sets, le Path Enumeration o le Closure Tables, ciascuna con diversi trade-off tra leggibilità, performance e facilità di aggiornamento.

La presentazione copre l'approccio al fetching gerarchico tramite **JDBC puro**, **Spring Data JPA**, **Spring Data JDBC**, **jOOQ** e **Blaze Persistence**. Ogni framework viene esaminato in termini di espressività delle query, supporto alle CTE (Common Table Expressions) ricorsive, e overhead introdotto dall'astrazione ORM. Le CTE ricorsive, supportate da tutti i principali RDBMS (PostgreSQL, MySQL 8+, Oracle, SQL Server), permettono di navigare strutture gerarchiche arbitrariamente profonde con una singola query SQL.

I benchmark presentati mettono in luce come l'approccio "corretto" dipenda fortemente dal caso d'uso: jOOQ eccelle nella costruzione di query SQL complesse con type-safety a compile time, Blaze Persistence offre un layer di astrazione JPQL potenziato per query CTE con Hibernate, mentre Spring Data JPA richiede spesso query native per scenari ricorsivi avanzati. JDBC rimane la scelta con il massimo controllo e la minima latenza, al costo di maggiore boilerplate.

La sessione include dimostrazioni di codice pratiche e risposte alle domande del pubblico, offrendo una visione pragmatica delle problematiche di N+1 query, eager/lazy loading e batch fetching nel contesto di dati ad albero, aspetti critici per le performance di applicazioni enterprise che gestiscono cataloghi, tassonomie o strutture organizzative complesse.
