---
tags:
  - git
  - distributed-systems
  - storage
  - scalability
  - version-control
feature:
type: article
author: Vicent Martí
source: https://cursor.com/blog/git-at-any-scale
date: 2026-08-23
---

# Git at Any Scale

## Sunto

Questo articolo tecnico del blog di Cursor — scritto da Vicent Martí — ripercorre l'evoluzione dell'hosting Git su larga scala, partendo dalle limitazioni fondamentali del design distribuito di Git fino all'architettura del sistema Continuity sviluppato internamente. L'articolo è un'analisi approfondita che copre 13+ anni di lezioni apprese dai sistemi di produzione di GitHub e, successivamente, di Cursor.

Il paradosso centrale è che Git, progettato per la distribuzione, è notoriamente difficile da ospitare in modo scalabile lato server. I clienti Git si connettono ai server con pattern di accesso massivamente paralleli che il sistema non era originariamente pensato per gestire. Tre approcci sono stati tentati nel corso degli anni: i filesystem distribuiti (NFS, GFS, DRBD), abbandonati perché i pattern di accesso casuale ai packfile di Git li rendevano inefficienti; la distribuzione a livello di packfile con consenso a tre fasi (il sistema Spokes di GitHub, ~2013); e infine l'architettura basata su write-ahead log con object storage come fonte di verità (Continuity di Cursor).

Il sistema Spokes di GitHub, con 3-5 repliche per repository, è diventato lo standard industriale ma mostra limitazioni fondamentali oltre quella soglia: il throughput di push degrada con le dimensioni del cluster a causa dell'overhead del consenso, e i repository piccoli richiedono comunque tre repliche idle per le garanzie di consistenza. Il sistema tratta i repository come "animali da compagnia" anziché come "bestiame", imponendo un carico operativo elevato.

L'innovazione chiave di Continuity è la separazione degli strati di storage: S3 diventa la fonte di verità atomica, mentre i repository NVMe locali fungono da cache. I push vengono riconosciuti solo dopo la persistenza completa su S3. La linearizzazione degli aggiornamenti avviene tramite operazioni compare-and-swap. Il routing stateless usa rendezvous hashing invece di database centralizzati. La replica è flessibile: il numero di repliche scala linearmente senza overhead di consenso. La diffusione ottimistica avviene via UDP (lossy), con verifica tramite ETag contro S3.

Le prestazioni misurate sono notevoli: S3 Standard supporta ~120 push/secondo, S3 Express One Zone supera 300 push/secondo, e la lettura scala linearmente fino a 100 repliche testate. La consistenza forte — critica per Git, dove anche un singolo read failure dopo un push causa errori visibili all'utente — è mantenuta attraverso la verifica WAL-based. La strategia di compaction è ottimizzata: solo il nodo primario esegue il repacking costoso, le repliche scaricano dati già compattati da S3, "scambiando banda per CPU".
