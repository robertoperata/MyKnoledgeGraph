---
tags:
  - microservices
  - team-topologies
  - platform-engineering
  - observability
  - monitoring
  - metrics
  - distributed-tracing
  - logging
feature:
type: article
author: Chris Richardson
source: https://microservices.io/post/architecture/2026/02/24/qconsf-microservices-platforms-part-5.html
date: 2026-05-03
---

# Microservices Platforms - part 5: Observability Platform

## Sunto

Questo articolo è il quinto di una serie basata sul talk tenuto da Chris Richardson al QCon San Francisco 2025, intitolato "Microservices Platforms: When Team Topologies Meets Microservices Patterns". Descrive il quarto layer della piattaforma: la **Observability Platform**, il cui scopo è rendere i microservizi **osservabili out of the box**.

Il punto di partenza è una verità fondamentale: i microservizi sono gestibili solo se i team riescono a capire cosa stanno facendo i propri servizi in produzione. Costruire osservabilità — metriche, log strutturati, trace distribuiti, dashboard e alerting — è però complesso e time-consuming. Se ogni team deve costruire questa capacità da zero, il progresso rallenta e i servizi diventano inconsistenti tra loro, rendendo difficile correlare eventi attraverso il sistema distribuito.

La **Observability Platform** risolve questo problema centralizzando l'infrastruttura di osservabilità e offrendola come capacità condivisa ai team applicativi. Comprende tipicamente:

- **Raccolta centralizzata di metriche** (es. Prometheus) con esportatori pre-configurati e regole di alerting condivise
- **Aggregazione e ricerca dei log** (es. ELK stack, Loki) con un formato di log strutturato imposto dal service chassis, che garantisce consistenza tra tutti i servizi
- **Distributed tracing** (es. Jaeger, Zipkin) per correlare le richieste attraverso più servizi e identificare colli di bottiglia
- **Dashboard pre-costruite** (es. Grafana) che i team possono usare immediatamente senza configurazione, con viste standard su latenza, error rate e throughput (i cosiddetti RED metrics)
- **Alerting** centralizzato con policy condivise

Un aspetto importante sottolineato da Richardson è che l'Observability Platform non esime i team applicativi da ogni responsabilità: i team devono comunque **strumentare i propri servizi** — aggiungere span significativi al tracing, emettere metriche di business domain-specific, scrivere log utili per il debugging. La piattaforma fornisce l'infrastruttura; il team applicativo fornisce il contesto.

Applicando le **Team Topologies**, il Platform Team che gestisce l'Observability Platform riduce drasticamente il carico cognitivo dei team applicativi, che possono focalizzarsi sulla logica di dominio anziché costruire e operare infrastruttura di osservabilità. Il risultato è un sistema in cui ogni nuovo servizio è osservabile by default dal momento del primo deploy.
