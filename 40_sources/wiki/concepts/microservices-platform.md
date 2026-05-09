---
title: Microservices Platform
type: concept
tags:
  - thread1-microservices
  - thread3-cloud
sources:
  - "[[chris_richardson_microservices-platforms-part-1-overview]]"
  - "[[chris_richardson_microservices-platforms-part-2-service-foundation-platform]]"
  - "[[chris_richardson_microservices-platforms-part-3-security-platform]]"
  - "[[chris_richardson_microservices-platforms-part-4-infrastructure-services-platform]]"
  - "[[chris_richardson_microservices-platforms-part-5-observability-platform]]"
  - "[[chris_richardson_microservices-platforms-part-6-build-platform]]"
  - "[[chris_richardson_microservices-platforms-part-7-deployment-platform]]"
updated: 2026-05-07
related:
  - "[[concepts/team-topologies]]"
  - "[[concepts/microservice-chassis]]"
  - "[[concepts/observability]]"
  - "[[technologies/kubernetes]]"
  - "[[technologies/gitops]]"
  - "[[technologies/istio]]"
---

# Microservices Platform

## Definizione

L'insieme strutturato di piattaforme interne che una organizzazione deve costruire per abilitare il fast flow nell'architettura a microservizi. Senza queste piattaforme, i team applicativi spendono la maggior parte del tempo su problemi infrastrutturali anziché sulla logica di dominio, generando il **modern legacy system** — una nuova architettura con gli stessi vecchi problemi organizzativi.

Framework: "Microservices Platforms: When Team Topologies Meets Microservices Patterns" (Chris Richardson, QCon San Francisco 2025).

## I sei layer della piattaforma

```
┌─────────────────────────────────────────────────────────┐
│              DEPLOYMENT PLATFORM                        │  ← ambienti prod/pre-prod
├─────────────────────────────────────────────────────────┤
│              BUILD PLATFORM                             │  ← CI/CD pipeline, artifact registry
├─────────────────────────────────────────────────────────┤
│              OBSERVABILITY PLATFORM                     │  ← metrics, logs, traces, dashboards
├─────────────────────────────────────────────────────────┤
│          INFRASTRUCTURE SERVICES PLATFORM               │  ← K8s, service mesh, IaC
├─────────────────────────────────────────────────────────┤
│              SECURITY PLATFORM                          │  ← IdP, mTLS, secrets, policy
├─────────────────────────────────────────────────────────┤
│          SERVICE FOUNDATION PLATFORM                    │  ← microservice chassis, service template
└─────────────────────────────────────────────────────────┘
```

### 1. Service Foundation Platform

**Problema:** ogni nuovo servizio deve affrontare le stesse sfide ricorrenti (bootstrapping, health check, tracing, config esternalizzata).

**Soluzione:**
- **[[concepts/microservice-chassis]]**: framework/librerie che incapsulano le preoccupazioni cross-cutting
- **Service template**: scaffolding che genera la struttura iniziale già integrata con il chassis

**Outcome**: un team può creare un nuovo microservizio pronto al deploy in pochi minuti.

### 2. Security Platform

**Problema:** sicurezza pervasiva e complessa (autenticazione, autorizzazione inter-servizio, segreti, TLS, audit logging).

**Soluzione:**
- **Identity Provider centralizzato** (es. Keycloak, Auth0): autenticazione e rilascio token OAuth 2.0 / OIDC
- **Service-to-service authentication**: mTLS via service mesh o JWT condivisi
- **Secrets management** (es. HashiCorp Vault): distribuzione sicura delle credenziali
- **Policy di autorizzazione** centralizzate

**Outcome**: servizi sicuri by default, senza che ogni team debba implementare sicurezza autonomamente.

### 3. Infrastructure Services Platform

**Problema:** complessità infrastrutturale elevata (K8s, service mesh, service discovery, networking, IaC).

**Soluzione:**
- Container orchestration (Kubernetes) con template standardizzati (Helm, Kustomize)
- Service mesh (Istio, Linkerd) per load balancing, retry, circuit breaking, mTLS
- Developer portal / CLI interne con interfacce semplificate

**Outcome**: un team fa il deploy in Kubernetes senza conoscere i dettagli di networking del cluster.

### 4. Observability Platform

**Problema:** costruire osservabilità (metrics, logs, traces, dashboards) è complesso e time-consuming.

**Soluzione:**
- Raccolta centralizzata di metriche (Prometheus, regole di alerting)
- Aggregazione log (ELK, Loki) con formato strutturato imposto dal chassis
- Distributed tracing (Jaeger, Zipkin)
- Dashboard pre-costruite (Grafana, RED metrics: Rate, Errors, Duration)
- Alerting centralizzato

**Nota importante**: la piattaforma non esime i team dall'aggiungere instrumentazione domain-specific (span significativi, metriche di business). La piattaforma fornisce l'infrastruttura; il team fornisce il contesto.

**Outcome**: ogni nuovo servizio è osservabile by default dal momento del primo deploy.

### 5. Build Platform

**Problema:** ogni microservizio ha bisogno di CI/CD pipeline; configurare e mantenere N pipeline richiede expertise specializzata.

**Soluzione:**
- Shared pipeline infrastructure (Jenkins, GitHub Actions runners, CircleCI)
- Reusable pipeline components (GitHub Actions reusable workflows, CircleCI Orbs)
- Deployment pipeline templates
- Artifact registry centralizzato (immagini Docker, artefatti)

**Outcome**: un team che usa il service template ottiene automaticamente anche un pipeline funzionante.

### 6. Deployment Platform

**Problema:** fornire ambienti prod/pre-prod consistenti e gestibili per decine di servizi.

**Soluzione:**
- Kubernetes per orchestrazione container
- **GitOps** (ArgoCD, Flux) per gestione dichiarativa dello stato infrastrutturale
- Infrastructure as Code per ambienti riproducibili
- Template di deployability

**Outcome**: i team rilasciano in autonomia senza occuparsi della complessità infrastrutturale.

## Tensioni identificate

> **Tensione:** costruire e mantenere tutte e sei le piattaforme richiede investimento significativo. Per organizzazioni piccole può essere overkill — il rischio è costruire piattaforme che nessuno usa. Richardson suggerisce di iniziare dalle piattaforme che generano il maggiore time-to-value per i team applicativi (tipicamente: Service Foundation + Observability + Build).

## Connessioni

- [[concepts/team-topologies]] — il Platform Team è responsabile di costruire e operare le piattaforme
- [[concepts/microservice-chassis]] — componente chiave della Service Foundation Platform
- [[concepts/observability]] — l'Observability Platform implementa il concetto di osservabilità distribuita
- [[technologies/kubernetes]] — base tecnologica della Infrastructure Services e Deployment Platform
- [[technologies/gitops]] — modalità operativa della Deployment Platform
- [[technologies/istio]] — service mesh usato nell'Infrastructure Services Platform
- [[technologies/keycloak]] — Identity Provider per la Security Platform
