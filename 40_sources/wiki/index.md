# Index — Tech Knowledge Wiki

## Concetti

### Thread 1 — Architettura distribuita e microservizi
- [[concepts/independent-deployability]] — Deployare senza coordinamento con altri servizi [thread1]
- [[concepts/information-hiding]] — Nascondere ciò che cambia dietro un confine stabile [thread1]
- [[concepts/bounded-context]] — Confine di dominio con lingua condivisa interna [thread1, thread5]
- [[concepts/ubiquitous-language]] — Lingua condivisa tra tecnici e business [thread1, thread5]
- [[concepts/twelve-factor]] — 12 principi per app cloud-native [thread1, thread3]
- [[concepts/architectural-governance]] — Governance dichiarativa a velocità AI [thread1, thread4]
- [[concepts/legacy-modernization]] — Evoluzione incrementale di sistemi legacy [thread1]
- [[concepts/async-workflow-patterns]] — Modello di maturità per workflow asincroni [thread1]
- [[concepts/hexagonal-architecture]] — Ports & Adapters: business logic al centro, porte e adapter [thread1]
- [[concepts/scalability-cube]] — X/Y/Z axis: la Y-axis è la microservice architecture [thread1]
- [[concepts/team-topologies]] — Platform Team vs Stream-aligned Team per il fast flow [thread1]
- [[concepts/microservices-platform]] — I 6 layer di piattaforma per abilitare il fast flow [thread1, thread3]
- [[concepts/microservice-chassis]] — Framework cross-cutting per ogni servizio [thread1]
- [[concepts/observability]] — Metrics/logs/traces: osservabilità distribuita nei microservizi [thread1, thread3]
- [[concepts/sync-async-hybrid-architecture]] — REST ai confini + async interno; queue ownership; anti-pattern catene sincrone [thread1]
- [[concepts/dual-write-model]] — Scrivere in event log + stato corrente nella stessa operazione; alternativa pragmatica all'event sourcing puro [thread1, thread5]

### Thread 2 — Java ecosystem
- [[concepts/jpa-hibernate-performance]] — Connection management, entity anti-patterns, projections; open-in-view, TransactionTemplate, N+1 [thread2]
- [[concepts/spring-transactional]] — @Transactional: proxy mode, self-invocation, rollback su checked exception, readOnly hint, REQUIRES_NEW, annotazioni composite [thread2]
- [[concepts/java-concurrency]] — Threading, Executor Framework, concurrent collections [thread2]
- [[concepts/virtual-threads]] — Thread JVM leggeri per workload IO-intensive (Java 21+); pinning synchronized risolto in Java 24 [thread2]
- [[concepts/completable-future]] — Composizione di operazioni asincrone [thread2]
- [[concepts/java-memory-model]] — Garanzie di visibilità tra thread (JSR 133) [thread2]
- [[concepts/garbage-collection]] — GC tuning: safepoints, TLAB, G1, ZGC; Compact Object Headers Java 25 [thread2]
- [[concepts/structured-concurrency]] — Structured Concurrency: unità di lavoro concorrente (5ª preview Java 25; attesa Java 29) [thread2]
- [[concepts/scope-values]] — Sostituzione immutabile di ThreadLocal; finale in Java 25 [thread2]

### Thread 3 — Cloud e infrastruttura AWS

- [[concepts/aws-vpc-fundamentals]] — VPC, subnet, route table, IGW, NAT Gateway [thread3, aws]
- [[concepts/aws-security-groups-nacls]] — SG (stateful, allow-only) vs NACL (stateless, allow+deny), micro-segmentation [thread3, aws]
- [[concepts/aws-vpc-connectivity]] — VPC Peering, Transit Gateway, Direct Connect, VPN, VPC Endpoints, data transfer charges [thread3, aws]
- [[concepts/aws-ddos-protection]] — Shield, WAF, CloudFront, Route 53, defense in depth, auto scaling anti-DDoS [thread3, aws]

### Thread 4 — AI e agenti
- [[concepts/agentic-patterns]] — Pattern per l'uso efficace di agenti AI nel coding; workflow human-in-the-loop vs AFK [thread4]
- [[concepts/context-management]] — CDLC: contesto come codice; smart zone/dumb zone; clear vs compact [thread4]
- [[concepts/spec-driven-development]] — SDD: specifiche eseguibili come source of truth [thread4]
- [[concepts/harness-engineering]] — Sistemi di controllo intorno agli agenti AI [thread4]
- [[concepts/autonomous-systems-architecture]] — Architettura a confini per sistemi autonomi multi-agent [thread4]
- [[concepts/deep-modules]] — Deep vs shallow modules: interfaccia piccola, molta logica interna; AI-friendly codebase [thread4, thread1]

### Thread 5 — DDD
- [[concepts/aggregate]] — Unità transazionale nel domain model [thread5]
- [[concepts/domain-event]] — Fatto di dominio accaduto, immutabile [thread5]
- [[concepts/value-object]] — Oggetto definito dai valori, senza identità [thread5]

---

## Pattern

- [[patterns/event-driven]] — Comunicazione disaccoppiata tramite eventi [thread1, thread5]
- [[patterns/request-reply-correlation-id]] — Request/response async via Kafka con correlation ID [thread1, thread2]
- [[patterns/cqrs-read-model]] — Read model locale denormalizzato alimentato da eventi [thread1, thread5]
- [[patterns/saga-pattern]] — Transazioni distribuite multi-servizio con compensazione [thread1, thread5]
- [[patterns/api-composition-bff]] — Aggregazione dati per client tramite livello intermedio [thread1]
- [[patterns/multi-cloud-event-driven]] — EDA cross-cloud con framework DEPOSITS [thread1, thread3]
- [[patterns/kafka-consumer-proxy]] — Proxy centralizzato per consumer Kafka a scala [thread1]
- [[patterns/transactional-outbox]] — Atomicità DB + pubblicazione messaggio senza 2PC [thread1]
- [[patterns/testing-pyramid]] — Strategia di testing per microservizi: unit→contract→component→E2E [thread1]
- [[patterns/anti-corruption-layer]] — Façade/adapter tra sistemi con semantiche diverse per proteggere il modello interno [thread1, thread5]
- [[patterns/modular-monolith]] — Deployment unico + architettura logica modulare guidata da DDD; DB as Queue; spettro di isolamento [thread1, thread5]

---

## Tecnologie

- [[technologies/kafka]] — Message broker per event-driven e async communication [thread1]
- [[technologies/kubernetes]] — Orchestratore container, networking, service types [thread3, thread1]
- [[technologies/istio]] — Service mesh per traffic management, mTLS, observability [thread3]
- [[technologies/keycloak]] — Fine-grained authorization con UMA 2.0 / OAuth2 [thread6]
- [[technologies/calm]] — Architecture as Code per sicurezza by-design [thread6, thread3]
- [[technologies/vector-databases]] — Storage e ricerca di embeddings per RAG e AI [thread4, thread3]
- [[technologies/gitops]] — Git come source of truth per deployment dichiarativo [thread1, thread3]

---

## Sintesi trasversali

- [[synthesis/microservizi-java-aggregazione]] — CQRS + Request-Reply in sistemi Java ad alta concorrenza (2026-04-09)

---

## Sorgenti ingested

### Articles
- `articles/chris-richardson/chris_richardson_cubes-hexagons-triangles-yow2019.md` — Hexagonal arch, Scale Cube, Testing Pyramid, canary deployment (YOW! 2019)
- `articles/chris-richardson/chris_richardson_not-just-events-async-microservices-goto2019.md` — Saga, CQRS, Transactional Outbox, Event Sourcing (GOTO 2019)
- `articles/chris-richardson/chris_richardson_microservices-platforms-part-1-overview.md` — Microservices Platforms: overview dei 6 layer
- `articles/chris-richardson/chris_richardson_microservices-platforms-part-2-service-foundation-platform.md` — Service Foundation: chassis + service template
- `articles/chris-richardson/chris_richardson_microservices-platforms-part-3-security-platform.md` — Security Platform: IdP, mTLS, secrets
- `articles/chris-richardson/chris_richardson_microservices-platforms-part-4-infrastructure-services-platform.md` — Infrastructure Platform: K8s, service mesh, IaC
- `articles/chris-richardson/chris_richardson_microservices-platforms-part-5-observability-platform.md` — Observability Platform: metrics, logs, tracing
- `articles/chris-richardson/chris_richardson_microservices-platforms-part-6-build-platform.md` — Build Platform: CI/CD pipeline, reusable workflows
- `articles/chris-richardson/chris_richardson_microservices-platforms-part-7-deployment-platform.md` — Deployment Platform: K8s, GitOps, ambienti
- `articles/infoq_architectural-governance-ai-speed.md` — Governance architetturale a velocità AI
- `articles/vercel_agent-responsibly.md` — Agenti responsabili, ownership del deployment
- `articles/tessl_context-development-lifecycle.md` — CDLC: contesto come codice
- `articles/como_preparation-collaborative-modeling-workshops.md` — Preparazione workshop collaborative modeling
- `articles/dear_architect-50k-user-journeys-async-workflows.md` — Modello di maturità workflow asincroni
- `articles/dear_architect-harness-engineering.md` — Harness engineering per coding agent
- `articles/dear_architect-multi-agent-architecture-smarter-advertising.md` — Multi-agent Spotify Ads
- `articles/dear_architect-postgres-optimization-treadmill.md` — PostgreSQL architectural mismatch per dati live
- `articles/dear_architect-redefining-architecture-boundaries-matter-most.md` — Architettura a confini per sistemi autonomi
- `articles/devoxx_modern-java-playful-way.md` — Java 21-25: Records, Structured Concurrency, Stream Gatherers
- `articles/infoq_aws-s3-vectors-ga-storage-first-rag.md` — S3 Vectors GA, storage-first per RAG
- `articles/infoq_enterprise-spec-driven-development-adoption.md` — SDD a scala enterprise
- `articles/infoq_multi-cloud-event-driven-architectures.md` — EDA cross-cloud con framework DEPOSITS
- `articles/infoq_netflix-graph-abstraction-650tb-milliseconds.md` — Graph Abstraction Netflix 650TB
- `articles/infoq_secure-connectivity-api-calm.md` — CALM: architettura sicura come codice
- `articles/infoq_software-evolution-microservices-genai.md` — Legacy modernization e GenAI (Chris Richardson)
- `articles/infoq_spec-driven-development-architecture-executable.md` — SDD: specifiche eseguibili
- `articles/infoq_uber-uforwarder-kafka-push-proxy.md` — uForwarder: Kafka consumer proxy Uber
- `articles/microsoft_anti-corruption-layer-pattern.md` — Anti-Corruption Layer pattern (Microsoft Azure Architecture Center)
- `articles/sookocheff_restful-http-async-event-driven-services.md` — REST ai confini + async interno; queue vs event stream; Christmas Tree anti-pattern
- `articles/youtube_majestic-modular-monolith-axel-fontaine.md` — Modular Monolith: spettro isolamento, DDD, DB as queue, enforcement architetturale (Devoxx 2018)
- `articles/youtube_building-bank-backend-revolut-yatsenko.md` — Revolut: MVP→event-driven monolith→services; dual-write; fraud pre-computation; @Transactional pitfall (Devoxx 2019)
- `articles/martinfowler_sdd-tools-kiro-spec-kit-tessl.md` — SDD: tassonomia 3 livelli, Kiro/spec-kit/Tessl, spec vs memory bank, parallelo MDD (Böckeler 2025)
- `articles/youtube_kiro-spec-driven-development-al-harris.md` — Kiro: EARS format, PBT dai requisiti, neurosymbolic backend, spec come living docs, incremental disclosure (AI Engineer 2026)
- `articles/youtube_java-25-lts-features-jchampions.md` — Java 25 LTS: Compact Object Headers, AOT, virtual thread pinning fix, Scope Values, Stream Gatherers, compact source files (JChampions 2026)
- `articles/getkafkanated_kip-881-kip-392-inter-az-kafka-costs.md` — KIP-392 (Fetch From Follower) + KIP-881 (Rack-Aware Assignment): ~50% riduzione costi inter-AZ Kafka su cloud
- `articles/youtube_spring-data-jpa-hibernate-performance-walkowiak.md` — Spring Data JPA & Hibernate: connection management, N+1, entity anti-patterns, projections (Devoxx 2024)
- `articles/youtube_ai-coding-workflow-matt-pocock.md` — Workflow AI coding: smart zone, grilling session, PRD, Kanban DAG, AFK loop, TDD, deep modules, human-in-loop vs AFK (Matt Pocock)
- `articles/spring_transactional-declarative-annotations.md` — @Transactional Spring: proxy mode, self-invocation, rollback rules, readOnly, propagation, annotazioni composite, reactive (Spring docs)

### Courses
- `courses/Microservices Fundamental.md` — Microservizi, DDD, independent deployability (Sam Newman)
- `courses/Microservices Collaboration.md` — Stili di comunicazione, saga, temporal coupling (Sam Newman)
- `courses/Managing Microservices with Kubernetes and Istio.md` — Container, Kubernetes, Istio
- `courses/Java Threads Basics.md` — Threading Java, Executor, CompletableFuture, Virtual Threads
- `courses/Java Threads Demystified.md` — Thread safety, JMM, concurrent utilities (Maurice Naftalin)
- `courses/Concurrent Programming Core Concepts.md` — Concetti fondamentali di concorrenza (note sparse)
- `courses/Java GC Tuning.md` — GC tuning: safepoints, TLAB, G1, ZGC (Kirk Pepperdine)
- `courses/High-Performance Java Persistence.md` — USL, response time, throughput (Vlad Mihalcea, note sparse)
- `courses/Java next Steps Streams Collectors.md` — Streams e Collectors (solo PDF allegato)
- `courses/Refactoring to Functional Programming in Java.md` — Refactoring funzionale (Victor Rentea, note sparse)
- `courses/Domain-Driven Design Aggregates Domain Events and Value Objects.md` — DDD tattico (Vaughn Vernon, note sparse)
- `courses/KodeKloud AWS Solutions Architect Associate Certification.md` — VPC, subnet, NAT, routing AWS *(aggiornato 2026-05-03)*
- `courses/hands-on-with-aws-vpcs/knowledge.md` — VPC completo: peering, Transit Gateway, ELB, Direct Connect, VPN, VPC Endpoints, Flow Logs, IaC
- `courses/aws-security-deep-dive-vpcs/knowledge.md` — VPC security: SG vs NACL, bastion host, WAF, Shield, CloudFront, DDoS mitigation
- `courses/Certified Kubernetes Application Developer.md` — K8s networking, Services, NetworkPolicy, Istio
- `courses/DevOps Pre-Requisite Course.md` — Networking Linux: ip link, ip addr, routing
- `courses/AI Talks - Lada Kesseler Augmented Coding.md` — Pattern per augmented coding con AI
- `courses/Claude Code and Large-Context Reasoning.md` — Context window, MCP, agents (Tim Warner)
- `courses/Kotlin and Spring.md` — Type system Kotlin: Unit, Nothing, Any, nullable (Ken Kousen, note sparse)
- `courses/Rust in motion.md` — Basics Rust: variabili, tipi, Cargo
- `courses/Graph Data Structures and Algorithms from Scratch.md` — Grafi, BFS/DFS, Dijkstra, topological sort
- `courses/The Essential Learning Foundations.md` — Linear algebra, tensors (ML foundations)
- `courses/twelve-factor application.md` — 12 fattori per app cloud-native

### Blog Posts
- `070_blog_posts/Architecture/microservizi_pattern_summary.md` — Pattern aggregazione microservizi + implementazioni Java
- `070_blog_posts/keycloak/Keycloak Authorization Concept.md` — Keycloak authorization: concetti e classi Java
- `070_blog_posts/keycloak/keycloak authorization concept blog post.md` — Guida pratica Keycloak fine-grained authorization

---

## Note su sorgenti con contenuto limitato

Le seguenti sorgenti hanno note molto sparse o puntano principalmente a file PDF allegati. Sono da re-ingesting quando le note saranno ampliate:
- `High-Performance Java Persistence.md` — solo USL/formule, nessuna nota sui pattern Hibernate/JPA
- `Java next Steps Streams Collectors.md` — solo PDF allegato, nessun testo
- `Refactoring to Functional Programming in Java.md` — solo intestazione
- `Domain-Driven Design Aggregates...md` — solo domande e PDF allegato
- `Concurrent Programming Core Concepts.md` — solo motivazioni iniziali

## Verifica file "Agentic Coding with Claude Code.md"

Il file citato in CLAUDE.md (`courses/Agentic Coding with Claude Code.md`) **non esiste** nella directory `courses/`. Il corso AWS (`KodeKloud AWS Solutions Architect Associate Certification.md`) ha contenuto corretto (VPC, subnet). Il corso sui pattern agentic è coperto da:
- `courses/AI Talks - Lada Kesseler Augmented Coding.md`
- `courses/Claude Code and Large-Context Reasoning.md`

---

## Lacune tematiche identificate

Vedi [[lacune-identificate]] per il dettaglio completo.

---

## Statistiche
- Totale pagine wiki: 61 (33 concepts, 11 patterns, 7 technologies, 1 synthesis, 1 index, 1 log, 1 lacune) *(+1 da ingest 2026-05-09)*
- Sorgenti ingested: 68 (42 articles + 23 courses + 3 blog posts)
- Ultimo aggiornamento: 2026-05-09
