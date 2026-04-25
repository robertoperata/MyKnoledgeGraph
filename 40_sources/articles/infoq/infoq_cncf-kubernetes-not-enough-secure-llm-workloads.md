---
tags:
  - kubernetes
  - llm-security
  - cncf
  - ai-workloads
  - threat-model
feature:
type: article
author: Craig Risi
source: https://www.infoq.com/news/2026/04/cncf-kubernetes-llm-security/
date: 2026-04-25
---

# CNCF Warns Kubernetes Alone Is Not Enough to Secure LLM Workloads

## Sunto

La Cloud Native Computing Foundation ha pubblicato un'analisi che evidenzia un divario critico nel modo in cui le organizzazioni stanno deployando Large Language Model su Kubernetes: sebbene Kubernetes eccella nell'orchestrazione e nell'isolamento dei workload containerizzati, non è in grado di monitorare o controllare il comportamento intrinseco dei sistemi AI, creando un threat model fondamentalmente diverso e più complesso rispetto alle applicazioni tradizionali.

Il problema core è che gli LLM operano su input non fidati e prendono decisioni dinamiche basate sul contesto semantico — qualcosa che le funzionalità native di Kubernetes come RBAC, network policy, e health check non sono progettate per governare. Kubernetes fornisce visibilità a livello infrastrutturale (CPU, memoria, pod health), ma rimane cieco rispetto a domande cruciali: il prompt inviato al modello è malevolo? Il modello sta esponendo dati sensibili attraverso le proprie risposte? Sta violando policy di sicurezza contestuali?

Il documento del CNCF introduce una distinzione fondamentale: i modelli LLM devono essere trattati come **entità programmabili e decisionali**, non semplicemente come workload di calcolo. Quando i modelli ottengono accesso a tool interni, API, e credenziali — come avviene nelle architetture agentiche — si aprono vettori di attacco nuovi, tra cui prompt injection, jailbreaking, e disclosure non autorizzata di dati, per i quali le contromisure tradizionali di Kubernetes sono insufficienti.

La raccomandazione del CNCF è un approccio multi-strato: le feature di Kubernetes (RBAC, network policies) rimangono necessarie ma non sufficienti. Le organizzazioni devono aggiungere controlli a livello applicativo, inclusi validazione dei prompt in input, filtraggio degli output, restrizioni granulari sull'accesso ai tool, e allineamento con framework come l'OWASP Top 10 for LLMs. Il monitoraggio runtime con supervisione umana e policy enforcement rigorose attorno alle capability dei sistemi AI completano il quadro di sicurezza consigliato.

Questo articolo evidenzia come la convergenza tra Kubernetes ecosystem e AI stia richiedendo una rivisitazione profonda dei modelli di sicurezza cloud-native: le best practice che hanno funzionato per i microservizi tradizionali non sono direttamente applicabili ai sistemi AI agentici, e l'industria si trova a dover costruire nuovi framework di governance specifici per questo dominio.
