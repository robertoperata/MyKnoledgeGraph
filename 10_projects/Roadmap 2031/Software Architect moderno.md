# Mappa delle conoscenze — Software Architect moderno

_Cosa deve sapere un architetto oggi, confrontato con 15-20 anni fa_

---

## Il cambio fondamentale

Negli anni 2000 l'architetto doveva conoscere **in profondità il codice** e **superficialmente l'infrastruttura**, perché l'infrastruttura era relativamente stabile e gestita da un team separato.

Oggi l'infrastruttura è codice — Terraform, Helm chart, pipeline GitOps — e l'architetto che non capisce come il suo sistema vive su Kubernetes non può prendere decisioni sensate sul design. Il confine tra "architettura applicativa" e "architettura infrastrutturale" è praticamente dissolto.

---

## Legenda

|Badge|Significato|
|---|---|
|🟢 Nuovo|Area sostanzialmente inesistente o irrilevante 20 anni fa|
|🟡 Evoluto|Esisteva, ma è cambiato profondamente|
|🔵 Stabile|Ancora rilevante, fondamentali invariati|

---

## 1. Distributed systems design 🟡 Radicalmente evoluto

**20 anni fa:** transazioni ACID su database singolo, consistency garantita dal DB.

**Oggi:** come garantire consistenza quando il dato attraversa 5 servizi e 3 database diversi.

### Topic essenziali

- CAP theorem (Consistency, Availability, Partition tolerance)
- Eventual consistency — quando è accettabile e come gestirla
- **Saga pattern** — transazioni distribuite senza 2PC
- **CQRS** (Command Query Responsibility Segregation)
- **Event sourcing** — il log degli eventi come fonte di verità
- **Outbox pattern** — garantire atomicità tra DB e message broker
- Idempotenza — operazioni sicure da ripetere
- Circuit breaker, Bulkhead, Backpressure — resilienza

---

## 2. Stili architetturali — scegliere il giusto 🟡 Evoluto

**Il pendolo si sta spostando.** Molte aziende che negli anni 2015-2020 hanno decomposto tutto in microservizi si trovano oggi a gestire sistemi estremamente complessi, con problemi di latenza di rete, distributed tracing complicatissimo, e team che faticano a ragionare sul sistema nel suo insieme.

La risposta emergente è il **modular monolith** — un monolite ben strutturato con confini interni netti, deployabile come unità singola ma progettato per essere spezzato quando e se necessario.

L'architetto moderno sa scegliere tra questi stili sulla base del contesto, non per moda.

### Stili da conoscere

- **Microservizi** — quando, perché, e i costi nascosti
- **Monolite modulare** — rivalutato e spesso la scelta giusta
- **Event-driven architecture** — disaccoppiamento asincrono
- **Serverless** — per workload stateless e intermittenti
- **Cell-based architecture** — per alta disponibilità multi-tenant
- Service mesh (Istio, Linkerd)
- BFF (Backend for Frontend)
- API Gateway pattern

---

## 3. Cloud architecture 🟢 Nuovo

Questa area praticamente non esisteva come competenza architetturale 20 anni fa.

Non basta "usare AWS". L'architetto deve saper disegnare come un sistema usa il cloud in modo sicuro, economico e scalabile — e documentare le scelte.

### Topic essenziali

- **Well-Architected Framework** (AWS / Azure) — i 6 pilastri
- **FinOps** — cost optimization, rightsizing, reserved vs spot
- **IaC** (Infrastructure as Code) — Terraform, Pulumi
- Landing zone design — struttura account/subscription
- Multi-tenancy — come isolare i clienti
- Disaster recovery e RTO/RPO
- Multi-region — quando e come
- Managed vs self-hosted — il trade-off reale
- Cloud-native patterns

---

## 4. Container e orchestrazione 🟢 Nuovo

Docker e Kubernetes come strumenti sono già nel toolkit. Il salto architetturale è capirli a livello sistemico: come strutturare i namespace, gestire i segreti, organizzare i deployment in modo sicuro e riproducibile.

### Topic essenziali

- Kubernetes — resource limits, QoS classes, namespace isolation
- Helm — templating e gestione rilasci
- **GitOps** (ArgoCD, Flux) — infrastruttura dichiarativa versionata
- Service mesh — mTLS automatico, traffic management
- Sidecar pattern
- Container security — image scanning, runtime security

---

## 5. Data architecture 🟡 Radicalmente evoluto

**20 anni fa:** un database relazionale per tutto, schema fisso, ACID ovunque.

**Oggi:** ogni servizio può usare il tipo di storage più adatto. La sfida è governare la coerenza e l'accesso tra tutti.

### Topic essenziali

- **Polyglot persistence** — relazionale, documentale, grafo, vettoriale, time-series
- **Event streaming** (Kafka) — il dato come stream di eventi
- **CDC** (Change Data Capture) — Debezium, sincronizzazione tra sistemi
- **Data mesh** — ownership distribuita dei dati
- Lake vs Lakehouse vs Data Warehouse — quando usare cosa
- **Vector database** — pgvector, Qdrant, Weaviate (rilevante per AI)
- Schema evolution — backward/forward compatibility
- Data contracts — accordi formali tra produttori e consumatori

---

## 6. API design e integrazione 🟡 Evoluto

### Topic essenziali

- REST maturity model (Richardson)
- **GraphQL** — quando ha senso vs REST
- **gRPC** — per comunicazione interna ad alte prestazioni
- **AsyncAPI** — specifica per API asincrone/event-driven
- API versioning strategies
- Contract-first design (OpenAPI / Swagger)
- Webhook vs polling — trade-off
- Idempotency keys — sicurezza nelle retry

---

## 7. Observability e reliability 🟢 Quasi nuovo

**20 anni fa:** un log file da analizzare dopo il crash.

**Oggi:** l'osservabilità si progetta prima, non si aggiunge dopo. Un sistema non osservabile non è producibile.

### I tre pilastri

- **Log** — strutturati, correlati, ricercabili
- **Metrics** — RED (Rate, Error, Duration), USE (Utilization, Saturation, Errors)
- **Traces** — distributed tracing, span correlation

### Topic essenziali

- **OpenTelemetry** — standard unificato per i tre pilastri
- **SLO / SLA / SLI** — come definire e misurare l'affidabilità
- Error budget — quanto "errore" è tollerato
- Chaos engineering — testare la resilienza in modo controllato
- Alerting strategy — alert actionable vs alert noise
- MTTR / MTBF — metriche di affidabilità

---

## 8. Security by design 🟡 Molto più centrale

Non è più un argomento separato affidato a un team di sicurezza. L'architetto incorpora le scelte di sicurezza nelle decisioni strutturali fin dall'inizio.

### Topic essenziali

- **Zero trust architecture** — "never trust, always verify"
- **OAuth2 / OIDC** — autenticazione e autorizzazione moderna
- **mTLS** — comunicazione sicura tra servizi
- Secrets management (Vault, AWS Secrets Manager)
- OWASP Top 10 — vulnerabilità più comuni
- **Threat modeling** — STRIDE, disegnare le minacce prima di costruire
- Supply chain security — dipendenze, SBOM
- Least privilege — minimo accesso necessario

---

## 9. Principi fondamentali — ancora validi 🔵 Stabile

I design pattern GoF non sono tramontati — ma sono strumenti, non religione. **DDD** è diventato ancora più centrale nell'era dei microservizi: i bounded context guidano come si tagliano i servizi.

### Topic essenziali

- **SOLID** — principi base ancora validi
- **DDD** (Domain-Driven Design) — linguaggio ubiquo, bounded context, aggregate
- **Bounded context** — il confine naturale di un microservizio
- Hexagonal architecture (Ports & Adapters)
- Clean architecture
- Design patterns GoF — selettivi, non meccanici
- **Fitness functions** — test automatici per le proprietà architetturali

---

## 10. AI systems architecture 🟢 Emergente — alto impatto

L'architetto non deve saper addestrare modelli — deve saper disegnare sistemi che li usano in modo affidabile, sicuro ed economicamente sostenibile. Questa skill è ancora rarissima e molto richiesta.

### Topic essenziali

- **LLM integration patterns** — come integrare LLM in sistemi esistenti
- **RAG architecture** — retrieval-augmented generation su knowledge base
- **Vector database design** — chunking, embedding, indexing strategy
- AI ops e model lifecycle — versioning, A/B testing, rollback
- **Prompt injection** — attack vector specifico dei sistemi AI
- AI cost governance — token cost, caching, batching
- Agentic systems — orchestrazione di agenti autonomi
- Evaluation frameworks — come misurare la qualità delle risposte

---

## La cosa più importante che non è nell'elenco

**Conway's Law:** _"Le organizzazioni producono sistemi che rispecchiano le loro strutture di comunicazione."_

Un architetto che capisce solo la tecnologia ma non come è organizzato il team che la costruisce farà scelte architetturali che il team non sarà in grado di implementare bene. Oggi l'architetto pensa anche a come i bounded context DDD si mappano sulla struttura dei team.

È per questo che il ruolo è inevitabilmente anche di leadership: non si può disegnare un sistema senza capire chi lo costruirà e come quelle persone comunicano tra loro.

---

## Confronto sintetico: 2005 vs 2025

|Dimensione|~2005|~2025|
|---|---|---|
|Focus principale|Codice ben strutturato|Sistema nel suo insieme|
|Infrastruttura|Delegata a un team separato|Codice, responsabilità dell'architetto|
|Database|Un RDBMS per tutto|Polyglot, scelto per use case|
|Deployment|WAR su application server|Container su Kubernetes|
|Comunicazione servizi|Chiamate sincrone (SOAP/REST)|Mix sync + async, event-driven|
|Resilienza|Try-catch, log|Circuit breaker, chaos engineering, SLO|
|Sicurezza|Perimetrale (firewall)|Zero trust, security by design|
|Osservabilità|Log file|Traces + metrics + logs (OpenTelemetry)|
|AI|Non contemplata|Componente architetturale da governare|
|Team|Struttura organizzativa separata dalla tech|Conway's Law — design e team allineati|

---

## Libri di riferimento

|Libro|Perché leggerlo|
|---|---|
|_Designing Data-Intensive Applications_ — Kleppmann|Fondamenta dei sistemi distribuiti moderni. Il libro più citato|
|_Software Architecture: The Hard Parts_ — Ford, Richards|Pattern per sistemi distribuiti, trade-off reali|
|_Building Microservices_ — Sam Newman|Reference completo sui microservizi|
|_Domain-Driven Design_ — Eric Evans|Il "libro blu", fondamento del DDD|
|_Fundamentals of Software Architecture_ — Ford, Richards|Panoramica moderna del ruolo|
|_Release It!_ — Nygard|Resilienza e pattern di produzione|
|_Team Topologies_ — Skelton, Pais|Come struttura del team e architettura si influenzano|

---

_Documento creato il 29 marzo 2026_