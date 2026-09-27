---
tags:
  - ai-agents
  - context-engineering
  - llm
  - multi-agent-systems
  - benchmarking
feature:
type: article
author: Dear Architects
source: https://tech.olx.com/does-it-matter-how-you-feed-an-agent-its-context-ee1519586520
date: 2026-09-27
---

# Does it matter how you feed an agent its context?

## Sunto

Questo articolo del team engineering di OLX affronta una domanda empirica: cambia il modo in cui si fornisce il contesto a un agente AI i risultati che produce? L'autore parte dalla constatazione che esiste un ampio dibattito su come strutturare il contesto per gli agenti — tutto in un unico prompt, suddiviso in cartelle, o affidato a un orchestratore con delega — ma mancano misurazioni sistematiche per valutare quale approccio funzioni meglio.

La metodologia dell'esperimento è rigorosa: stesso modello, stesso task, stesso sistema di scoring. L'unica variabile è come gli agenti ricevono il loro contesto. Quattro configurazioni diverse vengono testate con tre run ciascuna per ridurre la variabilità statistica. Questo approccio scientifico è raro nel dibattito sull'AI engineering, dove le opinioni spesso prevalgono sui dati.

I risultati mettono in discussione l'assunzione implicita che "il contesto è contesto, indipendentemente da come lo consegni". Le differenze nelle configurazioni producono outcome diversi, suggerendo che la modalità di delivery del contesto influenza effettivamente le capacità dell'agente — sia in termini di qualità dei risultati che di costo computazionale. Questo ha implicazioni pratiche significative per chi progetta sistemi agentici e vuole ottimizzare il rapporto qualità/costo.

L'articolo è particolarmente rilevante per gli ingegneri che costruiscono feature basate su agenti e si trovano a dover scegliere un'architettura per il context feeding senza dati empirici su cui basarsi. Il fatto che venga da un team di produzione (OLX) anziché da un laboratorio di ricerca lo rende ancora più applicabile a scenari reali.

L'esperimento apre anche domande più profonde sulla natura del context engineering: non è sufficiente ottimizzare il contenuto del contesto, ma è necessario ottimizzare anche la sua struttura e modalità di delivery. Questo sposta il focus dall'ingegneria del prompt all'architettura del sistema.
