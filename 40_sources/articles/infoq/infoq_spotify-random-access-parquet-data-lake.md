---
tags:
  - data-lake
  - apache-parquet
  - persistence
  - indexing
  - apache-iceberg
feature:
type: article
author: InfoQ
source: https://www.infoq.com/news/2026/08/spotify-data-lake-point-queries/
date: 2026-08-29
---

# Spotify Builds External Index to Enable Low Latency Point Queries on Its Data Lake

## Sunto

Spotify ha introdotto Random Access Parquet (RAP), un'architettura di storage che aggiunge un livello di indicizzazione esterno sopra i file Apache Parquet nel data lake, abilitando query puntuali a bassa latenza senza dover replicare i dataset in database operazionali separati. Il problema di partenza è strutturale: i query engine come Trino e BigQuery sono ottimizzati per la scansione di masse di dati, non per recuperare singoli record tramite chiave. Spotify gestisce petabyte su Bigtable ed exabyte su Google Cloud Storage, rendendo la replicazione su larga scala economicamente insostenibile.

Il meccanismo centrale di RAP è un indice esterno che mappa chiavi di lookup (es. user ID) a specifici file Parquet e posizioni di riga. Invece di scansionare migliaia di file, una query risolve la chiave attraverso l'indice ed emette una lettura a range mirata contro l'object storage. I frammenti dell'indice vengono generati come strutture append-only man mano che le tabelle Apache Iceberg ricevono nuovi dati, senza modificare i file Parquet immutabili sottostanti.

Le tecniche di ottimizzazione includono: ordinamento dei dati per chiave di lookup per minimizzare l'accesso ai file, raggruppamento di record correlati, interleaving delle colonne di valore per letture contigue in un solo accesso, e indici di copertura (covering indexes) che soddisfano la query senza aprire il file Parquet. RAP supporta indici secondari di tipo hash (per exact match) e sorted (per range query), gestiti a livello serving senza modifiche alle pipeline.

Per migliorare la data locality vengono applicate tecniche come Z-ordering e Hilbert curves, che ordinano i dati multidimensionalmente così che record con chiavi correlate risultino fisicamente vicini nello storage. Questo riduce il numero di letture necessarie per query con predicati multipli o range su campi diversi.

L'impatto architetturale principale è la capacità di servire lo stesso dataset per analisi, machine learning, notebook, agenti AI e applicazioni online latency-sensitive in parallelo, eliminando la necessità di mantenere sistemi di storage duplicati. Spotify può così abbattere il costo operativo e la complessità derivante dalla sincronizzazione tra data lake e database operazionali.
