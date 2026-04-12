---
name: Wiki Knowledge Base — Build Status
description: Stato della wiki tecnica personale in 40_sources/wiki/
type: project
---

La wiki è stata costruita il 2026-04-09 a partire da 28 sorgenti (4 articles, 21 courses, 3 blog posts).

**Struttura:** `40_sources/wiki/` con concepts/, patterns/, technologies/, synthesis/
**Pagine create:** 28 totali (13 concepts, 5 patterns, 4 technologies, 1 synthesis, 1 index, 1 log)

**Nota tecnica:** Il frontmatter YAML usa il formato lista con stringhe quotate per i wiki links:
```yaml
sources:
  - "[[NomeFile]]"
related:
  - "[[path/file]]"
```
Non usare `sources: [[link1]], [[link2]]` inline — causa errori di parsing in Obsidian.

**Why:** Il linter Obsidian modifica automaticamente il frontmatter se i wiki links non sono quotati.
**How to apply:** Sempre quotare i wiki links nel frontmatter quando si creano/aggiornano pagine wiki.

**Lacune aperte:**
- Kafka Streams/KTable non coperto
- Testing microservizi (Pact, contract testing) non coperto
- Java Persistence (JPA/Hibernate): sorgente con note quasi assenti
- CI/CD avanzato / GitOps: menzionato ma non approfondito
