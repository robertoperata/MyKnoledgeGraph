---
tags:
  - personalization
  - software-architecture
  - governance
  - recommendation-systems
  - enterprise-architecture
feature:
type: article
author: Jerald Selvaraj
source: https://www.infoq.com/articles/architecture-enterprise-personalization-relevance-governance/
date: 2026-09-27
---

# Beyond Relevance: A Governance-First Architecture for Enterprise Personalization

## Sunto

Il problema fondamentale dei sistemi di personalizzazione tradizionali è che sanno rispondere alla domanda "Cosa è rilevante?" ma non a "Dovremmo consegnare questa raccomandazione adesso?". Le decisioni di governance — consenso, fatica dell'utente, appropriatezza del canale, costo — avvengono tipicamente a valle come annotazioni post-ranking, anziché condizionare il ranking stesso. Questo articolo propone un'architettura a sei stadi che separa le responsabilità in componenti indipendenti e ispezionabili.

La pipeline decisionale è composta da: Experience Memory Layer (EML) per il recupero di fiducia cross-sessione, fatica e preferenze; Temporal Knowledge Graph Engine (TKGE) per la mappatura della fase corrente del customer journey; Hybrid AI Orchestration Engine (HAOE) per la selezione del livello inferenziale (regole → SLM → ML → LLM opzionale); Experience DNA Score (EDS) per il calcolo della rilevanza con scomposizione per componenti (intento, engagement, fit aziendale, contesto del journey); Trust-Aware Personalization Layer (TAPL) per la conversione dei segnali di governance in aggiustamenti del punteggio; e Outcome Simulation Engine (OSE) per la stima degli effetti su conversione, ricavo, fiducia e compliance.

Il principio chiave è che la governance cambia il ranking, non solo le annotazioni. Lo stesso candidato produce risultati diversi in base al contesto: con consenso e bassa fatica viene classificato completamente; con alta fatica viene declassato; senza consenso diventa un fallback generico. Questi sono operazioni misurabili sul percorso del punteggio, non metadati di logging.

Il routing basato su policy (invece di incorporare la selezione del livello nella logica applicativa) rende la scelta esplicita e verificabile: scenari strutturati usano regole, pattern ripetibili usano SLM, scoring addestrato usa ML classico, e la risoluzione di ambiguità usa LLM opzionali quando il budget lo consente. Evitare l'escalation all'LLM è spesso la decisione architetturale corretta, non un fallimento.

L'esplainability è definita come contratto API: le risposte espongono il livello selezionato e la sua motivazione, le regole attivate, le azioni di fiducia applicate, la scomposizione dei componenti del punteggio, le stime degli outcome e le fonti di spiegazione. Questo consente auditing operativo e investigazione degli incidenti. La memoria stateful cambia il significato: la stessa offerta rilevante ha un significato diverso dopo che il cliente l'ha rifiutata più volte.

## Codice

Lo stack implementativo usa FastAPI, SQLite per lo stato cross-sessione, policy YAML per le regole di governance, e una facade del profilo pluggabile per evitare il coupling diretto con CRM/CDP.

I risultati del benchmark nel travel scenario (upsell autonoleggio) hanno mostrato: latenza delle regole entro budget, 100% di fallback rate per il livello SLM in ambiente hosted, scoring ML entro 1500ms, e LLM entro 8000ms. I risultati evidenziano che il livello richiesto ≠ il livello eseguito, e che la latenza infrastrutturale domina quella del modello in ambienti free-tier.
