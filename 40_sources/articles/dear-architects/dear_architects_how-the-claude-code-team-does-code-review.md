---
tags:
  - code-review
  - ai-engineering
  - pull-requests
  - software-quality
  - continuous-integration
feature:
type: article
author: Addy Osmani, Morgan Westlee Lunt, Qing Wang
source: https://claude.dev/blog/solving-code-review-at-anthropic/
date: 2026-10-18
---

# How the Claude Code team does code review

## Sunto

Questo articolo (ottobre 2026) descrive come il team di Claude Code ha risolto il problema della code review in un contesto in cui il 90% del codice viene scritto da AI. Tra giugno e settembre, le PR mergiate sono più che raddoppiate mentre la quota di PR che richiedeva revert è calata di oltre il 40%. Il risultato non è stato ottenuto rimuovendo la review umana, ma spostando la maggior parte della lettura su check automatici e investendo in rollback rapido.

La filosofia di base è cambiata: la review non è più un gate finale, ma avviene **continuamente durante lo sviluppo**. La domanda chiave non è "un umano ha letto questo?" ma "cosa potrebbe andare storto con questo cambiamento, e quale check dovrebbe rilevarlo?". Il processo è stratificato in sette livelli: (1) verifica pre-PR (test, build, screenshot e subagent review del diff in contesto fresco), (2) AI review su ogni PR (agenti paralleli cercano bug, agenti verifier votano su ogni finding, solo i finding confermati vengono postati, taggati Important/Nit/Pre-existing), (3) security review bloccante, (4) CI obbligatoria (lint, tipi, test cross-platform), (5) approvazione umana risk-based (nessuna per rischio basso, una persona per rischio medio, due per rischio alto), (6) merge queue con feature flag e staged release, (7) detection e rollback rapido.

Un problema critico emerso è stato la **review convergence**: una PR è arrivata a subire oltre 50 cicli di review overnight, e circa il 59% del codice flaggato esisteva già dal primo passaggio. Le soluzioni adottate includono un cap di tre round di review, re-review solo del diff dopo i primi passaggi, flagging per bug class invece che per singola riga, e nessun finding ripetuto. La media di review necessarie per passare è scesa da 2.08 a 1.32.

Il ruolo degli umani è rimasto centrale per le decisioni strategiche: decidere cosa costruire, revisionare codice rischioso (billing, auto-updater, CI workflow), custodire l'architettura, leggere l'evidenza e possedere il risultato. La review non si basa più sulla lettura di ogni diff, ma sul chiedere a Claude di spiegare slice specifiche di codice. Il contesto si condivide in canali aperti invece che in thread privati.

La cultura di deployment si basa sul principio "you land it, you own it": cambiamenti piccoli e reversibili, blast radius come misura del livello di review richiesto, e incident review centrata sul sistema e non sulla colpa individuale. L'articolo si chiude con una checklist adattabile per team che vogliono adottare questo processo, coprendo pre-PR verification, PR template (Testing, Risk, How to revert), risk tier, CODEOWNERS, merge queue, feature flag e trasformazione degli incidenti in eval.
