---
tags:
  - machine-learning
  - data-loading
  - columnar-formats
  - gpu-computing
  - ml-infrastructure
feature:
type: article
author: Onur Satici
source: https://r.mail.infoq.com/mk/cl/f/sh/7nVU1aA2ng7ZOxOduCzgtt6Bp5Q89P7/nRK-jLpakl9T
date: 2026-09-26
---

# From S3 to GPU in One Copy: Rethinking Data Loading for ML Training

## Sunto

Onur Satici, in questa presentazione dal QCon London, descrive Vortex, un formato di file colonnare open-source sotto la Linux Foundation, progettato per eliminare i colli di bottiglia tradizionali nel data loading per il training di modelli ML. Il problema che Vortex risolve è fondamentale nell'infrastruttura ML moderna: spostare dati da S3 alla GPU implica normalmente multiple copie in memoria (deserializzazione, conversione di formato, preprocessing), ognuna delle quali consuma CPU, NVMe bandwidth, e tempo prezioso che rallenta l'utilizzo delle GPU.

L'architettura di Vortex si basa su tre innovazioni principali. La prima è l'uso di *cascading lightweight encodings*: invece di una singola codifica pesante, vengono applicate in cascata codifiche leggere specifiche per il tipo di dati — run-length encoding, bit-packing, dictionary encoding — che si combinano in modo da massimizzare la compressione preservando la possibilità di operare direttamente sui dati compressi senza decompressione completa. La seconda è il *layout-based segment pruning*: i metadati del file registrano statistiche per segmento (min/max, null counts) che permettono di saltare interi blocchi di dati non rilevanti senza leggerli da S3.

La terza e più rilevante innovazione è la pipeline *zero-copy memory*: sfruttando RDMA (Remote Direct Memory Access) e le capacità di DMA diretto dei driver GPU moderni, Vortex è in grado di trasferire dati da S3 direttamente nella memoria GPU senza passaggi intermedi attraverso la CPU o la memoria sistema. Il risultato misurato è un throughput di fino a 60 Gbps nel caricamento dei dati, eliminando il preprocessing upfront dei dataset — i dati vengono processati on-the-fly durante il caricamento. Questo riduce significativamente il tempo di avvio dei job di training e il costo di storage dato che i dati non necessitano di essere pre-convertiti in formato ottimizzato per GPU.

Il formato è stato progettato con l'ecosistema ML in mente: compatibilità con Apache Arrow per l'interoperabilità, supporto per tensori multi-dimensionali nativi, e integrazione con i principali framework di training (PyTorch, JAX). L'apertura sotto la Linux Foundation segnala l'intenzione di renderlo uno standard de facto per il data loading nell'infrastruttura ML, competendo con formati come Parquet (ottimizzato per analytics) e TFRecord (ottimizzato per TensorFlow ma non portabile).
