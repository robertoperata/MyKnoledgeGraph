---
tags:
  - data-platform
  - lakehouse
  - apache-trino
  - apache-iceberg
  - ai-analytics
  - data-governance
feature:
type: article
author: Leela Kumili
source: https://www.infoq.com/news/2026/07/cloudflare-unified-data-platform/
date: 2026-08-01
---

# Cloudflare Details Unified Data Platform Where Billing Workloads Account for 53% of Queries

## Sunto

Cloudflare ha presentato Town Lake, una piattaforma dati unificata interna progettata per consolidare l'accesso a dati operativi, di fatturazione, di sicurezza e di business precedentemente dispersi in sistemi frammentati. La piattaforma elabora oltre un miliardo di eventi al secondo attraverso 330+ città in 120 paesi. L'obiettivo primario era eliminare i silos informativi e permettere a dipendenti di diverse funzioni aziendali di accedere ai dati attraverso un'unica interfaccia coerente.

L'architettura adottata è di tipo lakehouse: Apache Trino funge da motore di query SQL, Apache Iceberg come formato di storage tabellare su Cloudflare R2, e DataHub per la gestione dei metadati. La caratteristica distintiva è la capacità di eseguire query SQL singole che attraversano tabelle Postgres, ClickHouse e Iceberg senza richiedere alcun movimento fisico dei dati. Le sorgenti dati includono Postgres, ClickHouse, stream Kafka e dataset BigQuery.

La governance segue un approccio "default chiuso": i dataset appena inseriti rimangono inaccessibili fino al completamento di una scansione automatizzata e di una revisione umana. Lo strumento Skimmer combina classificazione automatica con analisi AI per rilevare dati sensibili (PII), che vengono poi validati da revisori umani prima della concessione dell'accesso. Questo meccanismo garantisce che la velocità di ingestion non comprometta la sicurezza dei dati.

Skipper è l'agente AI di analisi costruito sopra Town Lake: traduce richieste in linguaggio naturale in SQL validato usando metadati, definizioni di schema, lineage delle trasformazioni e ispezione a runtime. L'agente viene utilizzato per analisi di fatturazione, investigazioni del supporto clienti, business intelligence e workflow di sicurezza. I workload di fatturazione rappresentano il 53% di tutte le query sulla piattaforma (91.760 query da 324 dipendenti nel periodo di misurazione), confermando quanto il self-service analitico sia critico per le operazioni aziendali.

I piani futuri includono una maggiore integrazione di Skipper con chat interne, sistemi di ticketing e workflow di sviluppo, l'espansione del pipeline Transformer per dataset curati definiti tramite SQL con deployment e monitoraggio automatici, e la migrazione di workload aggiuntivi di Town Lake su R2 SQL. Il caso Cloudflare dimostra come l'architettura lakehouse possa essere adottata anche internamente, non solo come offerta di prodotto verso clienti.
