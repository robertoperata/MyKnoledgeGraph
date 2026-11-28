---
tags:
  - agentic-ai
  - context-engineering
  - memory-management
  - llm-architecture
  - systems-engineering
feature:
type: article
author: Adi Polak
source: https://www.infoq.com/articles/agentic-ai-context-memory-knowledge/
date: 2026-11-28
---

# Systemic Approach to Memory, Knowledge, and Context in Agentic AI Architectures

## Sunto

La tesi centrale di Adi Polak è che la sfida architetturale principale degli agenti autonomi non riguarda il modello in sé, bensì la gestione del contesto: decidere cosa un agente deve sapere, ricordare e dimenticare in ogni momento. L'articolo separa nettamente tre componenti concettuali: il **contesto** (informazioni temporanee nella finestra attiva del modello), la **memoria** (persistenza attraverso le interazioni) e la **conoscenza** (base curata di fatti, regole e dati di dominio). Questa distinzione trascina una conseguenza architetturale diretta: non è più sufficiente fare prompt engineering — occorre costruire sistemi.

Il pezzo centrale dell'architettura proposta è il **context engine**: un livello che seleziona, filtra, ordina per rilevanza, comprime e consegna le informazioni al modello prima dell'inferenza. Polak lo paragona a un compilatore che trasforma dati grezzi in uno stato pronto per il ragionamento. Il ciclo di vita di questo motore si articola in quattro fasi: selezione, iniezione, compressione ed espirazione/refresh. La gestione della memoria è stratificata su tre livelli: memoria di lavoro (finestra del prompt), cache di sessione a breve termine e persistenza asincrona a lungo termine. La combinazione di dati storici e segnali real-time è essenziale: un'architettura che privilegia solo la memoria episodica di lungo periodo rischia la stasi; una che usa solo segnali in tempo reale perde la coerenza contestuale.

Un aspetto controintuitivo ma ben argomentato riguarda le allucinazioni: Polak le attribuisce principalmente a un contesto mal gestito, non a un modello difettoso. La cura è quindi nel sistema, non nel modello. Questo implica un'economia rigorosa dei token: il fatto che si possano inviare 100.000 token non significa che si debba farlo. L'articolo cita la ricerca "Lost in the Middle" per mostrare come il recall del modello degradi quando le informazioni rilevanti sono sepolte in un contesto troppo ampio, e raccomanda un budget di contesto esplicito come misura di qualità.

L'articolo chiude con una tassonomia di anti-pattern da evitare: **context stuffing** (riempire il prompt con tutto il disponibile sperando che il modello estragga ciò che serve), **stale data loops** (riciclo di informazioni obsolete), **generic retrieval** (recupero per similarità senza ranking per rilevanza contestuale), **explicit/implicit mismatch** (conflitto tra istruzioni esplicite e informazioni di contesto implicite) e **unbounded memory growth** (accumulo indefinito senza meccanismi di scadenza). Ciascun anti-pattern viene corredato di strategie di mitigazione, rendendo l'articolo una guida pratica per chi progetta sistemi agentici in produzione.

## Immagini

![Figura 1 - Historical Data vs Real-Time Memory Streams](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/agentic-ai-context-memory-knowledge/en/resources/253figure-1-1793173490913.jpg)

![Figura 2 - Agentic AI Systemic Context Lifecycle](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/agentic-ai-context-memory-knowledge/en/resources/189figure-2-1793173490913.jpg)

![Figura 3 - Lost in the Middle: recall vs. accuracy](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/agentic-ai-context-memory-knowledge/en/resources/155figure-3-1793173490913.jpg)
