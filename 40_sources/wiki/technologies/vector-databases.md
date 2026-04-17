---
title: Vector Databases
type: technology
tags: [thread4-ai, thread3-cloud]
sources:
  - "[[infoq_aws-s3-vectors-ga-storage-first-rag]]"
  - "[[infoq_netflix-graph-abstraction-650tb-milliseconds]]"
updated: 2026-04-17
related:
  - "[[concepts/agentic-patterns]]"
  - "[[concepts/spec-driven-development]]"
---

# Vector Databases

## Definizione

Sistemi di storage ottimizzati per l'archiviazione e la ricerca di vettori (embeddings). Fondamentali per applicazioni RAG (Retrieval-Augmented Generation) e semantic search.

## Paradigma: Compute-First vs Storage-First

| Compute-First (tradizionale) | Storage-First (es. S3 Vectors) |
|---|---|
| Cluster, pod, shard da gestire | Nessuna infrastruttura da gestire |
| Scaling manuale | Scaling automatico |
| TCO elevato | TCO ridotto fino al 90% |
| Segregazione per dataset | Fino a 2 miliardi di vettori in un singolo indice |

## AWS S3 Vectors (GA, gennaio 2026)

Servizio AWS che integra vector storage nativo in S3:

| Caratteristica | Valore |
|---|---|
| Capacità per indice | 2 miliardi di vettori |
| Latenza query frequenti | ≤ 100ms |
| Latenza query non frequenti | < 1 secondo |
| Throughput scrittura | 1.000 PUT/s per vettore |
| Regioni | 14 |

**Integrazioni native:**
- Amazon Bedrock Knowledge Base (RAG)
- Amazon OpenSearch (search + analytics)

## Pattern RAG con Vector Storage

```
Documenti → Encoder → Embeddings → Vector Store
                                        ↓
Query → Encoder → Query Vector → Similarity Search → Contesto rilevante
                                                           ↓
                                                    LLM → Risposta
```

## Graph Databases at Scale (Netflix)

Alternativa ai vector database per dati relazionali: Graph Abstraction di Netflix gestisce 650 TB di dati a grafo in millisecondi.

**Trade-off fondamentale:** flessibilità vs latenza
- Database a grafo tradizionali: traversal arbitrari, alta flessibilità, latenza variabile
- Graph Abstraction Netflix: traversal depth limitata, nodi di partenza definiti, latenza garantita

**Risultati:** latenza a singola cifra ms per traversal a 1 hop, <50ms per 2 hop al p90.

**Caching strategy:**
- Write-aside caching: previene scritture duplicate degli edge
- Read-aside caching: accelera l'accesso alle proprietà degli edge

## Connessioni

- [[concepts/agentic-patterns]] — i vector database sono l'infrastruttura per la memoria degli agenti AI
- [[concepts/spec-driven-development]] — RAG come meccanismo per arricchire il contesto delle specifiche
