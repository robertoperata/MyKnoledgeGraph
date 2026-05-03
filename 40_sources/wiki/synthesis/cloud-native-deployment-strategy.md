---
title: Strategia di deployment zero-downtime con Kubernetes, Istio e Twelve-Factor
type: synthesis
tags:
  - thread3-cloud
  - thread1-microservices
sources:
  - "[[kubernetes]]"
  - "[[istio]]"
  - "[[twelve-factor]]"
  - "[[independent-deployability]]"
  - "[[calm]]"
  - "[[agentic-patterns]]"
updated: 2026-04-18
related:
  - "[[technologies/kubernetes]]"
  - "[[technologies/istio]]"
  - "[[concepts/twelve-factor]]"
  - "[[concepts/independent-deployability]]"
  - "[[technologies/calm]]"
---

# Domanda originale

**Come si implementa una strategia di deployment zero-downtime per microservizi con Kubernetes e Istio, e come si collegano i principi Twelve-Factor a questa strategia?**

---

# Risposta sintetica

Il deployment zero-downtime non è una funzionalità da configurare: è il risultato di un'applicazione progettata secondo i principi Twelve-Factor, orchestrata da Kubernetes e con traffico gestito da Istio. I tre livelli si sovrappongono: Twelve-Factor garantisce che l'applicazione *possa* essere sostituita senza downtime, Kubernetes gestisce il *quando* e il *come*, Istio controlla il *quanto* del traffico.

---

## Livello 1 — Twelve-Factor: l'applicazione deve essere sostituibile

I principi Twelve-Factor non sono solo best practice di cloud: sono i **prerequisiti tecnici** perché Kubernetes possa fare il suo lavoro.

| Principio | Ruolo nel deployment zero-downtime |
|---|---|
| **Config (3)** | Env vars → nessun rebuild per cambiare configurazione tra ambienti |
| **Backing Services (4)** | Database, queue, cache come risorse connesse → sostituzione senza codice |
| **Build/Release/Run (5)** | Separazione netta → artifact immutabile per ogni release |
| **Stateless (6)** | Processi share-nothing → N istanze senza coordinamento → scaling orizzontale |
| **Disposability (9)** | Graceful shutdown su SIGTERM → rolling deploy senza downtime |
| **Dev/Prod Parity (10)** | Stessi backing services → no surprises in produzione |
| **Logs (11)** | Stdout come stream → aggregazione centralizzata senza configurazione |

**Il principio più critico per il zero-downtime:** il **Fattore 9 (Disposability)**. Un'applicazione che non gestisce SIGTERM lascia connessioni aperte quando viene rimpiazzata. Kubernetes invia SIGTERM e aspetta il `terminationGracePeriodSeconds` — l'app deve smettere di accettare nuove richieste e completare quelle in corso.

---

## Livello 2 — Kubernetes: orchestrazione del deployment

Kubernetes implementa nativamente i principi Twelve-Factor:

```yaml
# Config → ConfigMap e Secret
# Backing Services → Service resources
# Concurrency → HorizontalPodAutoscaler
# Disposability → terminationGracePeriodSeconds + preStop hook
```

### Rolling Deployment (default)

Kubernetes sostituisce i Pod uno alla volta, mantenendo sempre N istanze up:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0    # mai andare sotto la capacità
    maxSurge: 1          # creare un Pod in più durante il rollout
```

**Limite del rolling deployment:** non permette di testare la nuova versione con traffico reale prima di un rollout completo. Per questo serve Istio.

### Networking: come il traffico raggiunge i Pod

I Pod sono raggiungibili solo via Service (ClusterIP). Il Service bilancia il traffico tra tutti i Pod con il label selector corrispondente. Durante un rolling update, i nuovi Pod vengono aggiunti al pool del Service solo dopo che il readiness probe passa.

---

## Livello 3 — Istio: controllo fine del traffico

Istio si aggiunge sopra Kubernetes per il controllo fine del traffico senza modifiche al codice applicativo.

### Canary Release

```yaml
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
      weight: 5    # 5% del traffico alla nuova versione
```

Il traffico viene spostato gradualmente (5% → 20% → 50% → 100%) monitorando le metriche. Se la latenza o l'error rate aumentano, rollback automatico riportando il weight a 0.

**Connessione con gli agentic patterns:** questo è il *"self-driving deployment"* descritto in Vercel: canary release + rollback automatico basato su metriche = la prima linea di difesa per il deployment sicuro.

### Traffic Mirroring

```yaml
mirror:
  host: my-service
  subset: v2
mirrorPercentage:
  value: 100.0
```

Copia il traffico production verso la nuova versione *senza impatto sugli utenti*. La v2 risponde ma le risposte vengono scartate. Utile per testare sotto carico reale prima del canary.

### Security integrata

- **mTLS automatico:** cifratura del traffico tra tutti i servizi senza modifiche al codice
- **Authorization policies:** chi può parlare con chi, a livello di rete

---

## CALM: il percorso sicuro come percorso facile

CALM (Common Architecture Language Model) si inserisce *prima* di Kubernetes: i developer descrivono l'architettura, CALM genera automaticamente i manifest Kubernetes con le network policies per la micro-segmentazione.

> "The conformant path should be the easiest path."

Il developer non deve conoscere le NetworkPolicy — le ottiene gratis dal pattern CALM. Questo trasforma la sicurezza da gate bloccante a guardrail abilitante.

---

## La strategia completa

```
Twelve-Factor        →  l'app è sostituibile (stateless, graceful shutdown)
    ↓
Kubernetes           →  rolling deployment, readiness probe, HPA
    ↓
Istio                →  canary release, traffic mirroring, rollback automatico
    ↓
CALM                 →  network policies e sicurezza generate dal pattern
    ↓
Independent          →  ogni servizio deployabile senza coordinamento
Deployability
```

---

## Sorgenti utilizzate

- [[technologies/kubernetes]] — orchestrazione, Service types, NetworkPolicy
- [[technologies/istio]] — traffic management, canary release, mTLS
- [[concepts/twelve-factor]] — prerequisiti applicativi per il deployment zero-downtime
- [[concepts/independent-deployability]] — obiettivo finale della strategia
- [[technologies/calm]] — sicurezza come percorso facile
- [[concepts/agentic-patterns]] — self-driving deployment come pattern agentic

---

## Lacune / Argomenti da approfondire

- GitOps: ArgoCD e Flux per il deployment dichiarativo da git
- Helm: gestione dei template Kubernetes e release management
- Observability stack: Prometheus, Grafana, Jaeger per il monitoraggio delle release
- Service mesh alternatives: Linkerd vs Istio (trade-off semplicità/funzionalità)
- Progressive delivery avanzata: feature flags, A/B testing, Argo Rollouts
- Pod Disruption Budget: garantire disponibilità minima durante manutenzione cluster
- Readiness vs Liveness vs Startup probe: come configurarli correttamente
- Kubernetes RBAC: autorizzazione a livello di cluster e namespace
- Multi-cluster deployment: gestione di più cluster Kubernetes in ambienti multi-cloud
