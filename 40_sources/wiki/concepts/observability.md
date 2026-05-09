---
title: Observability nei Microservizi
type: concept
tags:
  - thread1-microservices
  - thread3-cloud
sources:
  - "[[chris_richardson_microservices-platforms-part-5-observability-platform]]"
  - "[[chris_richardson_cubes-hexagons-triangles-yow2019]]"
updated: 2026-05-07
related:
  - "[[concepts/microservices-platform]]"
  - "[[concepts/microservice-chassis]]"
  - "[[technologies/kubernetes]]"
  - "[[technologies/istio]]"
---

# Observability nei Microservizi

## Definizione

La capacità di comprendere lo stato interno di un sistema distribuito a partire dai suoi output (logs, metriche, trace). Nei microservizi, l'osservabilità è essenziale perché non è più possibile usare un debugger locale o una singola dashboard — le richieste attraversano decine di servizi.

## I tre pilastri (Three Pillars of Observability)

| Pilastro | Cosa misura | Tool tipici |
|---|---|---|
| **Metrics** | Aggregati numerici nel tempo (contatori, gauge, istogrammi) | Prometheus, Grafana |
| **Logs** | Record di eventi testuali strutturati | ELK stack (Elasticsearch, Logstash, Kibana), Loki |
| **Distributed Tracing** | Percorso di una richiesta attraverso più servizi | Jaeger, Zipkin, OpenTelemetry |

## RED Metrics

Metriche fondamentali per ogni servizio (approccio RED):
- **R**ate: quante richieste al secondo riceve il servizio
- **E**rrors: percentuale di richieste che falliscono
- **D**uration: quanto tempo impiegano le richieste (latenza)

Le dashboard Grafana pre-costruite sulla base RED consentono visibilità immediata su ogni servizio.

## Come funziona (Observability Platform)

```
                  Observability Platform
┌─────────────────────────────────────────────────────────┐
│  Prometheus ← (scraping) ← metrics endpoint (chassis)  │
│  Loki      ← (push)     ← structured logs (chassis)    │
│  Jaeger    ← (push)     ← trace spans (chassis/mesh)   │
│                                                         │
│  Grafana → dashboard pre-costruite (RED metrics)        │
│  AlertManager → policy di alerting centralizzate        │
└─────────────────────────────────────────────────────────┘
```

Il [[concepts/microservice-chassis]] garantisce che ogni servizio esporti logs strutturati, metrics e trace in modo uniforme — la piattaforma raccoglie il resto senza configurazione aggiuntiva.

## Responsabilità divisa (platform vs team applicativo)

La Observability Platform **non** esime i team applicativi da ogni responsabilità:

| Responsabilità | Chi la tiene |
|---|---|
| Infrastruttura di raccolta (Prometheus, Jaeger...) | Platform Team |
| Dashboard standard (RED metrics) | Platform Team |
| Span di tracing significativi per il dominio | Team applicativo |
| Metriche di business domain-specific | Team applicativo |
| Log utili per il debugging | Team applicativo |

La piattaforma fornisce l'infrastruttura; il team fornisce il **contesto**.

## Distributed Tracing

Il trace ID viene generato al punto di ingresso (API Gateway) e propagato in ogni chiamata inter-servizio tramite header HTTP (es. `X-B3-TraceId`). Il [[concepts/microservice-chassis]] inietta automaticamente il propagatore nel client HTTP outbound.

```
Client → API Gateway (trace_id=abc)
  → Order Service (trace_id=abc, span_id=1)
    → Customer Service (trace_id=abc, span_id=2)
      → Order DB (trace_id=abc, span_id=3)
```

Jaeger/Zipkin visualizza questo come un grafo (Gantt chart) che mostra dove si passa il tempo.

## Canary Deployment e Monitoring

Richardson (YOW! 2019) descrive la separazione tra **deployment** (codice in produzione) e **release** (traffico instradato):

```
v1 ── 100% traffico
v2 (deployed) → test interni → 5% traffico → 20% → 100%
                                    ↕
                    monitoring automatico (latency, 500s)
                                    ↕
                    rollback automatico se soglie superate
```

L'Observability Platform alimenta il monitoring automatico che decide il rollback.

## Connessioni

- [[concepts/microservice-chassis]] — il chassis implementa l'instrumentazione standard (logging, metrics, tracing)
- [[concepts/microservices-platform]] — l'Observability Platform è il quarto layer
- [[technologies/istio]] — il service mesh aggiunge metriche di rete e tracing automatici senza modifiche al codice
- [[technologies/kubernetes]] — K8s liveness/readiness probe si integrano con la piattaforma di alerting
