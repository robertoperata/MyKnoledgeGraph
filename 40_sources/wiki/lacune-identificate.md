# Lacune Identificate

Argomenti non ancora coperti dalla wiki. Aggiornato: 2026-04-09.

---

## Tipo 1 — Sorgenti presenti ma note quasi vuote

Il file esiste in `40_sources/courses/` ma contiene solo titolo + PDF allegato. Quando le note saranno scritte, re-ingestire.

| File sorgente | Argomento mancante |
|---|---|
| `Java next Steps Streams Collectors.md` | Stream collectors avanzati, collectors custom, Collectors factory methods |
| `Refactoring to Functional Programming in Java.md` | Refactoring OOP → FP, pattern funzionali in Java (Victor Rentea) |
| `Domain-Driven Design Aggregates Domain Events and Value Objects.md` | Strategic modeling, tactical modeling DDD in dettaglio (Vaughn Vernon) |
| `High-Performance Java Persistence.md` | Pattern Hibernate/JPA, connection pooling, N+1 problem, batch inserts, query optimization |
| `Concurrent Programming Core Concepts.md` | Concorrenza: concetti avanzati oltre le basi |

---

## Tipo 2 — Argomenti citati ma senza sorgente dedicata

Emergono di passaggio nelle sorgenti esistenti, ma non c'è materiale sufficiente per costruire una pagina wiki utile. Sono candidati per nuove acquisizioni.

### Kafka Streams / KTable
**Dove emerge:** `The Essential Learning Foundations.md` pone la domanda: "in un microservizio con stream application che fa join di più topic su una KTable, quando scala up, tutte le istanze leggono da tutte le partizioni?" — ma nessuna sorgente risponde.
**Lacuna:** Kafka Streams topology, partitioning, stateful operations, KTable vs KStream.

### Testing di microservizi
**Dove emerge:** `Microservices Fundamental` menziona che "cross-service testing is more involved" — ma si ferma lì.
**Lacuna:** contract testing (Pact), consumer-driven contracts, test pyramid per microservizi, integration testing con Testcontainers.

### Osservabilità
**Dove emerge:** `Microservices Fundamental` cita "monitoring is more complex (open telemetry)" e la necessità di monitorare metriche di business oltre a quelle di sistema.
**Lacuna:** OpenTelemetry, distributed tracing, log aggregation, metriche applicative, alerting.

### CI/CD avanzato / GitOps
**Dove emerge:** `Managing Microservices with Kubernetes and Istio` elenca GitOps tra i temi ("why using GitOps") ma le note sono quasi assenti (il contenuto è nel PDF allegato).
**Lacuna:** GitOps, ArgoCD/Flux, pipeline multi-stage, deployment strategies (blue/green, canary) via CI.

### Kotlin avanzato
**Dove emerge:** `Kotlin and Spring.md` ha solo note sul type system base (Unit, Nothing, Any, nullable).
**Lacuna:** Coroutines, extension functions, sealed classes, Kotlin con Spring Boot in produzione.
