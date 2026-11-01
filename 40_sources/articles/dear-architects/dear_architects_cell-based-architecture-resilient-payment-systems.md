---
tags:
  - cell-based-architecture
  - resilience
  - payment-systems
  - distributed-systems
  - microservices
feature:
type: article
author: American Express
source: https://americanexpress.io/cell-based-architecture-for-resilient-payment-systems/
date: 2026-11-01
---

# Cell-Based Architecture for Resilient Payment Systems

## Sunto

American Express ha modernizzato il proprio core payments ecosystem a partire dal 2018, adottando un'architettura basata su celle (cell-based architecture) per garantire alta disponibilità, bassa latenza e performance prevedibile su un sistema globale che processa milioni di transazioni di pagamento al giorno. La resilienza era un requisito di design primario fin dall'inizio, e i principi adottati dal team coincidono con quelli della cell-based architecture, pur essendo stati sviluppati indipendentemente prima che il termine diventasse comune nel settore.

Una **cella** è un'unità di deployment indipendente con i propri microservizi, database e componenti di supporto, in grado di processare pagamenti autonomamente. Le celle sono definite da confini di failure, non da confini infrastrutturali: una cella non si estende mai su più regioni. Il principale vantaggio è un **blast radius ridotto**: un guasto in una cella non impatta le altre. Il trade-off è una maggiore complessità operativa e di design, che il team considera giustificato per un sistema di pagamento mission-critical.

La gestione dei dati è il cuore del design. I dati statici e semi-statici (es. tassi di cambio, codici categorie merchant) vengono replicati preventivamente in ogni cella, eliminando le lookup sincrone durante il processing. Per i dati dinamici che cambiano per ogni transazione, il **Global Transaction Router** instrada le transazioni verso la cella dove risiede lo stato autoritativo. I microservizi comunicano esclusivamente con database e pod locali alla propria cella: nessuna comunicazione cross-cella è permessa a livello di servizi. Anche i dati di osservabilità vengono prima pubblicati in componenti interni alla cella e aggregati asincronamente, in modo che un guasto riduca la visibilità solo sulla cella colpita.

La gestione dei failure è deterministica: quando una cella fallisce, le transazioni in volo e quelle nuove vengono rerouted verso una cella sana e riprocessate dall'inizio usando i dati originali della transazione. Le transazioni parzialmente processate non vengono riprese a metà: il reprocessing ricomincia dall'inizio. Questo approccio è reso possibile dal "point of no return": nelle autorizzazioni con carta, questo punto arriva tardi nel flusso di processing, permettendo di abbortire e riprocessare la maggior parte dei casi. Gli identificatori di transazione rimangono costanti attraverso retry e reroute, consentendo ai sistemi downstream di sopprimere i duplicati. Il traffico può essere spostato tra celle per percentuale, supportando drenaggio graduale per manutenzione o validazione controllata di una cella in recovery.

Il principio guida per i componenti vicini all'edge è quello di avere il minor numero possibile di dipendenze esterne, e quelle presenti devono rimanere fuori dal critical path. Il logging usa un logger asincrono con politica di truncation del buffer: i log vengono scartati piuttosto che bloccare le transazioni. La configurazione è tenuta in memoria e aggiornata asincronamente, in modo che il sistema continui a girare sull'ultima configurazione nota durante un'interruzione. La resilienza emerge da confini di failure definiti e forzati tramite il design architetturale, non da monitoring e retry da soli. Locality, routing deterministico, processing idempotente e boundary enforcement rigoroso mantengono il rischio sotto controllo anche con la crescita e il cambiamento del sistema.
