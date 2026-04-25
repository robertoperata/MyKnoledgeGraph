---
tags:
  - claude-code
  - ai-code-review
  - multi-agent
  - pull-request
  - anthropic
feature:
type: article
author: Daniel Dominguez
source: https://www.infoq.com/news/2026/04/anthropic-code-review/
date: 2026-04-25
---

# Anthropic Introduces Agent-Based Code Review for Claude Code

## Sunto

Anthropic ha introdotto una funzionalità di Code Review per Claude Code, disponibile in research preview per utenti Team ed Enterprise. Il sistema si attiva automaticamente all'apertura di una pull request e impiega più agenti AI in parallelo per analizzare le modifiche al codice — un approccio distintivo rispetto ai tool di review AI tradizionali che utilizzano un singolo modello in modalità sequenziale.

Il meccanismo di revisione multi-agente prevede che il numero di agenti venga scalato dinamicamente in base alla dimensione e complessità della PR. Ogni agente si specializza nella ricerca di potenziali bug, verificando i risultati per minimizzare i falsi positivi, per poi aggregare e classificare i problemi riscontrati per severità prima di pubblicare una review riassuntiva con commenti inline sulla pull request. Il tempo medio di review è circa 20 minuti.

I dati interni di Anthropic mostrano risultati significativi: i commenti di review "sostanziali" sono aumentati dal 16% al 54% delle PR dopo l'adozione interna. Per le PR di grandi dimensioni (oltre 1.000 righe), l'84% ha generato finding con una media di 7.5 problemi; le PR piccole (meno di 50 righe) hanno generato finding nel 31% dei casi, con una media di 0.5 problemi. Il tasso di errore dichiarato è inferiore all'1%.

La community di sviluppatori ha accolto la feature con pareri contrastanti: apprezzamento per la profondità dell'analisi multi-agente, ma preoccupazioni riguardo al costo ($15-25 per PR), che la rende potenzialmente impraticabile per workflow ad alto volume di commit. Un tema di riflessione importante sollevato è il rischio implicito di fare revisionare codice generato da Claude tramite Claude stesso, con possibili blind spot nell'identificazione di errori di ragionamento sistematici del modello.

Il lancio posiziona Anthropic in competizione con strumenti esistenti come GitHub Copilot code review e CodeRabbit, con la differenziazione centrata su un'analisi più profonda tramite multi-agent rispetto a review leggere e rapide. Il tool è progettato esplicitamente per supportare i reviewer umani, non per sostituirli, e non approva automaticamente le PR.
