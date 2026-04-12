# Index — Tech Knowledge Wiki

## Concetti

### Thread 1 — Architettura distribuita e microservizi
- [[concepts/independent-deployability]] — Deployare senza coordinamento con altri servizi [thread1]
- [[concepts/information-hiding]] — Nascondere ciò che cambia dietro un confine stabile [thread1]
- [[concepts/bounded-context]] — Confine di dominio con lingua condivisa interna [thread1, thread5]
- [[concepts/ubiquitous-language]] — Lingua condivisa tra tecnici e business [thread1, thread5]
- [[concepts/twelve-factor]] — 12 principi per app cloud-native [thread1, thread3]
- [[concepts/architectural-governance]] — Governance dichiarativa a velocità AI [thread1, thread4]

### Thread 2 — Java ecosystem
- [[concepts/java-concurrency]] — Threading, Executor Framework, concurrent collections [thread2]
- [[concepts/virtual-threads]] — Thread JVM leggeri per workload IO-intensive (Java 21+) [thread2]
- [[concepts/completable-future]] — Composizione di operazioni asincrone [thread2]
- [[concepts/java-memory-model]] — Garanzie di visibilità tra thread (JSR 133) [thread2]
- [[concepts/garbage-collection]] — GC tuning: safepoints, TLAB, G1, ZGC [thread2]

### Thread 4 — AI e agenti
- [[concepts/agentic-patterns]] — Pattern per l'uso efficace di agenti AI nel coding [thread4]
- [[concepts/context-management]] — CDLC: contesto come codice da versionare e testare [thread4]

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

---

## Tecnologie

- [[technologies/kafka]] — Message broker per event-driven e async communication [thread1]
- [[technologies/kubernetes]] — Orchestratore container, networking, service types [thread3]
- [[technologies/istio]] — Service mesh per traffic management, mTLS, observability [thread3]
- [[technologies/keycloak]] — Fine-grained authorization con UMA 2.0 / OAuth2 [thread6]

---

## Sintesi trasversali

- [[synthesis/microservizi-java-aggregazione]] — CQRS + Request-Reply in sistemi Java ad alta concorrenza (2026-04-09)

---

## Sorgenti ingested

### Articles
- `articles/infoq_architectural-governance-ai-speed.md` — Governance architetturale a velocità AI
- `articles/vercel_agent-responsibly.md` — Agenti responsabili, ownership del deployment
- `articles/tessl_context-development-lifecycle.md` — CDLC: contesto come codice
- `articles/como_preparation-collaborative-modeling-workshops.md` — Preparazione workshop collaborative modeling

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
- `courses/KodeKloud AWS Solutions Architect Associate Certification.md` — VPC, subnet, NAT, routing AWS
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
- Totale pagine wiki: 22 (11 concepts, 5 patterns, 4 technologies, 1 synthesis, 1 index, 1 log)
- Sorgenti ingested: 28 (4 articles + 21 courses + 3 blog posts)
- Ultimo aggiornamento: 2026-04-09
