---
tags:
  - database
  - single-runtime-architecture
  - performance
  - persistence
  - benchmarking
feature:
type: article
author: Renato Losio
source: https://www.infoq.com/news/2026/08/harper-vercel-benchmark/
date: 2026-08-29
---

# Harper Argues against the Multi-System Stack and Releases 5.2

## Sunto

Harper è una piattaforma database che propone un approccio radicalmente diverso dallo stack applicativo tradizionale: invece di separare il database, la cache e il job runner come servizi distinti connessi via rete, Harper colloca codice applicativo e dati nello stesso processo (co-location). L'architettura "single-runtime" elimina i salti di rete tra i livelli, trattando le letture personalizzate come chiamate di funzione su tabelle in-memory anziché come richieste a un servizio esterno.

Il confronto di riferimento viene effettuato contro uno stack Vercel composto da Vercel Functions, Neon Postgres, Upstash Redis e Ably. Harper ha eseguito 474 test di carico in otto scenari, due regioni US e tre run, evidenziando che per i workload di lettura personalizzata (singoli record, valori live iniettati, streaming server-side, read fan-out) la performance supera lo stack Vercel fino a 14×. La latenza di accesso ai dati in-process si attesta intorno a 0,4ms contro circa 3ms per un hop di rete.

Harper 5.2, rilasciata recentemente, introduce un record cache che accelera le letture ripetute di 5-8×, percorsi di commit isolati che impediscono alle scritture pesanti di bloccare operazioni indipendenti, e migliorie al p99: le chiamate filesystem sono scese da 223,7ms a 2,6ms risolvendo la contention sul thread pool libuv di Node.js.

I vantaggi dell'architettura si rovesciano però per workload di distribuzione CDN o broadcast real-time ad alto fan-out illimitato, dove lo stack Vercel risulta superiore. Un caveat critico del benchmark: il dataset era completamente in-memory; su working set che eccedono la RAM disponibile i vantaggi potrebbero ridursi sensibilmente. Il benchmark inoltre non include la versione 5.2 e non è stato rieseguito con le nuove ottimizzazioni.

L'approccio si posiziona in contrapposizione con tendenze come Databricks Lakebase, che separa compute e storage. L'architetto Aleks Haugom riassume la filosofia: "Una lettura personalizzata non è una richiesta di rete a un altro servizio; è una chiamata di funzione su una tabella in-memory". La replica globale verso nodi regionali garantisce consistenza e serving locale.

## Immagini

![Benchmark results Harper vs Vercel stack across 8 scenarios](https://imgopt.infoq.com/articles/harper-vercel-benchmark/en/resources/harper-bench.jpg)
