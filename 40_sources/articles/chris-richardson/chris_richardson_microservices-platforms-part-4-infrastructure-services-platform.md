---
tags:
  - microservices
  - team-topologies
  - platform-engineering
  - kubernetes
  - infrastructure
  - developer-experience
  - service-mesh
feature:
type: article
author: Chris Richardson
source: https://microservices.io/post/architecture/2026/02/12/qconsf-microservices-platforms-part-4.html
date: 2026-05-03
---

# Microservices Platforms - part 4: Infrastructure Services Platform

## Sunto

Questo articolo è il quarto di una serie basata sul talk tenuto da Chris Richardson al QCon San Francisco 2025, intitolato "Microservices Platforms: When Team Topologies Meets Microservices Patterns". Affronta il terzo layer della piattaforma: la **Infrastructure Services Platform**, il cui obiettivo è consentire ai team applicativi di consegnare valore rapidamente **senza dover diventare esperti di infrastruttura**.

Il problema centrale è il seguente: in un'architettura a microservizi, la complessità infrastrutturale è molto elevata. I team applicativi devono fare i conti con orchestrazione dei container (Kubernetes), service mesh, service discovery, load balancing, configurazione degli ambienti, gestione dei volumi, networking tra pod. Se ogni team deve acquisire questa expertise autonomamente, il carico cognitivo diventa insostenibile e il fast flow ne risente.

La **Infrastructure Services Platform** risolve questo problema astraendo la complessità infrastrutturale e offrendo capacità self-service ai team applicativi. I suoi componenti tipici includono:

- **Container orchestration** tramite Kubernetes, con configurazioni standardizzate e template di deployment già pronti (Helm chart, Kustomize overlay)
- **Service mesh** (es. Istio, Linkerd) che gestisce in modo trasparente il traffico inter-servizio: load balancing, retry, circuit breaking, mTLS
- **Service discovery** automatico, che elimina la necessità di configurare manualmente gli endpoint tra servizi
- **Infrastructure as Code** per la gestione riproducibile degli ambienti, in modo che sviluppo, staging e produzione siano consistenti
- **Developer portal** o CLI interne che espongono queste capacità con interfacce semplificate, pensate per i team applicativi

Applicando le **Team Topologies**, il Platform Team che gestisce questa piattaforma è responsabile di tenere aggiornata, sicura e funzionante tutta l'infrastruttura, mentre i team applicativi interagiscono con essa attraverso astrazioni di alto livello. Il risultato è che un team può fare il deploy di un nuovo servizio in Kubernetes senza conoscere i dettagli di networking o di configurazione del cluster, concentrandosi esclusivamente sulla logica di business.
