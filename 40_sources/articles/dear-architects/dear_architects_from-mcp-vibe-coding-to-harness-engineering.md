---
tags:
  - ai-engineering
  - mcp
  - agentic-ai
  - harness-engineering
  - context-engineering
feature:
type: article
author: Birgitta Böckeler
source: https://www.infoq.com/podcasts/mcp-vibe-coding-harness-engineering/
date: 2026-06-14
---

# From MCP and Vibe Coding to Harness Engineering: How Did AI Native Engineering Evolve in One Year

## Sunto

Birgitta Böckeler (Distinguished Engineer, Thoughtworks) analizza in questo podcast InfoQ con Olimpiu Pop l'evoluzione rapida dell'AI-native engineering nell'arco di un anno. Il punto di partenza è il "vibe coding": un anno fa, questa pratica descriveva sviluppatori che usavano l'AI per generare snippet di codice da assemblare manualmente, una forma sofisticata di copia-incolla da Stack Overflow senza uscire dall'IDE. Il panorama si è spostato drasticamente verso agenti più autonomi con responsabilità sull'intero workflow di generazione del codice.

Il Model Context Protocol (MCP) ha avuto un'ascesa e una ricalibratura significativa. Inizialmente ha guadagnato trazione per consentire agli LLM di interagire con sistemi esterni come Figma. Tuttavia, l'ecosistema ha scoperto limitazioni strutturali: le strutture monolitiche dei file consumavano eccessivo spazio nella context window, e il caricamento statico era inefficiente. Approcci alternativi sono ora preferiti: "skills" (contesto caricato lazy), CLI, e script che caricano le risorse solo quando necessario. MCP rimane comunque valido per integrazioni specifiche con strumenti consolidati (software di modellazione 3D, piattaforme specializzate), ma non è più il default.

Il concetto centrale del podcast è l'"harness engineering": il completo scaffolding che circonda un agente AI. Storicamente "il developer era il harness", ora la disciplina include tool di quality assurance (mutation testing, code coverage, linter), architecture fitness functions che validano le decisioni di design, e meccanismi di feedback sulla salute della test suite. Il "context engineering" è la pratica correlata: si tratta di strutturare strategicamente ciò che il modello vede attraverso elementi feed-forward (convenzioni di codice, documentazione architetturale, contesto di business) e meccanismi di feedback (analisi statica, risultati dei test, errori del compilatore) che consentono l'auto-correzione. L'obiettivo è ridurre la supervisione umana aumentando la confidence dell'agente attraverso vincoli ambientali e loop di validazione immediata.

Per le decisioni di supervisione, Böckeler propone un framework a tre dimensioni: probabilità (confidence nel successo dell'AI basata sulla qualità del context engineering), impatto (criticità del caso d'uso: proof-of-concept vs. workflow critici di business), e rilevabilità (facilità di identificare errori dell'AI). Questo sostituisce approcci di supervisione indifferenziata con oversight mirato al profilo di rischio reale. Sul fronte degli strumenti, Claude Code (terminal-based) domina ma non per superiorità intrinseca dell'interfaccia: i tool terminal-based offrono esecuzione headless, integrazione nelle pipeline e operazione in background, mentre IDE come Cursor offrono debugging visuale, rollback e oversight su task complessi.

Le tendenze emergenti includono la renaissance dei terminal UI (TUI) con framework sofisticati, la leadership dell'open-source con OpenCode come agente terminal dominante, e la ripresa di approcci formali (formal validation, termination simulation testing) come meccanismi di validazione mentre la velocità di generazione aumenta. Böckeler anticipa: più storie di fallimento pubbliche che consentono apprendimento collettivo, migliore design delle API per integrazione agenti, e continua evoluzione rapida nonostante le aspettative di rallentamento.
