---
tags:
  - cloud-security
  - zero-trust
  - kubernetes
  - infrastruttura-distribuita
  - intelligenza-artificiale
feature:
type: article
author: Alex Zenla
source: https://www.infoq.com/podcasts/new-blueprint-cloud-security
date: 2026-07-19
---

# A New Blueprint for Cloud Security

## Sunto

Questo episodio del podcast InfoQ vede Alex Zenla, CTO e co-fondatore di Edera, discutere di "spite-driven development": costruire software per risolvere frustrazioni tecniche genuine invece di accettare astrazioni difettose. La filosofia centrale è che l'architettura dovrebbe emergere dall'insoddisfazione tecnica reale, non dall'accettazione passiva di sistemi rotti. I developer dovrebbero sfidare le astrazioni dello status quo piuttosto che sovrapporre complessità su fondamenta deboli.

Il tema principale riguarda le vulnerabilità del kernel Linux nell'infrastruttura cloud moderna. Il design monolitico del kernel crea spazi di memoria condivisi che abilitano leak di dati tra processi. Le tecnologie di isolamento dei container (namespaces, cgroups) forniscono una protezione multi-tenancy insufficiente: una singola CVE del kernel compromette interi nodi Kubernetes, poiché il kernel è condiviso tra tutti i container sullo stesso nodo. Questo crea una superficie di attacco critica che la maggior parte dei team, operando solo ai livelli superiori dello stack, non vede né comprende.

L'infrastruttura AI-native introduce ulteriori sfide di sicurezza. Le GPU consumer (progettate per il rendering) vengono riutilizzate per i workload LLM — un mismatch architetturale fondamentale. La memoria GPU è completamente condivisa: il fault di un tenant può interrompere l'accesso GPU di tutti gli altri. L'architettura single-tenancy delle GPU crea problemi di sicurezza ed efficienza negli ambienti cloud multi-tenant. Google TPU e acceleratori custom, progettati appositamente per il machine learning, rappresentano approcci superiori nel lungo termine rispetto all'architettura GPU riadattata.

Un concetto chiave discusso è il problema della stratificazione dello stack: lo stack cloud-native contemporaneo contiene troppi livelli di astrazione — hardware → firmware → kernel → virtualizzazione → container → Kubernetes → applicazioni. La maggior parte dei developer opera solo ai livelli superiori di orchestrazione, perdendo visibilità sulle vulnerabilità critiche degli strati inferiori. La citazione emblematica: "Se il kernel Linux fa schifo, tutto il resto farà schifo". Zenla avverte anche contro la fiducia cieca nel codice generato dall'AI: i developer devono mantenere umiltà tecnica, verificare l'output AI, comprendere i sistemi sottostanti e debuggare quando l'AI fallisce.

L'episodio si conclude con una visione positiva del ruolo della regolamentazione: i framework europei per la sovranità dei dati e la cybersicurezza sono visti come driver positivi. Le aziende spesso hanno bisogno di "spinte" regolatorie per implementare pratiche di sicurezza adeguate, e i tecnologi dovrebbero perseguire proattivamente la sicurezza invece di trattarla come un onere di compliance. La sicurezza robusta differenzia i prodotti nel mercato e dovrebbe essere considerata un vantaggio competitivo, non un centro di costo.
