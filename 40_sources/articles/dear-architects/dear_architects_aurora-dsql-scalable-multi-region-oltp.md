---
tags:
  - aurora-dsql
  - distributed-database
  - multi-region
  - optimistic-concurrency-control
  - serverless
feature:
type: article
author: Marc Brooker, Marc Bowes, Mike Hershey, Zak van der Merwe, James Morle, Matthys Strydom
source: https://arxiv.org/abs/2607.13276
date: 2026-07-26
---

# Aurora DSQL: Scalable, Multi-Region OLTP

## Sunto

Aurora DSQL è un database SQL serverless progettato da AWS per il transaction processing cloud-scale con capacità multi-region active-active. Il paper che ne descrive l'architettura (arXiv 2607.13276) è la prima risorsa pubblica che fornisce una visione end-to-end completa del sistema — dai query processor alle transazioni, dalla replicazione al control plane. L'obiettivo dichiarato era costruire "un database di prima scelta: semplice e facile da adottare a bassa scala, che cresce con l'applicazione senza aggiungere complessità".

L'architettura è **disaggregata**: compute, storage e transaction coordination sono servizi indipendenti con responsabilità ben definite. I query processor girano in Firecracker MicroVM ed eseguono SQL compatibile con PostgreSQL senza mantenere stato locale — una scelta che semplifica enormemente failover e scaling orizzontale. Questa lezione è stata appresa da Aurora e DynamoDB: evitare cache grandi semplifica i failover, la scalabilità delle letture non crea hot key problems, e le operazioni costose (come le scan) possono essere spinte allo storage layer.

Il meccanismo di concorrenza è **OCC (Optimistic Concurrency Control)**: invece di acquisire lock prima di ogni operazione, il sistema esegue le transazioni ottimisticamente e verifica i conflitti solo al momento del commit. Il coordinamento avviene attraverso "adjudicator" distribuiti e il sistema Journal. Questo design elimina il blocking client-to-client che è la principale causa di tail latency nei database con locking pessimistico — e quindi elimina anche i failure metastabili da lock contention che hanno causato molte delle outage operative più gravi nei sistemi tradizionali.

La coerenza forte a scala multi-region è resa possibile da una combinazione di progressi infrastrutturali: distribuzione precisa del tempo (time distribution), networking moderno nei datacenter, power management e protocolli distribuiti. Questi avanzamenti hanno spostato i trade-off rispetto al paradigma degli anni 2000, dove l'eventual consistency era spesso l'unica scelta pratica per sistemi distribuiti ad alta disponibilità. Aurora DSQL dimostra che strong consistency, ACID transactions e disponibilità continua attraverso fallimenti di availability zone o di intera region sono ora raggiungibili insieme a scala cloud.

Un aspetto particolarmente rilevante per i sistemi AI è la capacità di Aurora DSQL di servire come database di "prima scelta" per i sistemi agentico: gli agenti AI eccellono nel costruire e operare sistemi, ma faticano con la complessità operativa a lungo termine di database tradizionali (sharding, failover, repliche). Un database che scala elasticamente da zero a milioni di transazioni al secondo senza richiedere gestione operativa è nativamente compatibile con il modello mentale "build and forget" dei sistemi agentico moderni.
