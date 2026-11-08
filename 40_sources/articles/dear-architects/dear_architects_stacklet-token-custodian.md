---
tags:
  - ai-governance
  - token-budget
  - policy-as-code
  - llm-operations
  - cost-management
feature:
type: article
author: Stacklet
source: https://stacklet.ai/token-custodian/
date: 2026-11-08
---

# Token Custodian — Stacklet: Governance dei Token AI con Policy

## Sunto

Token Custodian è un control plane per la governance e l'attribuzione dei token AI, sviluppato dal team che ha creato Cloud Custodian — un motore di policy con oltre dieci anni di produzione. L'idea centrale è applicare lo stesso approccio policy-driven, già consolidato per le risorse cloud, alla spesa in token AI attraverso provider multipli.

Il prodotto risolve un problema comune nelle organizzazioni che adottano AI su larga scala: i token vengono spesi, ma nessuno sa esattamente da quale team, progetto o agente, e in quale provider. Token Custodian attribuisce ogni chiamata API a un'entità (team, progetto, app, cost center) creando un'unica vista cross-provider. Le chiamate non attribuite vengono segnalate per essere etichettate.

Il modello di governance si basa su **budget graduati con escalation policy**: invece di blocchi rigidi, il sistema implementa soglie progressive. Per esempio: al 50% del budget si genera un alert, al 70% si notifica il team FinOps, all'80% le richieste vengono redirezionate verso modelli più efficienti, al 100% si attiva un flusso di eccezioni e approvazioni. Questo approccio mantiene i developer produttivi mentre la spesa viene gestita con gradualità.

Per gli agenti autonomi, Token Custodian offre **limiti di contenimento**: massimo numero di step, durata massima di una run, e limite di crescita del contesto. Quando un limite viene raggiunto, la run può essere messa in pausa, il proprietario avvisato, o un ticket aperto automaticamente.

La struttura tecnica delle policy segue il pattern di Cloud Custodian: una risorsa (`ai.chat`), una modalità di enforcement (`chat-pre-call`), dei filtri (es. budget sopra il 90%) e delle azioni (es. routing verso un modello efficiente). Le policy vengono gestite in due modi: tramite interfaccia UI per i team FinOps, o come codice in Git per i platform team, garantendo review e audit trail.

Il sistema è vendor-neutral e supporta i principali provider: Anthropic, OpenAI, AWS Bedrock, Google Vertex e Azure AI. L'enforcement avviene in real-time, prima di ogni chiamata, anche all'interno di strumenti esistenti come Claude Code e Slack.

## Principi pratici

- Iniziare sempre dall'attribuzione (showback): è la base per budget e chargeback
- Usare soglie graduate per non bloccare i developer
- Mantenere le policy in Git per renderne il cambiamento tracciabile e auditabile
- Impostare limiti di step, durata e contesto sugli agenti autonomi per contenere i costi runaway
