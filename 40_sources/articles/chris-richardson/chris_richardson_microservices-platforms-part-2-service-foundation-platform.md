---
tags:
  - microservices
  - team-topologies
  - platform-engineering
  - microservice-chassis
  - service-template
feature:
type: article
author: Chris Richardson
source: https://microservices.io/post/architecture/2026/01/14/qconsf-microservices-platforms-part-2.html
date: 2026-05-03
---

# Microservices Platforms - part 2: Service Foundation Platform

## Sunto

Questo articolo è il secondo di una serie basata sul talk tenuto da Chris Richardson al QCon San Francisco 2025, intitolato "Microservices Platforms: When Team Topologies Meets Microservices Patterns". Approfondisce il primo dei sei layer della piattaforma: la **Service Foundation Platform**.

Il problema che la Service Foundation Platform risolve è fondamentale: ogni nuovo microservizio deve affrontare le stesse sfide ricorrenti — bootstrapping, configurazione, health check, sicurezza di base, logging strutturato, integrazione con il service mesh, esposizione di metriche. Se ogni team applicativo deve risolvere questi problemi da zero, il risultato è duplicazione, inconsistenza e lentezza nell'avvio dei nuovi servizi.

La Service Foundation Platform risponde a questo problema attraverso due strumenti complementari:

- Il **Microservice Chassis** è un framework (o insieme di librerie) che incapsula le preoccupazioni cross-cutting comuni a tutti i servizi: configurazione esternalizzata, tracing distribuito, health endpoint, gestione degli errori, client HTTP con retry e circuit breaker. I team applicativi ereditano queste capacità senza doverle implementare.

- Il **Service Template** è uno scaffolding pronto all'uso che genera la struttura iniziale di un nuovo servizio, già integrata con il chassis e le convenzioni organizzative. L'obiettivo è che un team possa creare un nuovo microservizio pronto al deploy in pochi minuti.

Applicando le **Team Topologies**, il team responsabile della Service Foundation Platform agisce come **Platform Team**, offrendo queste capacità in modalità self-service al team applicativo (stream-aligned team). Il contratto tra i due team è il chassis stesso: il Platform Team si fa carico dell'evoluzione e manutenzione, mentre il team applicativo si concentra sulla logica di dominio. Questo riduce drasticamente il carico cognitivo e accelera la creazione di nuovi servizi, abilitando il fast flow.
