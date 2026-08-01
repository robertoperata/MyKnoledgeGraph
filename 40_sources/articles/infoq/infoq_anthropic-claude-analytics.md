---
tags:
  - ai
  - llm
  - analytics
  - data-governance
  - natural-language-query
  - ai-engineering
feature:
type: article
author: Renato Losio
source: https://www.infoq.com/news/2026/06/anthropic-claude-analytics/
date: 2026-08-01
---

# Anthropic Reports Claude Now Handles 95% of Internal Analytics Queries

## Sunto

Anthropic ha raggiunto un risultato significativo nel dominio dell'analisi dati guidata dall'AI: Claude gestisce circa il 95% delle richieste di analisi interne con una precisione aggregata del ~95%. La scoperta più importante non riguarda il modello in sé, ma l'infrastruttura che lo supporta: l'azienda attribuisce questo successo principalmente alla governance dei dati e alle definizioni semantiche piuttosto che ai miglioramenti del modello linguistico.

L'architettura implementata è uno stack analitico agente a quattro livelli. Il primo livello, Data Foundations, comprende modelli governati, metriche e metadati che costituiscono la base del data warehouse. Il secondo livello, Knowledge Layer, include definizioni semantiche, lineage e contesto aziendale che permettono al modello di capire il significato dei dati. Il terzo livello, Skills, codifica workflow analitici ripetibili e pattern di analisi comuni. Il quarto livello, Validation Systems, verifica correttezza e coerenza degli output del modello prima che questi vengano consegnati agli utenti.

I risultati quantitativi dimostrano l'importanza delle Skills codificate: senza di esse, la precisione sulle domande analitiche era solo del 21%; con la loro implementazione, la precisione supera il 95% complessivamente, avvicinandosi al 99% in alcuni domini specifici. Questo miglioramento ha liberato il team dati per concentrarsi su lavori strategici come la modellazione causale, le previsioni e l'analisi approfondita, invece di rispondere manualmente a query di routine.

I principi fondamentali enfatizzati dal team di Anthropic sono tre: mantenere un'unica fonte di verità per le metriche (evitando definizioni contrastanti tra team diversi), rendere i dati rilevanti scopribili (affinché il modello possa trovare e interpretare correttamente le fonti appropriate), e rilevare continuamente le definizioni obsolete (il dato "stale" è uno dei principali sabotatori della precisione). La nota chiave è che "le fondamenta dei dati sono il data warehouse stesso", mentre le fonti di verità fungono da superfici di riferimento consultabili dall'agente.

Il caso Anthropic offre un modello replicabile per organizzazioni che vogliono democratizzare l'accesso ai dati tramite AI: la qualità del contesto semantico e la governance rigorosa valgono molto più dei parametri del modello per casi d'uso analitici interni.
