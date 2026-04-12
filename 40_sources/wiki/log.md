# Wiki Log

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
