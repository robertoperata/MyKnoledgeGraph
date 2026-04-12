---
title: Istio Service Mesh
type: technology
tags: [thread3-cloud, thread1-microservices]
sources:
  - "[[Managing Microservices with Kubernetes and Istio]]"
  - "[[Certified Kubernetes Application Developer]]"
updated: 2026-04-09
related:
  - "[[technologies/kubernetes]]"
  - "[[concepts/independent-deployability]]"
---

# Istio

## Ruolo

Istio è un **service mesh** che si inserisce tra i microservizi in Kubernetes per gestire la comunicazione service-to-service in modo trasparente alle applicazioni. Le funzionalità vengono iniettate tramite sidecar proxy (Envoy) automaticamente in ogni Pod.

## Funzionalità principali

### Traffic Management
- **Routing avanzato**: instradamento basato su header, percentuale di traffico, versione
- **Canary release**: graduale rollout di nuove versioni (es. 5% → 20% → 100%)
- **Traffic mirroring**: copia del traffico production verso una versione di test senza impatto
- **Circuit breaking**: stop automatico verso servizi non sani

### Security
- **mTLS automatico**: cifratura del traffico tra tutti i servizi senza modifiche al codice
- **Authorization policies**: regole di autorizzazione a livello di rete (chi può parlare con chi)

### Observability
- **Metriche**: latency, error rate, throughput per ogni coppia sorgente-destinazione
- **Distributed tracing**: traccia le richieste attraverso la chain di microservizi
- **Access logging**: log di ogni richiesta

## Connessione con Canary Deployment

Il Canary Deployment nei microservizi (descritto anche in Vercel per i deployment agentic) diventa naturale con Istio:

```yaml
# VirtualService: 5% del traffico alla versione v2
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
spec:
  http:
  - route:
    - destination:
        host: my-service
        subset: v1
      weight: 95
    - destination:
        host: my-service
        subset: v2
      weight: 5
```

Rollback automatico se le metriche degradano → questo è il "self-driving deployment" di cui parla Vercel.

## Connessioni

- [[technologies/kubernetes]] — Istio si installa sopra K8s
- [[concepts/independent-deployability]] — Istio abilita deployment sicuri con canary e rollback
- [[concepts/agentic-patterns]] — le "self-driving deployments" di Vercel si basano su meccanismi come Istio
