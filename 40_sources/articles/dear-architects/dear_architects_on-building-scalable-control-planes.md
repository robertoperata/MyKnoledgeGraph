---
tags:
  - control-plane
  - distributed-systems
  - database-scaling
  - aws
  - aurora-dsql
  - sharding
feature:
type: article
author: Zak van der Merwe
source: https://www.allthingsdistributed.com/2026/08/on-building-scalable-control-planes.html
date: 2026-08-09
---

# On Building Scalable Control Planes

## Sunto

Zak van der Merwe, dopo 14 anni ad AWS a costruire control plane prima per EC2 e poi per Aurora DSQL, condivide le lezioni apprese sull'architettura di sistemi che governano infrastrutture su scala planetaria. L'articolo funge da companion piece alla documentazione di DSQL e dimostra come le scelte architetturali del control plane abbiano plasmato un database progettato specificamente per risolvere i problemi che EC2 ha incontrato nel tempo.

Un control plane implementa il pattern del termostato: misura continuamente lo stato attuale, lo confronta con lo stato desiderato e spinge il sistema verso l'allineamento. Il principio fondamentale è la **static stability**: "qualunque cosa accada al control plane, le VM già in esecuzione devono continuare a funzionare". Questa distinzione tra outage che impediscono il lancio di nuove risorse e outage che bloccano tutti i servizi si rivela cruciale per le decisioni di scaling.

Il control plane di EC2 è cresciuto su un database relazionale MySQL al suo centro. Le sfide di scaling affrontate sono state tre: failure del primary server (risolto con hot standby e replicazione continua), read scaling (risolto con read replica, introducendo però eventual consistency che "mette uno scomodo carico cognitivo sui clienti"), e write scaling (risolto con sharding geografico tramite Availability Zone e poi cell interne, un percorso che ha richiesto anni di ingegneria su decine di servizi). Il problema dello sharding è brutale: "i lookup per chiave primaria sono semplici, ma qualsiasi altra cosa, come i join su dati che non si allineano con i confini di sharding, diventa molto più complicata."

Aurora DSQL, lanciato nel 2025, nasce dall'esperienza accumulata su EC2 con l'obiettivo di eliminare il concetto di "named database" e il relativo management manuale. Le innovazioni chiave: Firecracker micro-VM per connessione (i fallimenti isolano solo la singola connessione), read replica automatiche con strong consistency, e partitioning automatico del workload che elimina lo sharding manuale abilitando join multi-tabella su qualsiasi scala. La scelta più audace è che il control plane di DSQL gira su DSQL stesso: questo garantisce che zone outage non cascadino sul database di control plane, e crea un feedback loop diretto tra crescita di DSQL e capacità del sistema di auto-scalarsi.

L'articolo conclude con una riflessione sul futuro: mentre il coding agentico riduce i costi di sviluppo, il bottleneck si sposta verso il giudizio — determinare cosa costruire, garantire che le consegne siano sicure, anticipare i bisogni dei clienti. I control plane automatizzano la manutenzione dell'infrastruttura, liberando tempo per questo lavoro ad alto valore. L'auspicio di van der Merwe è che DSQL dia alle prossime generazioni di builder quella fondazione, restituendo loro tempo per "guardare oltre l'angolo" per i loro clienti.

