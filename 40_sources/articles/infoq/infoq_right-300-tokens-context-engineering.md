---
tags:
  - context-engineering
  - ai-agents
  - llm
  - coding-agents
  - ai-architecture
feature:
type: article
author: Baruch Sadogursky, Patrick Debois
source: https://www.infoq.com/presentations/architecture-context-engineering/
date: 2026-08-29
---

# The Right 300 Tokens Beat 100k Noisy Ones: The Architecture of Context Engineering

## Sunto

Baruch Sadogursky (DevRel di Tessl AI) e Patrick Debois (pioniere del DevOps) affrontano il problema fondamentale dei coding agent: falliscono non per mancanza di capacità, ma per un contesto mal strutturato. Riempire il context window con istruzioni generiche, documentazione integrale o tool inutilizzati degrada le performance più che migliorarle. La talk propone quattro antipattern con le relative soluzioni architetturali.

Il primo antipattern è lo **"Stuffed Prompt"**: caricare interi file CLAUDE.md con istruzioni conflittuali sovraccarica il contesto. La soluzione è il lazy loading delle skill: le skill si attivano solo quando la descrizione dell'agente corrisponde al task corrente. L'insight chiave è che l'attivazione dipende dalle descrizioni, non dai contenuti: copiare titoli senza curarli vanifica il meccanismo. Il secondo antipattern è l'**"uso dello strumento sbagliato"**: il RAG trova documentazione simile ma errata, la web search restituisce risposte obsolete da Stack Overflow. La soluzione sono i Context Artifact: bundle versionati e testabili di skill, regole e documentazione, da trattare come pacchetti software (Docker image, npm package) anziché come file markdown.

Il terzo antipattern è il **"Goldfish Agent"**: i sistemi di memoria black-box perdono il contesto critico, e chiudere le sessioni azzera tutto l'apprendimento accumulato. La soluzione è esternalizzare la memoria con schema espliciti (es. `.memory/decisions`), scrivendo le decisioni intenzionalmente anziché affidarsi alla compattazione automatica — un approccio ispirato al film "Memento". Il quarto antipattern è il **"Vibes Eval"**: la valutazione soggettiva non cattura i fallimenti in modo misurabile. La soluzione è l'LLM-as-a-judge con rubric di scoring esplicite: la libreria pidge è passata dal 35% di correttezza al 98% semplicemente aggiungendo context artifact (regole, skill, documentazione), con iterazione simultanea sui prompt di generazione e di valutazione.

Il messaggio centrale è che il context engineering è un problema architetturale, non di prompt engineering. Richiede versioning, distribuzione, testing e infrastruttura di gestione analoga a quella del software tradizionale. Il context window non va riempito — va curato: 300 token pertinenti battono 100.000 token di rumore su qualsiasi workload reale.
