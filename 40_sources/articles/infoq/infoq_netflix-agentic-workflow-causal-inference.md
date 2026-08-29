---
tags:
  - ai-agents
  - causal-inference
  - machine-learning
  - open-source
  - actor-critic
feature:
type: article
author: InfoQ
source: https://www.infoq.com/news/2026/08/netflix-oci-agent/
date: 2026-08-29
---

# Netflix Open-Sources Agentic Workflow for Causal Inference

## Sunto

Netflix ha rilasciato in open source un workflow agentico per l'Observational Causal Inference (OCI), disponibile su GitHub nel repository Netflix-Skunkworks/oci-agent. Il sistema mira a ridurre il toil analitico nelle analisi causali: dati osservazionali e un piano d'analisi umano in input, il workflow produce automaticamente stime di causalità, report e suggerimenti sui passi successivi. Le attività automatizzate includono la sensitivity analysis e il tracking di iterazioni multiple, lasciando all'utente i task di alto livello come la formulazione delle domande e la valutazione dei risultati.

L'architettura utilizza un loop actor-critic con due agenti distinti. L'**Actor Agent** esegue il piano di analisi: produce specifiche, parametrizza i notebook e lancia i calcoli. Il **Critic Agent** revisiona gli output, assegna valutazioni di soddisfazione (not_satisfactory, satisfactory_with_caveats, fully_satisfactory) e raccomanda raffinamenti. Il ciclo itera fino al raggiungimento di una qualità sufficiente, emulando il processo di revisione umana.

Il workflow si basa sul concetto di "target trial emulation": l'agente identifica il design ottimale di un A/B test per rispondere alla domanda di ricerca a partire da dati osservazionali, aggirando l'impossibilità pratica di eseguire esperimenti controllati su ogni fenomeno. Netflix ha valutato il sistema sul dataset dell'ACIC (Atlantic Causal Inference Conference) ottenendo risultati "competitive" rispetto ai benchmark esistenti.

Un caso d'uso reale riguarda l'analisi dell'impatto del tipo di intrattenimento sulla retention degli utenti. Il workflow agentico ha prodotto stime circa il 75% inferiori alla baseline, identificando problematiche come early adopter bias e placebo test falliti — insight che un semplice LLM senza il loop critico aveva mancato. Netflix sottolinea la trasparenza come principio progettuale: ogni passo è ispezionabile, con agenti che pubblicano piani, specifiche, grafici e notebook accessibili all'utente umano.

Il sistema include playbook per scenari analitici comuni e un meccanismo di "process audit with human oversight" che rende esplicit ogni decisione intermedia. L'approccio dimostra come i workflow agentici possano essere applicati non solo alla generazione di codice ma a task scientifici complessi, mantenendo il controllo umano sui decision point critici mentre si automatizzano le operazioni ripetitive e soggette a errori.
