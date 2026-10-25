---
tags:
  - microservices
  - agentic-ai
  - distributed-systems
  - software-architecture
  - llm-orchestration
feature:
type: article
author: Mallika Rao
source: https://www.infoq.com/articles/distributed-systems-microservices-agentic-ai/
date: 2026-10-25
---

# From Microservices to Agents: the Next Evolution of Distributed Systems

## Sunto

I sistemi distribuiti stanno attraversando una nuova fase di evoluzione: se i microservizi hanno risolto il problema della **decomposizione dei servizi**, i sistemi agentici stanno ora affrontando la **decomposizione delle decisioni**. Mentre nei microservizi il flusso di controllo è definito staticamente (un servizio chiama un altro in modo deterministico), nei sistemi agentici il percorso di esecuzione viene scelto a runtime da agenti LLM che selezionano strumenti, fonti dati e strategie in modo dinamico. Questo rende il comportamento del sistema meno prevedibile e introduce sfide architetturali del tutto nuove.

La gestione dello stato è uno degli aspetti più critici: i sistemi agentici devono mantenere coerente un contesto distribuito che include piani intermedi, documenti recuperati e output degli strumenti. Si tratta di un problema simile alla coordinazione dello stato distribuito già affrontato dai microservizi, ma con la complessità aggiuntiva che lo stato è semantico e non solo transazionale.

I fallimenti nei sistemi agentici si manifestano spesso non come errori espliciti ma come **degradazione della qualità decisionale**. Tre pattern di guasto ricorrenti sono: il *cascading decision degradation* (una decisione errata si propaga attraverso la catena di agenti amplificando l'errore), il *retry amplification* (i loop di valutazione generano un numero esplosivo di richieste), e il *silent quality decline* (il sistema risponde ma con qualità progressivamente peggiore senza generare alert).

Per quanto riguarda i pattern architetturali, l'autrice identifica quattro approcci chiave: separare la fase di pianificazione dalla fase di esecuzione, specializzare gli agenti per domini specifici, integrare cicli di valutazione nell'esecuzione stessa, e costruire layer di astrazione sugli strumenti. Questi pattern riflettono i principi già applicati nei microservizi (separazione delle responsabilità, single responsibility, interfacce ben definite), estesi al dominio del decision-making.

L'osservabilità per i sistemi agentici richiede il **tracing a livello di decisione** e il monitoraggio della deriva comportamentale, non solo la raccolta di metriche tecniche tradizionali. Le pratiche di reliability includono: autonomia delimitata (latency budget, limiti di retry e di profondità), fallback deterministici, e rollout progressivo. L'autrice conclude che i principi che hanno reso scalabili i microservizi (confini chiari, osservabilità, ingegneria della reliability, platform thinking) si applicano ancora, estesi a questo nuovo livello di decision-making distribuito.

## Immagini

![Modello di esecuzione: Microservizi vs Sistemi Agentici](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/distributed-systems-microservices-agentic-ai/en/resources/249figure-1-1792138502664.jpg)

![Architettura di un sistema agentico](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/distributed-systems-microservices-agentic-ai/en/resources/186figure-2-1792138502664.jpg)

![Modalità di fallimento silenzioso nei sistemi agentici](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/distributed-systems-microservices-agentic-ai/en/resources/153figure-3-1792138502664.jpg)
