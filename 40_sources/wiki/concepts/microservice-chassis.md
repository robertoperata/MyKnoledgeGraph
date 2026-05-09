---
title: Microservice Chassis
type: concept
tags:
  - thread1-microservices
sources:
  - "[[chris_richardson_microservices-platforms-part-2-service-foundation-platform]]"
  - "[[chris_richardson_cubes-hexagons-triangles-yow2019]]"
updated: 2026-05-07
related:
  - "[[concepts/microservices-platform]]"
  - "[[concepts/hexagonal-architecture]]"
  - "[[concepts/team-topologies]]"
  - "[[concepts/twelve-factor]]"
  - "[[concepts/observability]]"
---

# Microservice Chassis

## Definizione

Framework (o insieme di librerie) che incapsula le **preoccupazioni cross-cutting** comuni a tutti i microservizi di un'organizzazione. Parte centrale della Service Foundation Platform. I team applicativi ereditano queste capacità senza doverle implementare da zero.

## Come funziona

Il chassis incapsula, tra le altre:

| Preoccupazione | Descrizione | Esempio |
|---|---|---|
| **Externalized Configuration** | Config iniettata dall'esterno, non hardcoded | Env vars, Spring Cloud Config, HashiCorp Vault |
| **Health Check** | Endpoint per liveness/readiness probe | `/actuator/health` (Spring Boot) |
| **Distributed Tracing** | Propagazione del trace ID tra servizi | OpenTelemetry, Sleuth/Zipkin |
| **Structured Logging** | Log su stdout in formato standard (JSON) | Logback/SLF4J → Loki/ELK |
| **Metrics Exposure** | Esposizione metriche per scraping | Prometheus endpoint `/actuator/prometheus` |
| **Retry & Circuit Breaker** | Resilienza nelle chiamate outbound | Resilience4j, Istio (in alternativa) |
| **mTLS / Service Identity** | Autenticazione inter-servizio | Istio sidecar o client con certificati |

## Service Template

Complementare al chassis: scaffolding che genera la struttura iniziale di un nuovo servizio già integrata con il chassis e le convenzioni organizzative.

```
service-template/
├── src/main/java/
│   ├── config/          # Config classes (usano chassis)
│   ├── api/             # Inbound adapters
│   ├── domain/          # Business logic (esagono)
│   └── infrastructure/  # Outbound adapters
├── Dockerfile           # Build standardizzato
├── .github/workflows/   # Pipeline CI/CD (usa Build Platform)
├── helm/                # Chart K8s pre-configurato
└── README.md
```

**Obiettivo**: un team può creare un nuovo microservizio pronto al deploy in **pochi minuti**, non giorni.

## Capacità operative (Richardson, YOW! 2019)

Richardson nell'Iceberg model descrive le "service operational capabilities" che ogni servizio deve avere in produzione. Il chassis le fornisce gratuitamente:

- Externalized Configuration
- Logging (stdout, formato strutturato)
- Health Check (Kubernetes liveness/readiness)
- Telemetry/Metrics (Prometheus)
- Distributed Tracing (trace ID propagato)

## Evoluzione del chassis

Il Platform Team si fa carico dell'evoluzione e manutenzione del chassis. I team applicativi aggiornano la versione del chassis come una dipendenza. Questo crea un punto centralizzato per:
- Aggiornamenti di sicurezza (es. vulnerabilità in una libreria)
- Cambio del provider di tracing
- Aggiornamento alle nuove versioni del service mesh

> **Tensione:** se il chassis evolve in modo non retro-compatibile, tutti i servizi che lo usano devono aggiornarsi — crea un accoppiamento soft tra tutti i servizi. Soluzione: semantic versioning rigoroso, periodo di deprecazione, contratto esplicito.

## Connessioni

- [[concepts/hexagonal-architecture]] — il chassis implementa le preoccupazioni che circondano l'esagono
- [[concepts/microservices-platform]] — il chassis è il deliverable principale della Service Foundation Platform
- [[concepts/twelve-factor]] — il chassis implementa nativamente i principi 3 (Config), 11 (Logs), 12 (Admin processes)
- [[concepts/observability]] — logging strutturato e tracing distribuito sono parte del chassis
- [[concepts/team-topologies]] — Platform Team costruisce il chassis, Stream-aligned Team lo consuma
