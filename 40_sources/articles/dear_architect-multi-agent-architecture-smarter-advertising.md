---
tags:
  - ai
  - multi-agent
  - architecture
  - agentic
type: article
author: Dear Architects
source: https://engineering.atspotify.com/2026/2/our-multi-agent-architecture-for-smarter-advertising
date: 2026-04-12
---

# La Nostra Architettura Multi-Agent per la Pubblicità Intelligente (Spotify)

## Sunto

L'articolo degli ingegneri Spotify Pratik Rasam e Ralph Sylvain descrive come il team advertising abbia adottato un'architettura multi-agent per risolvere un problema strutturale: tre canali di acquisto (Direct, Self-Serve, Programmatic) condividevano la stessa infrastruttura backend ma reimplementavano separatamente la logica decisionale per budget allocation, selezione dell'inventario e ottimizzazione della copertura. Invece di costruire un altro servizio monolitico, Spotify ha scelto un layer di decisione agente capace di comprendere gli obiettivi dell'inserzionista e orchestrare le API esistenti in modo consistente tra tutti i canali.

L'architettura è composta da agenti specializzati costruiti su Google ADK 0.2.0 con Gemini 2.5 Pro su Vertex AI. Il **RouterAgent** analizza i messaggi in ingresso per determinare quali informazioni sono già presenti, evitando chiamate LLM superflue. Gli agenti resolver specializzati si occupano ciascuno di una dimensione distinta: **GoalResolverAgent** (mappatura degli obiettivi), **AudienceResolverAgent** (criteri di targeting), **BudgetAgent** (parsing di formati valutari multipli) e **ScheduleAgent** (interpretazione di riferimenti temporali relativi). Il **MediaPlannerAgent** genera raccomandazioni ottimizzate per gli ad set usando sette regole euristiche informate dai dati storici delle campagne.

Tre pattern architetturali emergono con chiarezza. Il primo è il *tool grounding*: i modelli LLM hanno accesso strutturato a dati reali tramite function calling, prevenendo le allucinazioni — gli agenti ragionano su cosa fare, i tool forniscono i fatti. Il secondo è la *separazione dell'intent layer dall'esecuzione*: gli agenti interpretano l'intenzione di business e orchestrano dinamicamente le chiamate, abilitando coerenza su superfici diverse (Spotify Ads Manager, Salesforce, Slack). Il terzo è il trattamento del *prompt engineering come software engineering*: i prompt sono versioned, testati e iterati con la stessa disciplina del codice di produzione — piccole variazioni di formulazione impattano significativamente la consistenza dell'output.

I risultati misurati sono notevoli: il tempo di creazione di un media plan passa da 15-30 minuti a 5-10 secondi, gli input richiesti scendono da 20+ campi di form a 1-3 messaggi in linguaggio naturale, e le raccomandazioni sono basate su migliaia di campagne storiche invece che sull'intuizione del singolo. La latenza di risposta è di 3-5 secondi.

Il principale insegnamento architetturale è la definizione dei confini degli agenti: troppi agenti aumentano l'overhead di coordinazione, troppo pochi creano prompt monolitici difficili da mantenere. La regola adottata da Spotify è un agente per skill distinta o sorgente di dati. Le direzioni future includono streaming delle risposte, raffinamento multi-turn, integrazione con A/B testing e fine-tuning su terminologia advertising specifica del dominio.
