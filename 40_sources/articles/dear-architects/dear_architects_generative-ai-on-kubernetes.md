---
tags:
  - kubernetes
  - generative-ai
  - llm
  - mlops
  - deployment
feature:
type: article
author: Dear Architects
source: https://www.amazon.com/Generative-AI-Kubernetes-Operationalizing-Language/dp/1098171926
date: 2026-05-03
---

# Generative AI on Kubernetes

## Sunto

Il libro "Generative AI on Kubernetes" è una guida pratica per la gestione di applicazioni AI generative in produzione su Kubernetes. Pubblicato da O'Reilly, il volume affronta le complessità operative del deployment di Large Language Models (LLM) su larga scala, guidando architetti e ingegneri attraverso le decisioni architetturali critiche relative a GPU, performance e pattern di deployment che determinano il successo o il fallimento di un'iniziativa AI in produzione.

Il libro copre l'intera catena operativa degli LLM su Kubernetes: dalla configurazione dei nodi con GPU e driver appropriati, alla gestione dello scheduling di workload AI-intensivi con risorse hardware specializzate, fino alle strategie di autoscaling che bilanciano costo e latenza. Vengono trattate le architetture di serving per l'inferenza — pattern come model sharding, tensor parallelism e pipeline parallelism per modelli che non entrano in memoria su una singola GPU — e le configurazioni di rete ottimizzate per l'alta larghezza di banda richiesta dalla comunicazione inter-GPU.

Un capitolo significativo è dedicato ai trade-off architetturali tra diverse strategie di deployment: hosting on-premise su Kubernetes vs. utilizzo di API cloud per l'inferenza, fine-tuning vs. RAG (Retrieval-Augmented Generation) vs. prompt engineering, e le implicazioni di ciascun approccio su latenza, costo, controllo dei dati e compliance. Il libro fornisce anche esempi concreti di integrazione con l'ecosistema CNCF: utilizzo di Knative per serverless inference, Prometheus e Grafana per il monitoring dei modelli, e Istio per il traffic management verso endpoint di inferenza.

L'aspetto più pratico del volume riguarda la costruzione di pipeline MLOps su Kubernetes: continuous training, versioning dei modelli, A/B testing tra versioni di modelli in produzione, e rollback sicuro in caso di regressioni nei comportamenti del modello. Questi pattern trasformano il deployment degli LLM da un'operazione one-shot a un processo continuo e governabile, allineando la gestione dei modelli AI alle best practice consolidate del software delivery moderno.
