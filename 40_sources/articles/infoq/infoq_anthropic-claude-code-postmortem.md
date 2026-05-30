---
tags:
  - ai-engineering
  - postmortem
  - ai-quality
  - claude-code
  - testing
feature:
type: article
author: Steef-Jan Wiggers
source: https://www.infoq.com/news/2026/05/anthropic-claude-code-postmortem/
date: 2026-05-30
---

# Anthropic Traces Six Weeks of Claude Code Quality Complaints to Three Overlapping Product Changes

## Sunto

Anthropic ha pubblicato un postmortem ingegneristico che spiega sei settimane di reclami degli utenti sulla qualità di Claude Code. Il caso è istruttivo non solo per i problemi specifici identificati, ma per la metodologia di investigazione e le lezioni architetturali che ne derivano sull'ingegneria dei sistemi IA in produzione.

Tre modifiche distinte al product layer, introdotte tra marzo e aprile 2026 con tempistiche sovrapposte, hanno creato sintomi diversi che colpivano segmenti differenti di utenti a seconda di quando usavano il prodotto e quali funzionalità utilizzavano. Il primo problema (4 marzo) è stato il downgrade del "reasoning effort" da "high" a "medium" per ridurre la latenza dell'UI — una scelta che Anthropic ha riconosciuto come "il trade-off sbagliato", ripristinata il 7 aprile. Il secondo (26 marzo) era un bug di caching: un'ottimizzazione per la pulizia delle sessioni si attivava ad ogni turno invece che una sola volta dopo i periodi di inattività, cancellando progressivamente il contesto di ragionamento del modello. Il terzo (16 aprile) riguardava nuovi limiti di verbosità nel system prompt che limitavano il testo tra le chiamate ai tool a 25 parole e le risposte finali a 100 parole, causando un calo di qualità misurabile del 3%.

Le limitazioni dei test interni hanno impedito di rilevare questi problemi prima: il personale usava build diverse, il bug di caching si manifestava solo in condizioni specifiche, e le suite di valutazione non riuscivano a rilevare cali di qualità modesti. Questo evidenzia una sfida sistematica nell'ingegneria IA: i tradizionali framework di testing non catturano adeguatamente la qualità percepita dall'utente in condizioni reali d'uso.

Un rischio silenzioso emerso dall'indagine riguarda la delega ad Haiku: Claude Code delega task a istanze del modello Haiku (più economico) più frequentemente di quanto sia visibile agli utenti. Questo pone rischi particolari per i workflow automatizzati dove il degrado della qualità passa inosservato. Un audit indipendente ha analizzato 6.852 file di sessione confermando spostamenti misurabili verso comportamenti "edit-first" anziché "research-first".

La lezione ingegneristica più importante è che nei sistemi IA complessi, modifiche multiple non correlate possono creare impatti di qualità composti che i framework di testing tradizionali non rilevano. La complessità dell'interazione tra reasoning effort, caching context e vincoli di verbosità ha prodotto sintomi che sembravano casuali agli utenti ma erano deterministici nel prodotto. Per chi costruisce sistemi IA in produzione, il caso Anthropic suggerisce di aumentare la granularità del monitoraggio della qualità, gestire con attenzione le finestre di cambiamento, e creare percorsi di feedback rapidi prima di generalizzare le modifiche all'intero traffico.
