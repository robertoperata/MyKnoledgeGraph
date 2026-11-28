---
tags:
  - source-control
  - ai-driven-development
  - monorepo
  - content-addressed-storage
  - developer-tooling
feature:
type: article
author: InfoQ
source: https://r.mail.infoq.com/mk/cl/f/sh/7nVU1aA2ng01QrNPzw3skRox9dQzBd3/V5vL22spfzj_
date: 2026-11-28
---

# How Anthropic Redesigned Source Control for AI-Driven Development

## Sunto

Anthropic ha descritto pubblicamente la migrazione del proprio monorepo da Git a **Scion**, un sistema di controllo versione progettato specificamente per i workload degli agenti AI e dei sistemi CI. La scelta di abbandonare Git non è stata dettata da una limitazione fondamentale del modello distribuito, ma dall'inadeguatezza delle sue assunzioni operative in un contesto in cui gli agenti eseguono migliaia di operazioni di repository in parallelo, spesso su sottoinsiemi molto ridotti dell'albero dei file.

Scion sposta le operazioni di repository sul lato server, eliminando la necessità di cloni locali completi. Il cuore tecnico del sistema è il **content-addressed storage** combinato con cache a livelli: ogni oggetto è identificato da un hash del suo contenuto, rendendo la deduplicazione e la cache banali a qualsiasi livello del sistema. Per un agente AI o un job CI che legge solo alcuni file specifici, questo si traduce in accessi quasi istantanei senza dover scaricare l'intera storia del repository. Il trade-off scelto è esplicito: si sacrifica l'indipendenza offline (un principio fondante di Git) in favore di meno dati locali e infrastruttura condivisa più efficiente.

Questo redesign riflette una tendenza emergente nell'ingegneria del software: gli strumenti di sviluppo devono evolvere per servire non solo gli sviluppatori umani ma anche gli agenti AI come first-class client. Git è stato progettato per flussi di lavoro umani — branch, merge, review — dove la latenza di qualche secondo è accettabile. In un sistema dove centinaia di agenti fanno checkout simultanei per eseguire task paralleli, le assunzioni di Git diventano colli di bottiglia architetturali. Scion è un segnale che le aziende all'avanguardia stanno iniziando a ripensare l'intera toolchain di sviluppo tenendo conto degli agenti come attori primari.
