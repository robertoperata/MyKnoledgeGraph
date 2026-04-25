---
tags:
  - context-engineering
  - llm
  - agentic-systems
  - kafka
  - memory-management
feature:
type: article
author: Adi Polak
source: https://www.infoq.com/podcasts/context-engineering-large-language-models/
date: 2026-04-25
---

# Context Engineering with Adi Polak

## Sunto

In questo episodio del podcast InfoQ, Thomas Betts intervista Adi Polak (Director at Confluent, autrice di "Scaling Machine Learning with Spark" e "High Performance Spark Second Edition") sull'evoluzione dal prompt engineering verso il **context engineering** per LLM e sistemi agentici. La distinzione fondamentale proposta è che il prompt engineering è un approccio **stateless** — ogni interazione è indipendente — mentre il context engineering permette di costruire sistemi AI **stateful**, capaci di mantenere contesto coerente attraverso interazioni multiple e task complessi.

Il cuore della discussione riguarda come le tecniche classiche di prompt engineering (assegnazione di ruoli, istruzioni esplicite) stiano diventando meno efficaci man mano che i modelli maturano. L'expertise di dominio diventa invece cruciale: capire il problema da risolvere permette di curarele informazioni da fornire al modello in modo strategico, evitando sia l'information overload sia la perdita di contesto critico. I workflow basati su **skill** — dove i processi che hanno funzionato vengono catturati come unità riusabili — permettono ai team di scalare l'utilizzo dell'AI senza dover re-derivare soluzioni ogni volta.

La gestione della memoria è un tema centrale dell'episodio. Polak distingue tra memoria a breve termine (requisiti del task immediato, contesto della sessione corrente) e memoria a lungo termine (knowledge durabile, repository di skill, storico decisionale). Questa distinzione permette di ottimizzare sia l'accuratezza delle risposte sia i costi operativi, evitando di caricare nel contesto informazioni non pertinenti all'interazione corrente.

Le architetture multi-agente event-driven rappresentano il terreno applicativo principale: sistemi dove più agenti specializzati collaborano automatizzando pipeline complesse attraverso cicli di sviluppo software, sistemi di ticketing, e pipeline di data enrichment. In questo contesto, Apache Kafka e Apache Flink fungono da backbone per il context enrichment in tempo reale, abilitando applicazioni come triage automatico del backlog, analisi della qualità del codice, e suggerimento automatico di soluzioni — tutti casi d'uso dove la statefulness del contesto è determinante per la qualità dei risultati.

RAG (Retrieval-Augmented Generation) viene trattato come uno dei tanti tool disponibili in workflow agentici, particolarmente prezioso per la discovery e l'arricchimento di dati organizzativi distribuiti. La visione finale è che la creatività resti la skill umana essenziale: l'AI accelera l'esecuzione, ma la definizione degli obiettivi ambiziosi e il design dei sistemi rimangono prerogativa umana.
