---
tags:
  - software-architecture
  - ai-gateway
  - ai-agents
  - evolutionary-architecture
  - security
feature:
type: article
author: Joe Price, Branimir Đurek, Pavlos Migkiros, Trevor Dearham
source: https://www.infoq.com/articles/evolutionary-architecture-pattern/
date: 2026-08-29
---

# An Evolutionary Architecture Pattern for Managing AI's Pace of Change

## Sunto

L'articolo affronta una sfida architetturale fondamentale: le capacità dell'AI evolvono molto più rapidamente di quanto i sistemi enterprise possano assorbire in modo sicuro. Gli autori — partecipanti all'InfoQ Certified Architect Program — propongono l'**AI Gateway** come pattern architetturale che isola i componenti più volatili dei sistemi AI (guardrail, model routing, agent identity, action policy, audit semantico) permettendo al resto della piattaforma di rimanere stabile.

Il gap critico rispetto agli API gateway tradizionali è strutturale. I gateway classici assumono tre proprietà che i sistemi agentici violano sistematicamente: (1) **determinismo** — lo stesso input produce lo stesso output, mentre gli agenti variano reasoning path su prompt identici; (2) **failure a livello schema** — i gateway rilevano richieste malformate, mentre i fallimenti agentici sono semantici (richieste valide che prendono decisioni sbagliate); (3) **intent client-specified** — i gateway validano azioni predeterminate, mentre gli agenti interpretano obiettivi e scelgono autonomamente quali tool invocare.

L'AI Gateway proposto centralizza diversi control plane. Il **model routing** isola il churn nei provider in termini di capacità e pricing. I **security control** coprono identità e delegated authorization, zero-trust policy per singola azione, segmentazione di ciò che gli agenti possono raggiungere, e content guarding bidirezionale contro prompt injection e data leakage. L'**observability** cattura il pattern Request → Decision → Action per ogni step, producendo log semantici utilizzabili sia per monitoring operativo che come evidenza regolatoria.

La configurazione dei guardrail è espressa come configuration versionata:

```yaml
agent: code-review-agent
authorization:
  allow:
    - repo.read
    - pr.comment
  deny:
    - repo.push
    - secrets.read
guardrails:
  input:
    - prompt_injection_filter
  output:
    - secret_redaction
routing:
  default: gpt-4o
  fallback: self-hosted-slm
```

Gli autori descrivono quattro stadi di adozione organizzativa: (1) singolo team, singolo provider con rischio limitato; (2) adozione frammentata cross-team con scelte incoerenti; (3) evento forzante (incidente o pressione regolatoria) che obbliga alla consolidazione; (4) governance centralizzata su un layer gateway condiviso. Organizzazioni con platform engineering maturo adottano il pattern proattivamente; la maggior parte lo adotta reattivamente dopo un incidente, a costo più elevato. Incidenti documentati includono la cancellazione del database di produzione da parte di un agente Replit (luglio 2025, 1196 aziende impattate), spesa non autorizzata da 500M$/mese in licenze AI, e la prima CVE di prompt injection armata (EchoLeak).

## Codice

La policy-as-config per un AI Gateway che gestisce routing, authorization e guardrail per un agente di code review:

```yaml
agent: code-review-agent
authorization:
  allow:
    - repo.read
    - pr.comment
  deny:
    - repo.push
    - secrets.read
guardrails:
  input:
    - prompt_injection_filter
  output:
    - secret_redaction
routing:
  default: gpt-4o
  fallback: self-hosted-slm
```
