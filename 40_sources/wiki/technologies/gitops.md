---
title: GitOps
type: technology
tags:
  - thread1-microservices
  - thread3-cloud
sources:
  - "[[chris_richardson_microservices-platforms-part-7-deployment-platform]]"
updated: 2026-05-07
related:
  - "[[concepts/microservices-platform]]"
  - "[[technologies/kubernetes]]"
  - "[[concepts/twelve-factor]]"
---

# GitOps

## Ruolo

Modalità operativa per la gestione dichiarativa dello stato infrastrutturale: **Git come source of truth** per la configurazione degli ambienti di produzione. Parte fondamentale della Deployment Platform.

## Come funziona

```
Developer → commit → Git repo (desired state)
                          ↓
                   GitOps controller (ArgoCD / Flux)
                   confronta desired state vs current state
                          ↓
                   Kubernetes cluster (actual state)
                   applica automaticamente le differenze
```

Il flusso è **pull-based**: il controller nel cluster osserva il repo Git e **tira** le modifiche, invece di avere una pipeline CI/CD che **spinge** configurazioni nel cluster. Questo garantisce:
- Auditability: ogni cambiamento infrastrutturale è un commit Git
- Drift detection: se qualcuno modifica manualmente il cluster, il controller lo rileva e corregge
- Rollback: basta fare `git revert` del commit problematico

## Tool principali

| Tool | Caratteristiche |
|---|---|
| **ArgoCD** | UI ricca, multi-cluster, sync manuale o automatica, RBAC integrato |
| **Flux** | Più leggero, cloud-native, multi-tenancy, supporto OCI registries |

## Integrazione con il Deployment Platform (Richardson)

Nel framework delle Microservices Platforms, il Deployment Platform usa GitOps in combinazione con:
- **Infrastructure as Code** (Terraform, Pulumi) per definire le risorse cloud
- **Kubernetes** per orchestrazione container
- **Helm / Kustomize** per la configurazione dei manifesti K8s

Il path to production diventa:
```
Developer → commit code → Build Platform (CI) → artefatto (immagine Docker)
                              ↓
              aggiorna manifest GitOps repo (bump image tag)
                              ↓
              GitOps controller → deploy in staging → produzione
```

## Connessioni con Twelve-Factor

| Principio | GitOps |
|---|---|
| **Config** (separare configurazione da codice) | Il repo GitOps contiene la configurazione dell'ambiente, separata dal codice dell'applicazione |
| **Build/Release/Run** | Build Platform produce l'artefatto; GitOps gestisce Release e Run |

## Connessioni

- [[concepts/microservices-platform]] — GitOps è il cuore del Deployment Platform
- [[technologies/kubernetes]] — GitOps gestisce lo stato Kubernetes
- [[concepts/twelve-factor]] — principio 3 (Config) e principio 5 (Build/Release/Run)
