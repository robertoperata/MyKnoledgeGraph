---
title: Kubernetes
type: technology
tags:
  - thread3-cloud
  - thread1-microservices
sources:
  - "[[Certified Kubernetes Application Developer]]"
  - "[[Managing Microservices with Kubernetes and Istio]]"
  - "[[chris_richardson_microservices-platforms-part-4-infrastructure-services-platform]]"
  - "[[chris_richardson_microservices-platforms-part-7-deployment-platform]]"
updated: 2026-05-07
related:
  - "[[concepts/twelve-factor]]"
  - "[[technologies/istio]]"
  - "[[concepts/independent-deployability]]"
  - "[[concepts/microservices-platform]]"
  - "[[technologies/gitops]]"
---

# Kubernetes

## Ruolo

Orchestratore di container per il deployment e la gestione di applicazioni distribuite. Implementa nativamente i principi [[concepts/twelve-factor]].

## Networking

Kubernetes crea due reti software-defined:
- **Cluster network**: rete interna isolata per uso amministrativo
- **Pod network**: rete interna per le applicazioni

I Pod hanno indirizzi IP solo sul pod network — non raggiungibili direttamente dall'esterno.

### Service Types

| Tipo | Accessibilità | Uso |
|---|---|---|
| **ClusterIP** (default) | Solo interna al cluster | Comunicazione inter-servizio |
| **NodePort** | Esterna (porta ~32000 su ogni nodo) | Dev/testing |
| **LoadBalancer** | External load balancer (cloud) | Production |
| **ExternalName** | Redirect DNS | Migrazione |
| **Headless** | Comunicazione diretta con Pod | StatefulSet |

### Service Ports

- **targetPort**: porta del container applicativo
- **port**: porta del Service sul cluster network
- **nodePort**: porta esposta esternamente (solo NodePort)

### DNS interno

I Service registrano automaticamente un nome DNS:
```
servicename.namespace.svc.clustername
```
Pod nello stesso namespace possono usare il nome breve (`servicename`). Da altri namespace serve il FQDN.

### Ingress e Gateway API

Per esporre servizi HTTP/HTTPS all'esterno senza NodePort:
- **Ingress**: reverse HTTP proxy, supporto TLS, routing per path/host
- **Gateway API**: evoluzione di Ingress con supporto TCP/UDP/gRPC, traffic splitting, WebSocket

### NetworkPolicy

Di default non ci sono restrizioni: tutti i Pod comunicano liberamente. Per limitare il traffico:
```yaml
# Richiede un network plugin che supporta NetworkPolicy (es. Calico, NON Weave)
kind: NetworkPolicy
spec:
  podSelector:
    matchLabels:
      app: db
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: backend
```

## Connessione con Istio

Istio si aggiunge sopra il networking Kubernetes per:
- Traffic management (routing avanzato, canary release, traffic mirroring)
- Security (mTLS automatico tra servizi)
- Observability (metriche, tracing, logging distribuito)

## Connessione con i principi Twelve-Factor

| Principio | Kubernetes |
|---|---|
| Config | ConfigMap e Secret |
| Backing Services | Service resources |
| Disposability | Pod lifecycle, graceful termination (SIGTERM) |
| Concurrency | HorizontalPodAutoscaler |
| Logs | Raccolta stdout via Fluentd/ELK |

## Kubernetes nella Microservices Platform (Richardson)

Nel framework delle [[concepts/microservices-platform]], Kubernetes è al centro di due layer:

**Infrastructure Services Platform**: orchestrazione container con template standardizzati (Helm chart, Kustomize overlay). I team applicativi fanno deploy senza conoscere i dettagli di networking. Kubernetes come infrastruttura self-service tramite Developer Portal / CLI interne.

**Deployment Platform**: Kubernetes è la runtime degli ambienti prod/pre-prod, gestito tramite [[technologies/gitops]] (ArgoCD, Flux) per la gestione dichiarativa dello stato.

## Connessioni

- [[technologies/istio]] — service mesh per gestire le comunicazioni tra microservizi
- [[concepts/twelve-factor]] — K8s implementa nativamente i principi twelve-factor
- [[concepts/independent-deployability]] — K8s abilita il rolling deployment per zero-downtime release
- [[concepts/microservices-platform]] — K8s è la base tecnologica della Infrastructure Services e Deployment Platform
- [[technologies/gitops]] — GitOps gestisce lo stato Kubernetes in modo dichiarativo
