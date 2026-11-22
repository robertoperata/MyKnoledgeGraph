---
tags:
  - observability
  - knowledge-graph
  - ontology
  - netflix
  - distributed-systems
feature:
type: article
author: Netflix Tech Blog
source: https://netflixtechblog.com/ontology-driven-observability-building-an-end-to-end-knowledge-graph-at-netflix-scale-421392cf5adf
date: 2026-11-22
---

# Ontology-Driven Observability: Building an End-to-End Knowledge Graph at Netflix Scale

## Sunto

Netflix opera a una scala senza paragoni nell'industria dello streaming: durante l'incontro tra Jake Paul e Mike Tyson nel novembre 2024, il sistema ha gestito circa 65 milioni di stream concorrenti. Nella quotidianità, l'infrastruttura riceve più di 2 miliardi di richieste al giorno e gestisce oltre 38 milioni di eventi di logging in tempo reale ogni secondo. A questa scala, l'osservabilità tradizionale — basata su metriche, log e trace isolati — non è sufficiente per comprendere le dipendenze end-to-end tra i servizi.

L'approccio descritto nell'articolo è l'osservabilità guidata dall'ontologia: costruire un knowledge graph che rappresenta l'intera infrastruttura Netflix come un grafo di entità (servizi, host, container, regioni, cluster) e relazioni tra di esse. L'ontologia definisce i tipi di entità e le relazioni ammesse, fornendo un vocabolario condiviso che permette di correlare automaticamente segnali provenienti da sistemi eterogenei (metriche Prometheus, log, eventi di deployment, topology dei servizi).

Il knowledge graph abilita una serie di capacità operative che sarebbero impossibili con i dati osservazionali tradizionali. Quando si verifica un incidente, il grafo permette di navigare le dipendenze a monte e a valle di un servizio degradato, identificare i componenti condivisi tra i servizi impattati, e correlare anomalie di latenza con eventi di infrastruttura come deploy o scalabilità. Ogni nodo del grafo può aggregare segnali di health provenienti da fonti diverse, normalizzati tramite l'ontologia.

La scala del knowledge graph a Netflix richiede soluzioni di ingegneria specifiche: aggiornamenti incrementali anziché ricostruzione completa, pipeline di ingestione idempotenti per gestire i retry, e query efficienti su grafi con milioni di nodi. L'ontologia funge anche da schema di governance: nuovi tipi di entità e relazioni devono essere proposti e revisionati prima di essere aggiunti al grafo, evitando la proliferazione incontrollata di concetti non standardizzati.

L'architettura ontology-driven rappresenta un'evoluzione significativa rispetto ai sistemi di osservabilità tradizionali: invece di cercare in log e metriche decontestualizzati, gli ingegneri possono formulare query semantiche sull'infrastruttura ("quali servizi dipendono da questo database?" "quali deploy sono avvenuti nell'ultima ora nelle regioni impattate?") e ottenere risposte contestuali. Questo approccio pone le basi anche per l'AIOps: agenti che usano il knowledge graph come contesto per diagnosi automatizzate.

*Nota: il contenuto completo dell'articolo non è stato accessibile al momento della pubblicazione di questo riassunto.*
