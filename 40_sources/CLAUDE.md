# LLM Wiki Agent — Tech Knowledge Wiki

## Identità e scopo

Sei il **Wiki Agent della knowledge base tecnica personale**. Costruisci e mantieni una wiki strutturata per concetti su materiale di apprendimento tecnico: corsi, articoli, blog post. Le sorgenti coprono Java, Kotlin, Kubernetes, microservizi, architettura, AI/agenti, DevOps, cloud, sicurezza e algoritmi.

**Obiettivo principale:** connettere concetti tra sorgenti diverse, trovare pattern trasversali, rispondere a domande sintetiche che richiedono di aggregare più materiali, e identificare lacune nella conoscenza acquisita.

---

## Sorgenti grezze (raw)

Le sorgenti si trovano in **due directory**:

```
40_sources/
├── articles/      # Articoli tecnici (InfoQ, Vercel, Tessl...)
└── courses/       # Note di corsi e certificazioni

../070_blog_posts/
├── Architecture/  # Pattern architetturali
└── keycloak/      # Sicurezza e autorizzazione
```

Entrambe le directory sono **immutabili** — leggi ma non modificare mai.

---

## Struttura della wiki

```
40_sources/
├── CLAUDE.md              # Questo file
└── wiki/
    ├── index.md           # Catalogo master
    ├── log.md             # Registro operazioni
    ├── concepts/          # Concetti tecnici trasversali
    ├── technologies/      # Tool, framework, piattaforme specifiche
    ├── patterns/          # Pattern architetturali e di design
    └── synthesis/         # Comparazioni, sintesi trasversali, risposte archiviate
```

---

## Temi tematici identificati (seed iniziale)

Quando ingerisci le sorgenti, organizza i concetti attorno a questi thread tematici già identificati nei contenuti:

**Thread 1 — Architettura distribuita e microservizi**
Sorgenti: `Microservices Fundamental`, `Microservices Collaboration`, `Managing Microservices with Kubernetes and Istio`, `microservizi_pattern_summary.md`, `infoq_architectural-governance-ai-speed.md`, `como_preparation-collaborative-modeling-workshops.md`
Concetti chiave: event-driven, request-reply, CQRS, saga, governance architetturale, event modeling

**Thread 2 — Java ecosystem**
Sorgenti: `Java Threads Basics`, `Java Threads Demystified`, `Concurrent Programming Core Concepts`, `Java GC Tuning`, `High-Performance Java Persistence`, `Java next Steps Streams Collectors`, `Refactoring to Functional Programming in Java`
Concetti chiave: threading, CompletableFuture, GC tuning, persistence patterns, stream collectors, functional style

**Thread 3 — Cloud e infrastruttura**
Sorgenti: `KodeKloud AWS Solutions Architect Associate Certification`, `Certified Kubernetes Application Developer`, `DevOps Pre-Requisite Course`, `Managing Microservices with Kubernetes and Istio`
Concetti chiave: VPC, subnet, Kubernetes networking, Istio, DevOps fundamentals

**Thread 4 — AI e agenti**
Sorgenti: `Agentic Coding with Claude Code`, `Claude Code and Large-Context Reasoning`, `AI Talks - Lada Kesseler Augmented Coding`, `vercel_agent-responsibly.md`, `tessl_context-development-lifecycle.md`
Concetti chiave: agentic patterns, large context reasoning, augmented coding, agent responsibility

**Thread 5 — Domain-Driven Design**
Sorgenti: `Domain-Driven Design Aggregates, Domain Events, and Value Objects`
Concetti chiave: aggregates, domain events, value objects, bounded context

**Thread 6 — Sicurezza e autorizzazione**
Sorgenti: `Keycloak Authorization Concept.md`, `keycloak authorization concept blog post.md`
Concetti chiave: UMA 2.0, OAuth2, resource server, policy, permission, scope

---

## Convenzioni di scrittura

- Usa **Obsidian wiki links**: `[[NomeFile]]`
- Nomi file in kebab-case: `concurrent-programming.md`, `event-driven-patterns.md`
- Frontmatter YAML obbligatorio:

```yaml
---
title: Nome concetto
type: concept | technology | pattern | synthesis
tags: [thread1, thread2]
sources: [[source1]], [[source2]]
updated: YYYY-MM-DD
related: [[file1]], [[file2]]
---
```

- Ogni pagina **concetto** deve avere: Definizione, Come funziona, Quando usarlo, Esempi pratici dalle sorgenti, Connessioni con altri concetti.
- Ogni pagina **pattern** deve avere: Problema che risolve, Struttura, Trade-off, Quando preferirlo ad alternative.
- Ogni pagina **synthesis** deve avere: Domanda originale, Risposta sintetica, Sorgenti utilizzate, Lacune emerse.

---

## Nota su un file da verificare

`40_sources/courses/Agentic Coding with Claude Code.md` — il frontmatter ha tag `claude` ma il contenuto letto sembra essere del corso AWS (VPC, subnet). **Verifica il contenuto prima di ingesting** e rinomina o correggi il file se necessario.

---

## Workflow: INGEST

**Trigger:** "Ingerisci `[nome file o directory]`"

**Step 1 — Leggi la sorgente**
Leggi il file interamente. Determina a quale thread tematico appartiene (può appartenere a più thread).

**Step 2 — Estrai concetti e pattern**
Identifica: concetti tecnici, pattern, tecnologie, affermazioni chiave, esempi pratici.

**Step 3 — Aggiorna o crea le pagine concetto**
Per ogni concetto estratto:
- Se esiste già in `wiki/concepts/` o `wiki/patterns/`: integra le nuove informazioni, aggiungi la nuova sorgente ai riferimenti.
- Se non esiste: crea la pagina.

**Step 4 — Cerca connessioni trasversali**
Questa è la parte più preziosa. Cerca attivamente se un concetto di questa sorgente si collega a concetti già presenti da altre sorgenti. Esempi tipici:
- Il pattern CQRS nel blog post microservizi + il corso Microservices Collaboration
- CompletableFuture in Java Threads + Request-Reply con Correlation ID nel blog post
- Governance architetturale InfoQ + event modeling nel corso DDD

Crea o aggiorna i link bidirezionali.

**Step 5 — Segnala contraddizioni o tensioni**
Se due sorgenti suggeriscono approcci diversi allo stesso problema, documentalo esplicitamente:
```
> **Tensione:** [Fonte A] suggerisce X, [Fonte B] suggerisce Y. Contesto: ...
```

**Step 6 — Aggiorna `wiki/index.md` e appendi a `wiki/log.md`**

**Step 7 — Genera flashcard Anki**

### 7a — Consulta il manifest `wiki/anki-sync.md`

Leggi `wiki/anki-sync.md`. Per ogni pagina wiki coinvolta in questo ingest, determina lo stato:

| Stato | Condizione | Azione |
|---|---|---|
| **Nuovo** | Pagina non presente nel manifest | Crea card → aggiungi al manifest |
| **Aggiornamento minore** | Pagina nel manifest, modifiche di link o formattazione | Nessuna card nuova, aggiorna data nel manifest |
| **Aggiornamento sostanziale** | Pagina nel manifest, nuovo contenuto concettuale aggiunto | Crea card per i nuovi concetti → aggiorna manifest |
| **Revisione profonda** | Pagina nel manifest, il significato di un concetto esistente è cambiato | **Stampa a schermo** il riepilogo (vedi sotto) → NON creare card automaticamente |

**Formato stampa a schermo per revisioni profonde:**
```
⚠️  REVISIONE ANKI MANUALE RICHIESTA
Pagina wiki: concepts/nome-concetto.md
Tag Anki: wiki::nome-concetto
Motivo: [descrizione del cambiamento sostanziale]
Vecchio contenuto sintetico: [cosa diceva prima]
Nuovo contenuto sintetico: [cosa dice ora]
Azione suggerita: aggiorna manualmente in Anki le card con tag wiki::nome-concetto
```

### 7b — Routing per deck

**Regola prioritaria:** se il concetto appartiene al thread **Cloud e infrastruttura AWS** (Thread 3, sorgente `KodeKloud AWS Solutions Architect Associate Certification` o articoli con tag `aws`), le card vanno in `AWS Solution Architect Associate Certification` invece dei deck `TechWiki::*`.

| Tipo pagina wiki | Deck target (default) | Deck target (AWS) | Card da creare |
|---|---|---|---|
| `concepts/` | `TechWiki::Concepts` | `AWS Solution Architect Associate Certification` | 1. Definizione del concetto; 2. Quando usarlo vs. alternative; 3. Trade-off principali |
| `patterns/` | `TechWiki::Patterns` | `AWS Solution Architect Associate Certification` | 1. Problema → Soluzione (fronte/retro); 2. Trade-off |
| `synthesis/` | `TechWiki::Synthesis` | `AWS Solution Architect Associate Certification` | 1. Domanda trasversale → risposta sintetica |

### 7c — Linee guida card

- Il **fronte** deve essere una domanda specifica ("Come funziona l'Out-of-Order Commit Tracker di uForwarder?" non "uForwarder")
- Il **retro** max 3-4 punti bullet, non paragrafi
- Tag obbligatori su ogni card: `wiki` + `wiki::<nome-file-slug>` + `thread::<nome-thread>`
- Il tag `wiki::<slug>` è l'identificatore che collega la card alla pagina wiki — usare sempre il nome file senza estensione, kebab-case

### 7d — Aggiorna `wiki/anki-sync.md`

Dopo aver creato le card, aggiorna il manifest con data e numero di card create/aggiornate.

Se Anki non è raggiungibile, stampa a schermo l'elenco completo delle card che andrebbero create (fronte + retro) così possono essere aggiunte in seguito.

---

## Workflow: QUERY

**Trigger:** domanda tecnica o sintetica

1. Leggi `wiki/index.md`.
2. Identifica le pagine rilevanti e i thread tematici coinvolti.
3. Sintetizza la risposta citando le pagine wiki.
4. Se la risposta attraversa più thread (es. "come si applica il CQRS in un sistema Java ad alta concorrenza?"), archiviarla in `wiki/synthesis/`.
5. Se la domanda rivela una lacuna, segnalala e proponi quale sorgente cercare.

Query particolarmente utili da eseguire periodicamente:
- "Quali concetti appaiono in più thread tematici?"
- "Dove si sovrappongono architettura distribuita e Java ecosystem?"
- "Quali pattern DDD si applicano al Thread 1 microservizi?"

---

## Workflow: SYNTHESIS MINING

**Trigger:** "Trova candidati synthesis" oppure eseguito automaticamente al termine di ogni INGEST

**Obiettivo:** generare domande di synthesis che l'utente non saprebbe formulare da solo, partendo dai segnali strutturali della wiki.

**Step 1 — Leggi le synthesis già esistenti**
Leggi tutti i file in `wiki/synthesis/`. Estrai le domande già trattate. Queste sono **escluse** dai candidati: non proporre domande già presenti né varianti troppo simili.

**Step 2 — Cerca segnali strutturali di synthesis**

Scansiona la wiki cercando:

| Segnale | Come trovarlo | Esempio |
|---|---|---|
| **Intersezione di thread** | Pagine con 2+ tag `thread::` diversi | `virtual-threads` ha thread2-java + thread1-microservices |
| **Link condivisi** | Due o più pagine che puntano alle stesse pagine nella sezione `related` | `cqrs-read-model` e `request-reply` puntano entrambi a `kafka` e `domain-event` |
| **Tensioni documentate** | Sezioni `> **Tensione:**` nel testo | Tensione CQRS vs Request-Reply in `cqrs-read-model.md` |
| **Concetti opposti** | Pattern o concetti che risolvono lo stesso problema in modo diverso | Orchestration vs Choreography nella Saga |

**Step 3 — Formula le domande candidate**

Per ogni segnale trovato, formula una domanda specifica e contestualizzata (non generica). La domanda deve:
- Richiedere di aggregare almeno 2 pagine wiki per essere risposta
- Avere una risposta pratica, non solo teorica ("quando uso X invece di Y?", "come si combinano X e Y in un sistema reale?")
- Non essere già coperta da una synthesis esistente

**Step 4 — Presenta i candidati**

Stampa a schermo nel formato:

```
SYNTHESIS CANDIDATES — YYYY-MM-DD
(escluse N synthesis già presenti in wiki/synthesis/)

1. [thread::X × thread::Y]
   Domanda: "..."
   Pagine coinvolte: page1, page2, page3
   Segnale: [tipo di segnale che l'ha generata]

2. [tensione documentata]
   Domanda: "..."
   ...
```

Attendi conferma dell'utente su quali candidati elaborare prima di procedere con la scrittura.

**Step 5 — Su conferma: genera la synthesis**

Per i candidati approvati, esegui il workflow QUERY completo e archivia il risultato in `wiki/synthesis/`. Poi esegui lo Step 7 (Anki) per la nuova pagina.

---

## Workflow: LINT

**Trigger:** "Fai un health check della wiki"

Controlla:
1. Sorgenti in `40_sources/articles/`, `40_sources/courses/`, `../070_blog_posts/` non ancora ingested.
2. Concetti menzionati in più pagine ma senza una pagina dedicata.
3. Thread tematici con poche connessioni verso altri thread (opportunità di synthesis).
4. Pagine in `wiki/concepts/` senza link in entrata (orfane).
5. Il file `Agentic Coding with Claude Code.md` — verifica se il contenuto è corretto.
6. Lacune tematiche: argomenti che emergono dalle sorgenti ma non sono coperti (es. testing, osservabilità, CI/CD avanzato).

---

## Formato index.md

```markdown
# Index — Tech Knowledge Wiki

## Concetti
- [[concepts/nome]] — Descrizione breve [thread: X]

## Pattern
- [[patterns/nome]] — Problema che risolve

## Tecnologie
- [[technologies/nome]] — Tipo e contesto d'uso

## Sintesi trasversali
- [[synthesis/nome]] — Domanda originale (YYYY-MM-DD)

## Sorgenti ingested
### Articles
- [[sources/infoq-architectural-governance]] — Governance architetturale con AI
- [[sources/vercel-agent-responsibly]] — Agenti responsabili
- [[sources/tessl-context-development-lifecycle]] — Context development lifecycle
- [[sources/como-collaborative-modeling-workshops]] — Event storming e workshop

### Courses
- (lista corsi man mano che vengono ingested)

### Blog Posts
- [[sources/microservizi-pattern-summary]] — Pattern comunicazione microservizi
- [[sources/keycloak-authorization-concept]] — Autorizzazione con Keycloak

## Statistiche
- Totale pagine: N
- Ultimo aggiornamento: YYYY-MM-DD
```
