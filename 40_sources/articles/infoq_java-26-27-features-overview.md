---
tags:
  - java
  - performance
  - security
  - concurrency
type: article
author: Michael Redlich
source: https://www.infoq.com/news/2026/02/java-26-so-far
date: 2026-02-20
---

# JDK 26 and JDK 27: What We Know So Far

## Sunto

JDK 26, primo rilascio non-LTS dopo JDK 25, è alla seconda *release candidate* al momento dell'articolo con **rilascio formale previsto il 17 marzo 2026**. L'articolo elenca i 10 JEP inclusi e anticipa i candidati per JDK 27 (settembre 2026).

### JDK 26 — 10 JEP

| JEP | Titolo | Area | Stato |
|---|---|---|---|
| **500** | Prepare to Make Final Mean Final | Linguaggio | — |
| **504** | Remove the Applet API | Client Library | Rimozione definitiva |
| **516** | Ahead-of-Time Object Caching with Any GC | HotSpot | Basato su JDK 24 AOT class loading |
| **517** | HTTP/3 for the HTTP Client API | Core Library | Nuovo protocollo |
| **522** | G1 GC: Improve Throughput by Reducing Synchronization | HotSpot | Riduce overhead sync in G1 |
| **524** | PEM Encodings of Cryptographic Objects (Second Preview) | Security | Rinomina PEMRecord → PEM; aggiunge encrypt/decrypt per KeyPair e PKCS8EncodedKeySpec |
| **525** | Structured Concurrency (Sixth Preview) | Core Library / Loom | Aggiunge `onTimeout()` a `StructuredTaskScope.Joiner` |
| **526** | Lazy Constants (Second Preview) | Core Library | — |
| **529** | Vector API (Eleventh Incubator) | Panama | Attende Project Valhalla, nessuna modifica sostanziale |
| **530** | Primitive Types in Patterns, instanceof, and switch (Fourth Preview) | Linguaggio / Amber | Nuove definizioni di exactness e controlli dominance in switch |

**Note di rilievo per JEP:**

- **JEP 516 (AOT Object Caching)** — estende il lavoro di JDK 24 sull'AOT class loading per migliorare lo startup time con qualsiasi GC (non solo ZGC).
- **JEP 517 (HTTP/3)** — aggiunge il supporto HTTP/3 all'API `HttpClient` esistente.
- **JEP 525 (Structured Concurrency)** — sesta preview: il metodo `onTimeout()` permette di definire un comportamento di timeout direttamente nel `Joiner`, rendendo la gestione del tempo di attesa più dichiarativa.
- **JEP 529 (Vector API)** — all'undicesima incubazione senza cambiamenti di rilievo: il JEP è in attesa che *Project Valhalla* introduca i *value types* necessari per la sua implementazione finale.
- **JEP 530 (Primitive Types in Patterns)** — quarta preview: inasprisce le regole di *dominance checking* negli switch e raffina la definizione di *unconditional exactness* per i tipi primitivi.

---

### JDK 27 — Candidati (settembre 2026)

| JEP | Titolo | Progetto | Note |
|---|---|---|---|
| **527** | Post-Quantum Hybrid Key Exchange for TLS 1.3 | Security | Già *targeted*; potenzia RFC 8446 con *hybrid key exchange* post-quantistico |
| **401** | Value Classes and Objects (Preview) | Valhalla | Oggetti con soli campi final e senza identità |
| **Draft 8376595** | Lazy Constants (Third Preview) | Amber | Rimuove `isInitialized()` e `orElse()`; aggiunge factory `ofLazy()` per List, Set, Map |
| **Draft 8329758** | Faster Startup and Warmup with ZGC | HotSpot | Migliora l'efficienza di allocazione memoria in ZGC |

Il JEP più atteso è **JEP 401 (Value Classes)** di Project Valhalla: introduce classi con soli campi `final` e senza *object identity*, abilitando ottimizzazioni di layout in memoria fondamentali per l'ecosistema Java (e prerequisito per sbloccare il Vector API).

---

## Link esterni

- [JDK 26 Project Page](https://openjdk.org/projects/jdk/26/) — lista ufficiale dei JEP e stato del release
- [JDK 27 Early Access](https://jdk.java.net/27/) — build early access di JDK 27
- [RFC 8446 – TLS 1.3](https://datatracker.ietf.org/doc/rfc8446/) — specifica TLS 1.3 su cui si basa JEP 527
- [Draft – TLS Hybrid Key Exchange](https://datatracker.ietf.org/doc/draft-ietf-tls-hybrid-design/) — draft IETF per il key exchange post-quantistico ibrido

---

## Immagini

Nessuna immagine presente
