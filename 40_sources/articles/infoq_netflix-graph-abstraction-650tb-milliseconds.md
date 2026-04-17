---
tags:
  - architecture
  - distributed-systems
  - database
  - performance
  - streaming
type: article
author: Leela Kumili
source: https://www.infoq.com/news/2026/03/netflix-graph-abstraction/
date: 2026-03-23
---

# Inside Netflix's Graph Abstraction: Handling 650TB of Graph Data in Milliseconds Globally

## Sunto

Netflix ha sviluppato un sistema interno chiamato **Graph Abstraction**, progettato per gestire dati di grafi su larga scala in tempo reale. Il sistema supporta molteplici servizi interni — tra cui grafi sociali per Netflix Gaming e grafi di topologia dei servizi per il monitoraggio operativo — riuscendo a eseguire query su circa **650 TB di dati di grafo in pochi millisecondi** a livello globale.

La scelta architetturale più rilevante riguarda il *trade-off* tra flessibilità e latenza. A differenza dei database a grafo tradizionali che privilegiano traversal arbitrari e flessibili, Graph Abstraction **limita la profondità di traversal** e richiede nodi di partenza definiti. Questo vincolo consente di garantire latenza bassa e consistente anche ad alta scala, dove la flessibilità sarebbe incompatibile con i requisiti di performance.

Il sistema è costruito a strati sopra l'infrastruttura esistente di Netflix: lo stato corrente del grafo è memorizzato tramite l'astrazione *Key-Value*, mentre le variazioni storiche sono registrate tramite l'astrazione *TimeSeries*. L'integrazione con **EVCache** riduce ulteriormente la latenza nelle letture. Gli schemi dei grafi vengono caricati in memoria per la validazione e per ottimizzare il piano di traversal.

Per la **strategia di caching**, il sistema adotta due pattern distinti: *write-aside caching* per prevenire scritture duplicate degli edge, e *read-aside caching* per accelerare l'accesso alle proprietà degli edge. La separazione tra le connessioni (edge) e le loro proprietà è un elemento architetturale chiave che consente di replicare i dati globalmente in modo efficiente.

L'API esposta è un'API gRPC ispirata a **Gremlin**, che consente traversal a step concatenati con risultati filtrati. La replica asincrona tra regioni garantisce *eventual consistency* mantenendo alto throughput. I risultati di performance dichiarati sono notevoli: latenza a singola cifra di millisecondi per traversal a un hop, e meno di 50ms per query a due hop al 90° percentile. Netflix prevede di espandere Graph Abstraction a nuovi verticali: contenuti live, gaming e advertising.

| Use Case | Descrizione |
|----------|-------------|
| Social graph | Relazioni tra utenti per Netflix Gaming |
| Service topology graph | Interazioni tra servizi per incident analysis e root cause investigation |
| TimeSeries abstraction | Tracciamento dello stato storico del grafo |
| Real-time distributed graphs | Cattura delle interazioni tra servizi in tempo reale |

---

## Link esterni

Nessuno

---

## Immagini

Nessuna immagine presente
