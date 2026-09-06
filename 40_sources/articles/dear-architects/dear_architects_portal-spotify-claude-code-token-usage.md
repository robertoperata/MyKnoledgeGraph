---
tags:
  - ai-coding
  - claude-code
  - token-optimization
  - agent-routing
  - developer-productivity
feature:
type: article
author: Dimitri Mazmanov
source: https://engineering.atspotify.com/2026/9/portal-by-spotify-cut-my-claude-code-token-usage-by-90
date: 2026-09-06
---

# Portal by Spotify cut my Claude Code token usage by 90%

## Sunto

Dimitri Mazmanov, Principal Product Manager di Spotify, analizza in questo articolo dell'Engineering Blog (settembre 2026) una problematica sempre più urgente nello sviluppo assistito da AI: il consumo eccessivo di token con agenti di coding come Claude Code. Il punto centrale dell'analisi è una distinzione fondamentale: "La maggior parte di ciò che un agente AI fa per me non è pensiero. È I/O." I modelli frontier vengono impiegati per operazioni banali come la lettura di file o la generazione di boilerplate, con un costo sproporzionato rispetto al valore prodotto. Le proiezioni del settore indicano che i costi degli agenti AI di coding supereranno gli stipendi medi degli sviluppatori entro il 2028.

La soluzione proposta è **Portal**, uno strumento di Spotify che introduce le **AiKA Modes**: agenti dichiarativi configurabili che permettono di delegare operazioni I/O-intensive a modelli più economici (come Gemini 2.5 Flash), preservando i modelli costosi per il ragionamento complesso. Un "mode" in questo contesto è un agente dichiarativo che gira su un runtime effimero — paragonabile a AWS Lambda, ma per gli agenti AI. Si definiscono le istruzioni, si sceglie il modello, si impostano parametri come la temperatura e si allegano strumenti MCP.

I due mode principali implementati sono **bulk-reader** e **code-writer**. Il bulk-reader gestisce l'analisi di file multipli di grandi dimensioni: legge i file e restituisce riassunti strutturati usando Gemini 2.5 Flash con temperatura 0.2 per massimizzare la precisione, rimuovendo testo superfluo per minimizzare il consumo di token di Claude. Il code-writer genera boilerplate (test, configurazioni, stub) seguendo i pattern esistenti nel codebase, producendo codice grezzo senza formattazione Markdown per evitare che Claude consumi token nell'analisi di wrapper testuali inutili.

L'architettura del sistema di routing opera su tre livelli distinti: **Hooks** (ganci PreToolUse che intercettano le letture di file superiori a 350 righe — soglia configurabile — e le reindirizzano al bulk-reader), **Scripts** (wrapper Bash per la gestione delle invocazioni Portal CLI e la gestione degli errori), e **Skills** (file Markdown che documentano quando e come invocare la delega). I test su un monorepo Java hanno dimostrato un risparmio di circa il 90% nei token per gli scenari bulk-read. Anche i scenari code-write mostrano efficienze comparabili, impedendo a Claude di consumare i token generati in output.

Le limitazioni del sistema sono chiaramente identificate: il routing non funziona per le operazioni di editing (i numeri di riga diventano inaffidabili), per le analisi di ragionamento complesse, o per l'analisi di sicurezza del codice critico. Questi casi d'uso richiedono ancora l'impiego del modello frontier completo. L'approccio dimostra come un'architettura di routing intelligente degli agenti AI possa trasformare l'economics dello sviluppo software assistito da AI, rendendo sostenibili workflow intensivi che altrimenti risulterebbero proibitivi.
