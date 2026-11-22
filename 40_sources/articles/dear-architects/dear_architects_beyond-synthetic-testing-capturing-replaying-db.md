---
tags:
  - database
  - mysql
  - testing
  - proxysql
  - performance-testing
feature:
type: article
author: Zuofei Wang, Erluo Li
source: https://airbnb.tech/infrastructure/beyond-synthetic-testing-capturing-and-replaying-real-database-workloads-at-airbnb/
date: 2026-11-22
---

# Beyond synthetic testing: Capturing and replaying real database workloads at Airbnb

## Sunto

Airbnb gestisce centinaia di cluster MySQL-compatibili che supportano migliaia di use case a milioni di query al secondo. L'articolo descrive il sistema di cattura e replay del traffico database che il team ha costruito per affrontare tre esigenze critiche: load testing con traffico reale, verifica della compatibilità delle query durante upgrade e migrazioni, e riproduzione di problemi di performance. L'approccio precedente registrava le query lato client in ogni linguaggio (Java, Python, Ruby), con costi di manutenzione elevati e mancanza di contesto transazionale.

Il nuovo design cattura il traffico al livello ProxySQL — il proxy che si interpone tra le applicazioni e i database MySQL — eliminando la necessità di modifiche alle applicazioni. Il sistema è composto da tre componenti sviluppati internamente. Il **Log Mover** è un sidecar affiancato a ogni pod ProxySQL che invia i log binari locali all'object storage cloud. Il **Log Processor** è un job offline che decodifica i log binari, li suddivide per cluster, riassembla le transazioni, le raggruppa in finestre di cinque minuti e riscrive le INSERT. Il **Log Replayer** è un control plane composto da un API server, uno scheduler e worker scalati orizzontalmente che eseguono il replay contro i database target.

Il Replayer opera in due modalità. La modalità "replay only" misura carico, errori e latenza a una velocità configurabile (es. 2x il traffico reale), utile per il capacity planning. La modalità "replay and compare" esegue le stesse query su due database e compara i risultati, ideale per validare migrazioni o upgrade. Per preservare la fedeltà temporale del traffico reale, lo scheduler assegna a tutti i task in un batch un "expected start time" condiviso, così i worker che elaborano le finestre da cinque minuti partono insieme e mantengono la concorrenza originale.

Un problema tecnico rilevante è la gestione degli auto-increment: in MySQL, INSERT senza colonna ID genera un `last_insert_id` che potrebbe differire tra il database originale e il target, causando falsi fallimenti nelle query successive che usano quell'ID. Il Log Processor riscrive le INSERT includendo il `last_insert_id` catturato, fissando i valori invece di lasciarli generare dal target. Il trade-off è che questa scelta non testa la generazione nativa degli auto-increment del target.

Il sistema ha rivelato problemi critici durante la migrazione da MySQL 5.7 a 8.0: la rimozione della query cache ha esposto query duplicate che la cache nascondeva; un pattern di query ha visto la latenza salire da 0,03s a 2,6s, leggendo 273 MB per join invece di 5 MB a causa di un cambiamento nel modo in cui MySQL 8.0.20+ legge le righe durante l'ordinamento; query senza ORDER BY deterministico restituivano righe diverse sotto LIMIT. Per il capacity planning, aumentare dell'80% il traffico di write su un cluster di grandi dimensioni ha elevato la latenza media di commit da circa 6ms a 34ms, identificando il tetto prima che il traffico reale lo raggiungesse.

## Immagini

![Legacy Query Capture and Replay System - architettura con Java, Python, Ruby e Kafka](https://miro.medium.com/v2/resize:fit:1400/1*NwEE8F6fNi-4SIJaINLSKQ.png)

![Log Mover come sidecar di ProxySQL - client apps inviano query a ProxySQL pods con Log Mover](https://miro.medium.com/v2/resize:fit:1400/1*cAuybMif5--5pgLOjF5Y1w.png)

![Log Processor come job offline - dal Log Mover al query log storage con elaborazione per cluster e finestre temporali](https://miro.medium.com/v2/resize:fit:1400/1*b_sYU3dInfj2SAnpBsyP3w.png)

![Log Replayer Architecture - API server, task scheduler, worker e target databases](https://miro.medium.com/v2/resize:fit:1400/1*CCj9PFdtI_vx7e2J04Vbwg.png)

## Codice

Riscrittura delle INSERT per fissare il valore auto-increment catturato durante il replay:

```sql
-- Query originale (genera last_insert_id=1 sul database sorgente)
INSERT INTO users (name) VALUES ('bob');

-- Query riscritta dal Log Processor (include l'ID catturato per evitare falsi fallimenti)
INSERT INTO users (id, name) VALUES (1, 'bob');
```
