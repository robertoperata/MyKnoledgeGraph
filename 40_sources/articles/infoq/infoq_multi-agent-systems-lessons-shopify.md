---
tags:
  - multi-agent-systems
  - ai-engineering
  - agent-microservices
  - llm
  - developer-productivity
feature:
type: article
author: Paulo Arruda
source: https://www.infoq.com/presentations/multi-agent-system-lessons/
date: 2026-05-30
---

# What I Learned Building Multi-Agent Systems from Scratch

## Sunto

Paulo Arruda, staff engineer di Shopify, condivide le lezioni apprese nella costruzione di sistemi multi-agente su larga scala in una presentazione al QCon AI 2025. Il percorso di Shopify è iniziato con semplici strumenti di chat e ha portato allo sviluppo di uno "sciame" di agenti specializzati, seguendo un'evoluzione che riflette le sfide reali dell'adozione dell'IA in ambienti di produzione complessi.

Il cambiamento architetturale principale è stato il passaggio da prompt monolitici "all-in-one" a microservizi agente lean e focalizzati su un singolo obiettivo. Questo approccio ha prodotto risultati misurabili: il processo di revisione dei temi è passato da 22 ore a 7-20 minuti, le valutazioni dei candidati si completano in meno di un'ora, e 15 istanze di Claude Code coordinano compiti di ricerca attraverso i sistemi interni di Shopify.

Dal punto di vista tecnico, il team ha creato SwarmSDK, una gem Ruby che abilita il supporto multi-provider, la configurazione tramite YAML/DSL, l'orchestrazione dei workflow, event hook e osservabilità integrata. La scelta di costruire questo framework interno ha permesso di mantenere il controllo sull'infrastruttura agente senza dipendere interamente da librerie di terze parti, garantendo flessibilità nell'aggiornamento dei modelli e nell'integrazione con i sistemi Shopify esistenti.

Sul piano organizzativo, Arruda sottolinea l'importanza di evitare l'"AI SWAT team" — un team centralizzato che gestisce tutta l'IA — in favore di un approccio che empowera i singoli team con gli strumenti e i framework appropriati. Questo cambio di mentalità riduce i colli di bottiglia e accelera l'adozione distribuita dell'IA nell'organizzazione.

Per il futuro, Arruda propone il concetto "llm-fuse": uno strato di adapter basato su filesystem che permetterebbe agli agenti di trattare varie fonti di dati come file, affrontando il problema del context bloat sfruttando la capacità dei modelli di codice per la scoperta e il recupero delle informazioni. Questo approccio speculativo mira a risolvere uno dei problemi fondamentali dei sistemi agente a lungo termine: la gestione efficiente del contesto man mano che cresce.
