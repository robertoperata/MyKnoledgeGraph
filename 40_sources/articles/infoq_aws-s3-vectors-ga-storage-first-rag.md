---
tags:
  - aws
  - ai
  - database
  - cloud
  - architecture
type: article
author: Steef-Jan Wiggers
source: https://www.infoq.com/news/2026/01/aws-s3-vectors-ga/
date: 2026-01-02
---

# Amazon S3 Vectors Reaches GA, Introducing "Storage-First" Architecture for RAG

## Sunto

AWS ha annunciato il rilascio in GA (General Availability) di **S3 Vectors**, un servizio di object storage cloud con supporto nativo per l'archiviazione e l'interrogazione di dati vettoriali. La novità principale rispetto alla fase di preview (luglio 2025) è un aumento di quaranta volte della capacità per indice, che arriva ora a **2 miliardi di vettori** con latenze di query inferiori a 100ms. Durante la preview, gli utenti avevano già creato oltre 250.000 indici e inserito più di 40 miliardi di vettori.

Il cambiamento architetturale centrale è il passaggio da un approccio *Compute-First* — tipico dei database vettoriali tradizionali basati su cluster, pod e shard — a un approccio **Storage-First**. Con S3 Vectors non è più necessario gestire l'infrastruttura sottostante: l'intero dataset vettoriale può essere consolidato in un singolo indice, riducendo il total cost of ownership fino al **90%**.

> "You can now store and search across up to 2 billion vectors in a single index... This means you can consolidate your entire vector dataset into a single index."
> — Sebastian Stromacq, Principal Developer, AWS

Le caratteristiche tecniche chiave del servizio GA:

| Caratteristica | Valore |
|---|---|
| Capacità per indice | 2 miliardi di vettori |
| Latenza query frequenti | ≤ 100ms |
| Latenza query non frequenti | < 1 secondo |
| Risultati per query | Fino a 100 |
| Throughput scrittura | Fino a 1.000 PUT/s per singolo vettore |
| Regioni disponibili | 14 (vs 5 in preview) |

Il servizio si integra nativamente con **Amazon Bedrock Knowledge Base** (per applicazioni *RAG*) e con **Amazon OpenSearch** (per ricerca e analytics). Il pricing si articola su tre dimensioni: costo per PUT (basato sui GB logici caricati), costo di storage (spazio logico totale), e costo per query (API call + $/TB in base alla dimensione dell'indice).

---

## Link esterni

- [AWS S3 Vectors Features](https://aws.amazon.com/s3/features/vectors/) — pagina ufficiale del servizio
- [S3 Vectors + Bedrock Knowledge Base](https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-vectors-bedrock-kb.html) — documentazione integrazione con Bedrock per RAG
- [S3 Vectors + OpenSearch](https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-vectors-opensearch.html) — documentazione integrazione con OpenSearch
- [S3 Pricing](https://aws.amazon.com/s3/pricing/) — dettagli pricing

---

## Immagini

Nessuna immagine presente
