---
tags:
  - kafka
  - apache-iceberg
  - data-lakehouse
  - event-streaming
  - diskless-topics
  - open-table-format
feature:
type: article
author: Stanislav Kozlovski
source: https://blog.2minutestreaming.com/p/what-is-a-lakestream
date: 2026-05-09
---

# What is a Lakestream?

## Sunto

In 2026 il paradigma dominante per lo storage analitico è il **data lakehouse**: un'astrazione tabellare mutabile costruita sopra file immutabili strutturati (tipicamente Parquet su S3) tramite Open Table Format come Apache Iceberg o Delta Lake. A differenza di un data warehouse classico, il lakehouse disaccoppia lo storage dal compute — chiunque porta il proprio motore di query e legge gli stessi dati senza duplicarli (architettura zero-copy). Questo consente governance, consistenza transazionale e query SQL su dati che fisicamente risiedono in un object storage economico.

In parallelo, il trend dominante nell'ecosistema Kafka nel 2026 è lo spostamento dei dati verso l'object storage. Il Tiered Storage (KIP-405) è ormai standard, e i **Diskless Topics** (KIP-1150) sono in arrivo. I dati Aiven su oltre 4000 cluster confermano che circa il 60% dei sink connector punta a destinazioni compatibili con Iceberg (Databricks, Snowflake, S3), e circa l'85% del throughput totale in uscita da Kafka finisce nel lake. In sostanza: Kafka è già un producer del lake.

Da questa convergenza nasce il concetto di **LakeStream** — coniato da StreamNative — che tratta gli stream di eventi come primitive di prima classe del lakehouse. Invece di separare il mondo operativo (Kafka) da quello analitico (lake), il LakeStream unifica i due: i topic Kafka vengono memorizzati nativamente in un Open Table Format, rendendoli interrogabili da qualsiasi motore senza duplicazione, senza ETL e senza la necessità di passare attraverso un consumer Kafka.

I benefici dell'integrazione nativa sono notevoli: storage infinito a basso costo (S3 + Parquet con compressione ottima), eliminazione del vendor lock-in sul query engine, e dati immediatamente utili per tutta l'organizzazione. L'architettura classica di Kafka trattiene i dati in un formato proprietario (log segment) leggibile solo da Kafka stessa, richiedendo inoltre uno schema registry separato. Con il LakeStream, creare un topic equivale a creare una tabella Iceberg — qualcosa che Databricks Zerobus ha già dimostrato con un semplice `CREATE TABLE`.

L'implementazione più avanzata di questa visione è **Ursa for Kafka** di StreamNative: un fork minimale di Kafka che aggiunge un nuovo tipo di topic abilitabile per-topic con la configurazione `ursa.storage.enabled`. Questi topic sono interamente diskless e leaderless, e scrivono nativamente in un Open Table Format. Ursa combina la flessibilità di avere profili di topic distinti (classico vs diskless) con il supporto nativo agli OTF. StreamNative prevede di aprire il sorgente di questo fork intorno a gennaio 2027; nel frattempo ha già rilasciato il [protocollo leaderless log formalmente verificato](https://github.com/lakestream-io/leaderless-log-protocol/) su cui Ursa si basa.

## Immagini

![Architettura high-level di un data lakehouse con zero-copy](https://media.beehiiv.com/cdn-cgi/image/fit=scale-down,format=auto,onerror=redirect,quality=80/uploads/asset/file/55e255a9-74fb-4298-81ce-b905780b2936/Second_Charts_Email_Size__17_.png)

![Architettura diskless ("direct-to-s3") vs topic tiered classico in Kafka](https://media.beehiiv.com/cdn-cgi/image/fit=scale-down,format=auto,onerror=redirect,quality=80/uploads/asset/file/4a8ea783-6bf3-4501-9338-eea7bf19eff0/Second_Charts_Email_Size__21_.png)

![Visione ambiziosa di uno stack lakestream: Kafka + OTF nativo](https://media.beehiiv.com/cdn-cgi/image/fit=scale-down,format=auto,onerror=redirect,quality=80/uploads/asset/file/5d1ed8ec-d0e2-414d-b482-935e4c96eaf6/Second_Charts_Email_Size__12_.png)

![Architettura zero-copy con Ursa topics](https://media.beehiiv.com/cdn-cgi/image/fit=scale-down,format=auto,onerror=redirect,quality=80/uploads/asset/file/13ed1e03-b1a4-4aee-8ded-d22e88960108/Second_Charts_Email_Size__14_.png)

![Convergenza inevitabile: diskless topics + Iceberg native support](https://media.beehiiv.com/cdn-cgi/image/fit=scale-down,format=auto,onerror=redirect,quality=80/uploads/asset/file/d58c3e51-ac08-4794-b97c-3b111fe5cf83/Second_Charts_Email_Size__20_.png)
