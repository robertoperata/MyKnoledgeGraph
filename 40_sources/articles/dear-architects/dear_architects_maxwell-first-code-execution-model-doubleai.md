---
tags:
  - ai
  - code-optimization
  - formal-verification
  - performance
  - runtime-optimization
feature:
type: article
author: doubleAI
source: https://heymaxwell.ai/
date: 2026-11-01
---

# Maxwell — Il Primo Modello di Esecuzione del Codice di doubleAI

## Sunto

Maxwell è il primo **Code Execution Model (CEM)** di doubleAI, un sistema AI che ottimizza i programmi mentre sono in esecuzione, senza richiedere modifiche al codice sorgente. Si posiziona come complemento ai coding agent: mentre questi ultimi scrivono il codice, Maxwell si occupa di farlo girare in modo efficiente sull'hardware disponibile. Il sistema è attualmente in early access.

L'architettura di Maxwell si colloca tra il codice applicativo e l'hardware. Osserva un programma in esecuzione, identifica i colli di bottiglia e genera versioni più veloci del machine code, adattandole alla CPU o GPU specifica e ai dati effettivi che fluiscono nel sistema. Il codice sorgente rimane invariato: Maxwell emette nuovi binari ottimizzati mantenendo la semantica originale del programma.

La caratteristica tecnica più rilevante è la **verifica formale**: ogni trasformazione del codice è accompagnata da una prova machine-checked che garantisce l'equivalenza semantica con il codice originale. Questo distingue Maxwell da un tradizionale compiler optimizer o profiler-guided optimization: invece di affidarsi ai test per verificare la correttezza, le trasformazioni vengono provate matematicamente. doubleAI dichiara di aver formalizzato Python, incluse le sue caratteristiche dinamiche, per poter enunciare e dimostrare affermazioni di equivalenza su di esso. I blocchi di prova sono componibili e riutilizzabili, permettendo di costruire prove complesse da componenti più semplici.

Le ottimizzazioni applicate da Maxwell includono riscrittura di algoritmi (non solo istruzioni), specializzazione del codice in base al comportamento osservato a runtime, e re-ottimizzazione iterativa dove ogni versione migliorata diventa il punto di partenza per la successiva. Maxwell gestisce sia codice CPU che GPU: può sostituire codice Python con codice di basso livello ottimizzato, e kernel PyTorch con versioni sintonizzate per la GPU presente. Se una pre-condizione che ha motivato un'ottimizzazione dovesse venire violata, il sistema torna automaticamente al codice originale.

I risultati sperimentali riportati sono significativi: 38× speedup su data processing pipeline, 4.25× su AI inference kernel, 1.87× su generazione video LTX-2.5 4K, 2.13× su trascrizione Whisper su RTX 4090. Su SOL-ExecBench Blackwell kernels, Maxwell ha superato i baseline PyTorch di NVIDIA nel 90% dei casi dopo un solo giorno di ricerca. Questi risultati si riferiscono a configurazioni specifiche e potrebbero non generalizzarsi direttamente ad altri contesti.
