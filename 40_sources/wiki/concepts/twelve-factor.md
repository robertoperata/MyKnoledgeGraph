---
title: Twelve-Factor Application
type: concept
tags: [thread1-microservices, thread3-cloud]
sources:
  - "[[twelve-factor application]]"
updated: 2026-04-09
related:
  - "[[concepts/independent-deployability]]"
  - "[[technologies/kubernetes]]"
---

# Twelve-Factor Application

## Definizione

Metodologia per costruire applicazioni software-as-a-service (SaaS) progettate per deploy su piattaforme cloud, massima portabilità e scalabilità orizzontale. Scritta da Adam Wiggins di Heroku.

## I 12 fattori

| # | Fattore | Principio chiave |
|---|---|---|
| 1 | **Codebase** | Un codebase per applicazione, tracciato in version control |
| 2 | **Dependencies** | Dichiarazione esplicita e isolamento delle dipendenze (no system-wide packages) |
| 3 | **Config** | La config è nell'ambiente (env vars), non nel codice |
| 4 | **Backing Services** | Trattare i backing services come risorse connesse (database, queue, cache) |
| 5 | **Build/Release/Run** | Separazione netta tra build, release e run stage |
| 6 | **Processes** | Processi stateless e share-nothing; sticky sessions = violazione |
| 7 | **Port Binding** | L'app è completamente self-contained; espone servizi tramite port binding |
| 8 | **Concurrency** | Scalare orizzontalmente tramite il process model |
| 9 | **Disposability** | Processi avviabili e stoppabili al volo; graceful shutdown su SIGTERM |
| 10 | **Dev/Prod Parity** | Gap minimo tra development e production (stessi backing services) |
| 11 | **Logs** | L'app non gestisce routing/storage dei log; li scrive sullo stdout come event stream |
| 12 | **Admin Processes** | Task amministrativi come one-off process nell'ambiente identico all'app |

## Connessione con Microservizi

I principi twelve-factor supportano direttamente l'**independent deployability**:
- **Fattore 3 (Config)**: env vars per la config → nessun rebuild per cambiare ambienti → release pipeline più semplice
- **Fattore 6 (Stateless)**: processi stateless → puoi deployare N istanze senza coordinamento → scaling orizzontale
- **Fattore 9 (Disposability)**: graceful shutdown → rolling deploy senza downtime in Kubernetes

## Connessione con Kubernetes

Kubernetes è progettato per ospitare applicazioni twelve-factor:
- **Config** → ConfigMap e Secret
- **Backing Services** → Service resources
- **Disposability** → Pod lifecycle, graceful termination
- **Concurrency** → HorizontalPodAutoscaler
- **Logs** → log aggregation (Fluentd, ELK) dai stdout dei container

## Connessioni

- [[concepts/independent-deployability]] — i 12 fattori sono i prerequisiti tecnici per l'independent deployability
- [[technologies/kubernetes]] — K8s implementa nativamente molti principi twelve-factor
