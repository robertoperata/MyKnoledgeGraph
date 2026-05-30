---
tags:
  - cloudflare
  - ai-agents
  - persistent-memory
  - context-management
  - rag
feature:
type: article
author: Steef-Jan Wiggers
source: https://www.infoq.com/news/2026/04/cloudflare-agent-memory-beta/
date: 2026-05-30
---

# Cloudflare Announces Agent Memory, a Managed Persistent Memory Service for AI Agents

## Sunto

Cloudflare ha annunciato Agent Memory, un servizio gestito in private beta che fornisce agli agenti IA memoria persistente attraverso sessioni, compattazione del contesto e riavvii. Il servizio affronta un problema fondamentale nello sviluppo di agenti IA a lungo termine: la gestione della memoria in modo che rimanga utile man mano che cresce, senza degradare la qualità delle risposte.

Il problema centrale che Agent Memory risolve è il "context rot": la qualità dell'output dell'IA degrada man mano che la context window si riempie, anche quando quella window supera un milione di token. I developer si trovano davanti a un dilemma impossibile — tenere tutto rischiando un calo di qualità, o potare aggressivamente rischiando di perdere informazioni critiche. La soluzione di Cloudflare è estrarre ricordi strutturati dalle conversazioni e recuperare solo quelli rilevanti su richiesta, invece di saturare la context window con l'intera storia.

L'architettura tecnica è sofisticata. Nel processo di ingestione, ogni messaggio riceve un ID content-addressed SHA-256 per la re-ingestione idempotente, poi vengono effettuate due passate di estrazione parallele: una per chunking ampio (~10K caratteri) e una per l'estrazione di dettagli. Otto verifiche classificano i ricordi in quattro tipi: fatti, eventi, istruzioni e task. Il sistema di retrieval usa cinque canali paralleli con Reciprocal Rank Fusion: full-text search, exact fact-key lookup, raw message search, direct vector search e HyDE vector search.

Per la selezione dei modelli, Cloudflare usa di default Llama 4 Scout (17B MoE) per l'estrazione e Nemotron 3 (120B MoE) per la sintesi, avendo scoperto che i modelli più grandi portano benefici principalmente nella fase di sintesi piuttosto che nell'estrazione. Questa scelta riflette un principio più generale: non tutti gli step di una pipeline IA richiedono lo stesso livello di capacità del modello, e differenziare porta a un migliore rapporto costo-qualità.

Il design come servizio gestito (piuttosto che come libreria) riflette la direzione in cui si sta muovendo l'industria per gli agenti a lungo termine: infrastruttura specializzata con garanzie operative che i team applicativi non devono implementare da soli. La dichiarazione del team di Cloudflare che "memory should work against real codebases for weeks or months, not just perform well on clean benchmarks" segnala una maturità nell'approccio — riconoscendo che i benchmark standard non catturano le condizioni reali di produzione degli agenti persistenti.
