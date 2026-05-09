# Wiki Log

## 2026-05-09 — Ingestione Spring @Transactional (Thread 2)

**Operazione:** Ingestione di 1 articolo (Spring Framework Reference Documentation — "Using @Transactional").

**Sorgente processata:**
- `articles/spring_transactional-declarative-annotations.md` — @Transactional dichiarativo: proxy mode vs AspectJ, self-invocation problem, rollback su checked exception (insidia critica), readOnly come hint, propagazione (REQUIRED/REQUIRES_NEW/NESTED), multipli transaction manager, annotazioni composite personalizzate, transazioni reattive

**Nuove pagine create:** 1
- `concepts/spring-transactional.md` — semantica completa di @Transactional; default di rollback (solo RuntimeException); self-invocation con proxy AOP; readOnly hint; propagazione e anti-pattern REQUIRES_NEW; reactive transactions; tensione documentata @Transactional-come-soluzione vs @Transactional-come-problema (da Walkowiak)

**Pagine aggiornate (aggiornamento minore):** 2
- `concepts/jpa-hibernate-performance.md` — aggiunto link bidirezionale a spring-transactional
- `patterns/transactional-outbox.md` — aggiunto link bidirezionale a spring-transactional + nota sul rischio checked exception

**Connessioni trasversali:**
- Spring @Transactional ↔ JPA Hibernate Performance (Thread 2): tensione @Transactional come soluzione (sessioni separate) vs @Transactional come problema (transazione a grana grossa + connessione bloccata) — Walkowiak e Spring docs convergono sullo stesso anti-pattern con prospettive diverse
- Spring @Transactional ↔ Transactional Outbox (Thread 1 × Thread 2): il comportamento di default (no rollback su checked exception) è criticamente rilevante per il pattern outbox — una IOException nel servizio che scrive nell'outbox potrebbe causare un commit parziale silenzioso

**Tensioni documentate:**
- **Tensione: @Transactional come soluzione vs problema in JPA** — senza @Transactional si hanno sessioni separate (anti-pattern 6 Walkowiak); con @Transactional a grana grossa si blocca la connessione (anti-pattern 3). Risoluzione: TransactionTemplate per controllo esplicito dei confini.

**Anki:** nuova pagina spring-transactional → vedi Step 7

---

## 2026-05-09 — Ingestione AI Coding Workflow - Matt Pocock (Thread 4)

**Operazione:** Ingestione di 1 video YouTube (Matt Pocock — "Full Walkthrough: Workflow for AI Coding", 1:36:30).

**Sorgente processata:**
- `articles/youtube_ai-coding-workflow-matt-pocock.md` — workflow completo AI coding: smart zone/dumb zone, grilling session, design concept (Brooks), PRD come destination document, Kanban board DAG, vertical slices, AFK implementation loop (Ralph pattern), TDD come feedback loop strutturale per AI, deep modules (Ousterhout), push vs pull per coding standards, doc rot, Sandcastle parallelizzazione agenti, human-in-the-loop vs AFK

**Nuove pagine create:** 1
- `concepts/deep-modules.md` — deep vs shallow modules (Ousterhout): interfaccia piccola, molta logica interna; gray box mental model; AI-friendly codebase; connessione con information hiding e testing pyramid

**Pagine aggiornate (aggiornamento sostanziale):** 2
- `concepts/agentic-patterns.md` — aggiunta sezione completa "Workflow AI-Human: il framework di Matt Pocock" con: human-in-the-loop vs AFK, grilling session/design concept, PRD come destination document, doc rot, Kanban board con DAG, vertical slices, AFK implementation loop (Ralph pattern), TDD come feedback loop, push vs pull, Sandcastle
- `concepts/context-management.md` — aggiunte sezioni: Smart Zone e Dumb Zone (~100k token limite pratico, 1M context window espande dumb zone non smart zone), Clear vs Compact (clear preferibile per predictability), tensione con approccio Kiro (cache hit rate vs pulizia totale)

**Connessioni trasversali:**
- Deep Modules ↔ Information Hiding (Thread 1 × Thread 4): Parnas e Ousterhout convergono sullo stesso principio — nascondere complessità — con motivazioni diverse (backwards compatibility vs cognitive load/AI navigabilità)
- Smart Zone ↔ Incremental Disclosure (Kiro): entrambi rispondono ai limiti pratici del contesto; approcci diversi (clear vs caching)
- TDD come feedback loop ↔ Testing Pyramid (Thread 1 × Thread 4): il TDD strutturale per AI si collega al livello component del testing pyramid
- Vertical Slices ↔ Testing Pyramid: ogni issue verticale produce qualcosa di testabile end-to-end — allineato con il concetto di traceable bullets

**Tensioni documentate:**
- **Tensione Clear vs Compact (Pocock) vs Prompt Caching (Kiro):** Pocock preferisce azzerare il contesto per predictability; Kiro ottimizza per cache hit rate (90-95%) e quindi evita la context summarization. Due approcci validi per contesti diversi: loop brevi ripetibili vs sessioni lunghe con caching.

**Anki:** aggiornamento sostanziale su agentic-patterns e context-management + nuova pagina deep-modules → vedi Step 7

---

## 2026-05-08 — Ingestione JPA & Hibernate Performance (Thread 2)

**Operazione:** Ingestione di 1 video YouTube (Maciej Walkowiak, Devoxx 2024 — Spring Data JPA & Hibernate performance).

**Sorgente processata:**
- `articles/youtube_spring-data-jpa-hibernate-performance-walkowiak.md` — open-in-view, auto-commit, TransactionTemplate, REQUIRES_NEW, ID manuale, getReferenceById, LAZY fetch, @DynamicUpdate, projections, QuickPerf

**Nuove pagine create:** 1
- `concepts/jpa-hibernate-performance.md` — 8 anti-pattern con fix, regola fondamentale sulle projections, connection management, strumenti di diagnostica

**Connessioni trasversali:**
- JPA Connection Pool ↔ Java Concurrency: troppi thread con transazioni lunghe esauriscono il pool — stesso tradeoff dei thread pool
- JPA Dirty Checking ↔ Garbage Collection: entity inutili nel persistence context aumentano la pressione sul GC
- TransactionTemplate ↔ Transactional Outbox: la gestione della transazione è critica per la pubblicazione atomica dei messaggi
- Projections ↔ CQRS Read Model: le projections sono la stessa idea del read model — forma ottimizzata per una specifica query

**Anki:** 2 card create in TechWiki::Concepts

---

## 2026-05-08 — Ingestione KIP-392 + KIP-881 Kafka Inter-AZ Costs (Thread 1)

**Operazione:** Ingestione di 1 articolo (Stanislav Kozlovski, Get Kafka-Nated, Feb 2026).

**Sorgente processata:**
- `articles/getkafkanated_kip-881-kip-392-inter-az-kafka-costs.md` — KIP-392 (Fetch From Follower), KIP-881 (Rack-Aware Assignment), costi inter-AZ, gotcha IP pubblici AWS, ~50% risparmio con fanout elevato

**Nuove pagine create:** 0

**Pagine aggiornate (aggiornamento sostanziale):** 1
- `technologies/kafka.md` — aggiunta sezione "Ottimizzazione costi inter-AZ" con KIP-392, KIP-881, configurazione, gotcha IP pubblici AWS

**Connessioni trasversali:**
- Inter-AZ optimization ↔ Multi-cloud event-driven: entrambi ottimizzano la rete per Kafka; KIP-392/881 per within-cloud AZ, DEPOSITS per cross-cloud

**Anki:** ⚠️ REVISIONE MANUALE RICHIESTA per `wiki::kafka` (aggiornamento sostanziale con nuova sezione)
Nuova card aggiunta per KIP-392/881.

---

## 2026-05-08 — Ingestione Java 25 LTS (Thread 2)

**Operazione:** Ingestione di 1 video YouTube (Ario de Montreal, JChampions Conference 2026 — Java 25 LTS overview).

**Sorgente processata:**
- `articles/youtube_java-25-lts-features-jchampions.md` — Compact Object Headers (-25% heap), AOT caching, virtual thread pinning fix, Scope Values, Stream Gatherers, compact source files, flexible constructor bodies, Markdown JavaDoc, JFR improvements

**Nuove pagine create:** 1
- `concepts/scope-values.md` — sostituzione immutabile di ThreadLocal; finale in Java 25; propagazione automatica con StructuredTaskScope (attesa Java 29)

**Pagine aggiornate (aggiornamento sostanziale):** 3
- `concepts/virtual-threads.md` — ⚠️ BREAKING UPDATE: pinning su `synchronized` RISOLTO in Java 24; raccomandazione "usa ReentrantLock" non più necessaria; aggiunto link a scope-values
- `concepts/garbage-collection.md` — aggiunto: Compact Object Headers (~25% heap), stato GC Java 25 (ZGC generazionale, non-gen rimosso), AOT caching
- `concepts/structured-concurrency.md` — aggiornata nota di stato: 5ª preview in Java 25, attesa Java 29

**Connessioni trasversali:**
- Scope Values ↔ Virtual Threads: ScopeValue è progettato per essere thread-agnostic; la propagazione automatica ai subtask richiede StructuredTaskScope
- Compact Object Headers ↔ Virtual Threads: meno memoria per oggetti + meno carrier thread = minore pressione sul GC

**Anki:** ⚠️ REVISIONE MANUALE URGENTE per `wiki::virtual-threads` — la raccomandazione su synchronized/ReentrantLock è cambiata in Java 24.
Nuove card aggiunte per Scope Values e Java 25 highlights.

---

## 2026-05-08 — Ingestione Kiro SDD — Al Harris (Thread 4)

**Operazione:** Ingestione di 1 video YouTube (Al Harris, AI Engineer World's Fair 2026 — Kiro SDD).

**Sorgente processata:**
- `articles/youtube_kiro-spec-driven-development-al-harris.md` — EARS format, PBT dai requisiti, backend neurosimbolico, spec come living docs mutabili, incremental disclosure, task isolation, prompt caching

**Nuove pagine create:** 0

**Pagine aggiornate (aggiornamento sostanziale):** 2
- `concepts/spec-driven-development.md` — aggiunte sezioni: EARS format (sintassi strutturata + automated reasoning), PBT dai requisiti (loop requisiti→proprietà→test), spec come living docs, backend neurosimbolico Kiro
- `concepts/context-management.md` — aggiunte sezioni: Incremental Disclosure (minimo contesto + tool per auto-scoperta), Prompt Caching come strategia (90-95% hit rate), link bidirezionale a SDD

**Connessioni trasversali:**
- EARS + PBT ↔ Harness Engineering: i property test generati da EARS sono un meccanismo di harness — validazione automatica dell'allineamento spec/codice
- Incremental Disclosure ↔ CDLC: stessa idea del contesto come artefatto gestito; l'agente "scopre" il contesto invece di riceverlo tutto
- Neurosymbolic backend ↔ SDD Validation Layer: il backend classico sostituisce l'LLM per le operazioni deterministiche del Validation Layer

**Anki:** ⚠️ REVISIONE MANUALE RICHIESTA (aggiornamento sostanziale a spec-driven-development già sincronizzata)
Nuove card aggiunte per i concetti nuovi (EARS + PBT).

---

## 2026-05-08 — Ingestione SDD Tools: Kiro, spec-kit, Tessl (Thread 4)

**Operazione:** Ingestione di 1 articolo (Birgitta Böckeler, martinfowler.com 2025 — SDD tool comparison).

**Sorgente processata:**
- `articles/martinfowler_sdd-tools-kiro-spec-kit-tessl.md` — tassonomia 3 livelli SDD, confronto Kiro/spec-kit/Tessl, spec vs memory bank, parallelo MDD, instruction adherence, Verschlimmbesserung

**Nuove pagine create:** 0

**Pagine aggiornate (aggiornamento sostanziale):** 1
- `concepts/spec-driven-development.md` — aggiunte sezioni: tassonomia 3 livelli (spec-first/anchored/as-source), distinzione spec vs memory bank, tabella tool 2025, parallelo MDD, preoccupazioni aperte (instruction adherence, review burden, Verschlimmbesserung)

**Connessioni trasversali:**
- Instruction adherence (Böckeler) ↔ Harness Engineering: il problema dell'agente che ignora le spec è esattamente il problema che l'harness deve risolvere
- Spec vs Memory Bank ↔ Context Management (CDLC): il memory bank è il contesto persistente del progetto; la spec è il contesto del task corrente

**Anki:** ⚠️ REVISIONE MANUALE RICHIESTA (aggiornamento sostanziale a pagina già sincronizzata)

```
⚠️  REVISIONE ANKI MANUALE RICHIESTA
Pagina wiki: concepts/spec-driven-development.md
Tag Anki: wiki::spec-driven-development
Motivo: aggiunto nuovo contenuto concettuale — tassonomia 3 livelli, 
        distinzione spec/memory bank, confronto tool, parallelo MDD
Azione: aggiorna le card esistenti e aggiungi card per i nuovi concetti
```

Nuove card aggiunte automaticamente per i concetti nuovi (non presenti in Anki).

---

## 2026-05-08 — Ingestione Revolut Banking Backend — Vlad Yatsenko (Thread 1 + Thread 5)

**Operazione:** Ingestione di 1 video YouTube (Vlad Yatsenko, Devoxx 2019 — Anyone can build a bank).

**Sorgente processata:**
- `articles/youtube_building-bank-backend-revolut-yatsenko.md` — Evoluzione architetturale Revolut: MVP monolite → event-driven monolite → event-driven services; dual-write model; fraud detection pre-computation; @Transactional pitfall Spring; TDD mandatory; jOOQ

**Nuove pagine create:** 1
- `concepts/dual-write-model.md` — pattern: scrive in event log + stato corrente nella stessa operazione; alternativa pragmatica a event sourcing puro; differenza da CQRS; uso per pre-computation time-critical (fraud, risk hedging)

**Pagine aggiornate:** 3
- `patterns/cqrs-read-model.md` — link bidirezionale a dual-write come meccanismo di sincronizzazione intra-servizio
- `patterns/modular-monolith.md` — aggiunto caso reale Revolut come validazione empirica del pattern
- `patterns/event-driven.md` — aggiunta sezione "event-driven monolith come step intermedio" (Revolut 2016-2017)

**Connessioni trasversali identificate:**
- Dual-write ↔ CQRS: il dual-write è il meccanismo intra-servizio che alimenta il read side; complementari ma a livelli diversi
- Dual-write ↔ Transactional Outbox: l'outbox garantisce l'atomicità tra write allo stato e pubblicazione evento nel log
- Modular Monolith ↔ Event-driven Monolith (Revolut): la fase 2 di Revolut è esattamente il modular monolith di Fontaine applicato — moduli isolati, comunicazione interna via eventi, deployment unico
- Pre-computation pattern ↔ CQRS Read Model: lo score di fraud detection è un read model pre-calcolato che elimina la latenza al momento della decisione critica

**Anki:** 2 card create in TechWiki::Concepts

---

## 2026-05-08 — Ingestione Modular Monolith — Axel Fontaine (Thread 1 + Thread 5)

**Operazione:** Ingestione di 1 video YouTube (Axel Fontaine, Devoxx 2018 — Majestic Modular Monoliths).

**Sorgente processata:**
- `articles/youtube_majestic-modular-monolith-axel-fontaine.md` — Modular Monolith come alternativa pragmatica ai microservizi; spettro isolamento codice/DB; DDD per confini di modulo; DB as Queue; enforcement architetturale

**Nuove pagine create:** 1
- `patterns/modular-monolith.md` — definizione, spettro isolamento, regola DAG, module API vs impl, isolamento DB, DB as Queue (SKIP LOCKED), scaling, trade-off vs microservizi, quando estrarre

**Pagine aggiornate:** 3
- `concepts/bounded-context.md` — aggregate roots come confini di modulo nel modular monolith
- `concepts/legacy-modernization.md` — modular monolith come step intermedio naturale prima di Strangler Fig
- `concepts/information-hiding.md` — API pubblica vs implementazione privata come information hiding a livello di modulo

**Connessioni trasversali identificate:**
- Modular Monolith ↔ Bounded Context: DDD applicato intra-applicazione — aggregate root = confine del modulo
- Modular Monolith ↔ Legacy Modernization: i confini già ben definiti rendono l'estrazione con Strangler Fig chirurgica
- Modular Monolith ↔ Information Hiding: dipendenze solo sulle API, mai sulle implementazioni
- DB as Queue ↔ Sync/Async Hybrid Architecture: comunicazione asincrona interna senza infrastruttura aggiuntiva

**Anki:** 2 card create in TechWiki::Patterns

---

## 2026-05-08 — Ingestione Sookocheff: REST + Async Hybrid (Thread 1)

**Operazione:** Ingestione di 1 articolo (Kevin Sookocheff, 2020 — REST + async/event-driven).

**Sorgente processata:**
- `articles/sookocheff_restful-http-async-event-driven-services.md` — REST ai confini del sistema + async internamente; queue vs event stream; Christmas Tree Lights anti-pattern; queue ownership; resist DRY inter-sistema

**Nuove pagine create:** 1
- `concepts/sync-async-hybrid-architecture.md` — principio ibrido; anti-pattern catene sincrone; queue ownership; queue vs event stream; progressione verso piattaforma composable

**Pagine aggiornate:** 3
- `patterns/event-driven.md` — aggiunta sezione Queue vs Event Stream con tabella comparativa e nota su queue ownership
- `patterns/api-composition-bff.md` — aggiunta sezione "BFF come confine REST del sistema" e link a sync-async-hybrid
- `concepts/bounded-context.md` — aggiunto link: "resist DRY inter-sistema" come rinforzo del principio bounded context

**Connessioni trasversali identificate:**
- Sync/Async Hybrid ↔ BFF: il BFF è il punto di stabilità REST verso l'esterno mentre il sistema interno è asincrono
- Sync/Async Hybrid ↔ Bounded Context: il "resist DRY inter-sistema" è la stessa cosa del bounded context applicata alle API
- Sync/Async Hybrid ↔ Anti-Corruption Layer: i punti di sovrapposizione tra sistemi con modelli diversi sono integration point da gestire con ACL
- Queue vs Event Stream: distinzione nuova che chiarisce un'ambiguità frequente nell'event-driven

**Anki:** 2 card create in TechWiki::Concepts

---

## 2026-05-08 — Ingestione Anti-Corruption Layer (Thread 1 + Thread 5)

**Operazione:** Ingestione di 1 articolo (Microsoft Azure Architecture Center, pattern ACL).

**Sorgente processata:**
- `articles/microsoft_anti-corruption-layer-pattern.md` — Anti-Corruption Layer pattern: façade/adapter tra sistemi con semantiche diverse

**Nuove pagine create:** 1
- `patterns/anti-corruption-layer.md` — Problema/struttura/trade-off/quando usarlo; pattern DDD originariamente descritto da Eric Evans

**Pagine aggiornate:** 3
- `concepts/bounded-context.md` — aggiunto link bidirezionale ad ACL: l'ACL preserva l'integrità del bounded context nell'integrazione cross-contesto
- `concepts/legacy-modernization.md` — aggiunto link: ACL come abilitatore dello Strangler Fig durante le fasi di coesistenza
- `concepts/hexagonal-architecture.md` — aggiunto link: ACL come estensione del concetto di adapter all'inter-sistema

**Connessioni trasversali identificate:**
- ACL ↔ Hexagonal Architecture: stessa filosofia di isolamento (adapter), scope diverso (inter-sistema vs intra-servizio)
- ACL ↔ Bounded Context: l'ACL è il meccanismo DDD per proteggere la purezza del bounded context dall'inquinamento di modelli esterni
- ACL ↔ Legacy Modernization: ACL e Strangler Fig sono pattern complementari — Strangler Fig definisce la strategia, ACL abilita la coesistenza

**Anki:** 2 card create in TechWiki::Patterns

---

## 2026-05-07 — Ingestione batch Chris Richardson (Thread 1 — microservizi)

**Operazione:** Ingestione di 9 articoli di Chris Richardson (2 talk standalone + serie Microservices Platforms in 7 parti).

**Sorgenti processate:** 9 articles
- `chris_richardson_cubes-hexagons-triangles-yow2019.md` — YOW! 2019: Hexagonal Architecture, Scale Cube, Testing Pyramid, canary deployment, principio Iceberg
- `chris_richardson_not-just-events-async-microservices-goto2019.md` — GOTO 2019: Saga (orchestration + choreography), CQRS, Transactional Outbox, Event Sourcing
- `chris_richardson_microservices-platforms-part-1-overview.md` — Overview dei 6 layer della piattaforma
- `chris_richardson_microservices-platforms-part-2-service-foundation-platform.md` — Service Foundation: microservice chassis + service template
- `chris_richardson_microservices-platforms-part-3-security-platform.md` — Security Platform: IdP, mTLS, secrets management
- `chris_richardson_microservices-platforms-part-4-infrastructure-services-platform.md` — Infrastructure Services: K8s, service mesh, Developer Portal
- `chris_richardson_microservices-platforms-part-5-observability-platform.md` — Observability: metrics, logs, tracing, RED metrics, dashboard Grafana
- `chris_richardson_microservices-platforms-part-6-build-platform.md` — Build Platform: CI/CD pipeline, reusable workflows, artifact registry
- `chris_richardson_microservices-platforms-part-7-deployment-platform.md` — Deployment Platform: K8s, GitOps (ArgoCD/Flux), IaC

**Nuove pagine create:** 8
- `concepts/hexagonal-architecture.md` — Ports & Adapters, inbound/outbound ports, principio Iceberg
- `concepts/scalability-cube.md` — X/Y/Z axis, triangolo del successo, DORA metrics
- `concepts/team-topologies.md` — Platform Team vs Stream-aligned Team, Conway's Law, modern legacy system
- `concepts/microservices-platform.md` — I 6 layer della piattaforma (Richardson/QCon SF 2025)
- `concepts/microservice-chassis.md` — Framework cross-cutting + service template
- `concepts/observability.md` — Three pillars, RED metrics, responsabilità divisa piattaforma/team
- `patterns/transactional-outbox.md` — Atomicità DB + pubblicazione: transaction log tailing vs polling
- `patterns/testing-pyramid.md` — Unit → Contract → Component → E2E, canary deployment
- `technologies/gitops.md` — GitOps pull-based, ArgoCD/Flux, path to production

**Pagine aggiornate:** 5
- `patterns/saga-pattern.md` — aggiunto: ACD (non ACID), semantic lock countermeasure, Saga come state machine persistente, requisiti message broker
- `patterns/cqrs-read-model.md` — aggiunto: API Composition come alternativa semplice, view store eliminabile/ricostruibile, replication lag trade-off
- `concepts/independent-deployability.md` — aggiunto: design-time vs runtime coupling, abilitatori infrastrutturali, anti-pattern iceberg inverso
- `technologies/kafka.md` — aggiunto: Kafka NON è un event store, Transactional Outbox con Debezium
- `technologies/kubernetes.md` — aggiunto: K8s nella Microservices Platform (Infrastructure + Deployment layer)

**Connessioni trasversali notevoli:**
- Thread 1 si consolida significativamente: il framework Richardson lega hexagonal architecture → microservice chassis → platform → team topologies → GitOps in una catena coerente
- `hexagonal-architecture` ↔ `information-hiding`: le porte sono il meccanismo tecnico dell'information hiding
- `transactional-outbox` ↔ `saga-pattern` ↔ `domain-event`: il triangolo fondamentale per l'event-driven affidabile
- `team-topologies` ↔ `independent-deployability`: Conway's Law come legame tra struttura organizzativa e architettura
- `observability` ↔ `testing-pyramid` (canary deployment): la piattaforma di osservabilità abilita il rollback automatico nel canary
- `microservice-chassis` ↔ `twelve-factor`: il chassis implementa nativamente i principi 3 (Config), 11 (Logs), 12 (Admin)
- Thread 1 ↔ Thread 3: `microservices-platform` + `gitops` + `kubernetes` creano un ponte solido tra architettura distribuita e cloud/infrastruttura
- Thread 1 ↔ Thread 6: Security Platform ↔ `technologies/keycloak` — Keycloak è l'Identity Provider della Security Platform

**Tensioni documentate:**
- Design-time coupling vs runtime coupling: strategie diverse (API piccole vs messaggistica asincrona) per problemi fondamentalmente diversi
- Microservice Chassis: se evolve in modo non retro-compatibile, tutti i servizi accoppiati devono aggiornarsi — punto di accoppiamento soft
- Transactional Outbox vs Event Sourcing: stesso problema (atomicità), soluzioni con trade-off radicalmente diversi

**Anki cards create:** 17 (deck TechWiki::Concepts: 12, deck TechWiki::Patterns: 5)

---

## 2026-05-03 — Ingestione corsi AWS Networking (Thread 3)

**Operazione:** Ingestione di 2 nuovi corsi + aggiornamento KodeKloud AWS.

**Sorgenti processate:**
- `courses/hands-on-with-aws-vpcs/knowledge.md` — Rick Crisci, 6h, VPC completo Giorno 1+2
- `courses/aws-security-deep-dive-vpcs/knowledge.md` — Rick Crisci, 4h, VPC security + DDoS
- `courses/KodeKloud AWS Solutions Architect Associate Certification.md` — aggiornamento (le lacune "Argomenti mancanti" ora coperte dai nuovi corsi)

**Nuove pagine create:** 4
- `concepts/aws-vpc-fundamentals.md`
- `concepts/aws-security-groups-nacls.md`
- `concepts/aws-vpc-connectivity.md`
- `concepts/aws-ddos-protection.md`

**Connessioni trasversali notevoli:**
- Thread 3 si consolida: 3 corsi sullo stesso argomento si integrano senza contraddizioni
- `aws-vpc-connectivity` ↔ `technologies/kubernetes` — cluster EKS usano subnet VPC, Transit Gateway per multi-account
- `aws-ddos-protection` ↔ `concepts/aws-security-groups-nacls` — SG è l'ultimo layer nella chain defense in depth
- VPC Gateway Endpoint (gratuito per S3) → elimina costi egress + sicurezza
- **Tensione resuelta:** KodeKloud aveva "Argomenti mancanti" (VPC Endpoints, Transit Gateway, Direct Connect, VPN, Flow Logs) — tutti coperti dai nuovi corsi
- **Concetto nuovo**: Micro-segmentation (SG per istanza) vs subnet-level NACL — rilevante anche per Kubernetes NetworkPolicy

**Anki deck target:** `AWS Solution Architect Associate Certification` (regola prioritaria Thread 3)

---

## 2026-04-17 — Ingestione batch articoli

**Operazione:** Ingestione di 14 nuovi articoli da `40_sources/articles/`.

**Sorgenti processate:** 14 articles
- 5 Dear Architects (async workflows, harness engineering, multi-agent Spotify, PostgreSQL treadmill, autonomous systems)
- 1 Devoxx (Modern Java 21-25)
- 8 InfoQ (S3 Vectors, SDD enterprise, multi-cloud EDA, Netflix graph, CALM, software evolution, SDD, uForwarder)

**Nuove pagine create:** 10
- `concepts/structured-concurrency.md`
- `concepts/spec-driven-development.md`
- `concepts/harness-engineering.md`
- `concepts/legacy-modernization.md`
- `concepts/async-workflow-patterns.md`
- `concepts/autonomous-systems-architecture.md`
- `patterns/multi-cloud-event-driven.md`
- `patterns/kafka-consumer-proxy.md`
- `technologies/calm.md`
- `technologies/vector-databases.md`

**Pagine aggiornate:** 3
- `concepts/agentic-patterns.md` — aggiunto: multi-agent (Spotify), tool grounding, connessioni a harness/SDD/autonomous
- `concepts/virtual-threads.md` — aggiunto: Stream Gatherers mapConcurrent, link a structured-concurrency
- `technologies/kafka.md` — aggiunto: consumer proxy pattern (uForwarder), ottimizzazione multi-cloud

**Connessioni trasversali notevoli:**
- Thread 4 (AI) si consolida: SDD ↔ Harness Engineering ↔ Autonomous Systems formano un cluster coeso
- Thread 2 (Java): Structured Concurrency completa il trittico virtual-threads + completable-future + SC
- Thread 1 ↔ Thread 3: Multi-cloud EDA crea il primo ponte solido tra architettura distribuita e cloud

**Tensioni identificate:**
- PostgreSQL: l'ottimizzazione ha un tetto architetturale — nessuna ottimizzazione risolve l'MVCC overhead per dati append-only
- SDD vs governance tradizionale: le spec ereditano la stessa complessità del codice che dovrebbero semplificare

---

## 2026-04-09 — Build iniziale

**Operazione:** Ingestione completa di tutte le sorgenti disponibili e costruzione della wiki.

**Sorgenti processate:** 28
- 4 articles (40_sources/articles/)
- 21 courses (40_sources/courses/)
- 3 blog posts (070_blog_posts/)

**Pagine create:** 22
- 13 concepts
- 5 patterns
- 4 technologies
- 1 synthesis

**Connessioni trasversali identificate:**
- `CompletableFuture` (Java Threads) ↔ `Request-Reply con Correlation ID` (microservizi blog post): stesso meccanismo, sorgenti diverse
- `Virtual Threads` (Java 21, corsi Java) ↔ scalabilità del Request-Reply (microservizi blog): soluzione di concorrenza applicata al problema architetturale
- `Governance Dichiarativa` (InfoQ) ↔ `CDLC / Context Management` (tessl.io): stessa idea — rendere esplicito e automatizzabile ciò che è implicito
- `Event Models` (InfoQ governance) ↔ `Domain Events` (DDD): base comune
- `Canary Deployment` (Vercel) ↔ `Istio Traffic Management`: implementazione infrastrutturale del principio
- `Workshop collaborative modeling` (CoMo) ↔ `Ubiquitous Language` (Microservices + DDD): il workshop è il processo per costruire la lingua condivisa

**Tensioni documentate:**
- Request-Reply (strong consistency, latenza alta) vs CQRS (eventual consistency, latenza zero) — in `patterns/request-reply-correlation-id.md`
- Virtual Thread (semplifcità, IO) vs WebFlux (throughput massimo, complessità) — in `concepts/virtual-threads.md`

**Lacune emerse:**
- Kafka Streams / KTable: non coperto
- Testing di microservizi (contract testing, Pact): non coperto
- Osservabilità avanzata (OpenTelemetry distribuito): non coperto
- Java Persistence (JPA/Hibernate): sorgente presente ma note quasi assenti
- CI/CD avanzato / GitOps: menzionato ma non approfondito

**Nota su file mancante:**
- `courses/Agentic Coding with Claude Code.md` non esiste. Il CLAUDE.md ne descriveva il frontmatter con tag `claude` ma contenuto AWS. Verificato: il file AWS è correttamente `KodeKloud AWS Solutions Architect Associate Certification.md`. Il tema agentic è coperto da altri due corsi (Lada Kesseler, Tim Warner).
