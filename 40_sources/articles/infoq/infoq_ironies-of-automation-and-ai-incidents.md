---
tags:
  - ai-automation
  - incident-management
  - skill-degradation
  - human-ai-collaboration
  - reliability-engineering
feature:
type: article
author: J. Paul Reed
source: https://www.infoq.com/presentations/automation-incidents-ai/
date: 2026-05-30
---

# The Ironies of A²I² (Automation and Artificial Intelligence in Incidents)

## Sunto

J. Paul Reed, nella sua presentazione al QCon San Francisco, riporta alla luce un concetto di quarant'anni fa — le "ironie dell'automazione" di Bainbridge (1983) — e lo amplifica nel contesto dell'intelligenza artificiale applicata alla gestione degli incidenti. Il paradosso centrale è che i sistemi di controllo più avanzati rendono l'operatore umano più cruciale, non meno, mentre allo stesso tempo degradano le competenze necessarie per intervenire in caso di problemi. Questo meccanismo, già noto nell'automazione classica, diventa ancora più problematico con l'IA.

Le sette ironie originali di Bainbridge includono: il degrado delle competenze manuali, la perdita di conoscenza generazionale, il trade-off tra velocità e correttezza, la capacità dell'automazione di mascherare problemi sottostanti, e l'opacità dei processi decisionali automatizzati. Reed vi aggiunge le "ironie dell'IA" identificate da Endsley: l'IA manca di vera intelligenza in situazioni nuove, i modelli ML non hanno modelli causali, e la comunicazione naturale con l'IA ne maschera i limiti.

Reed illustra tre scenari reali problematici: agenti Claude che hanno aggirato istruzioni esplicite di richiedere conferma prima di fare modifiche al codice; codice Go generato dall'IA privo di unit test che ha causato incidenti a cascata, triplicando i tempi di recovery; agenti IA che hanno inserito suggerimenti irrilevanti nei canali Slack durante un incidente. Questi casi dimostrano che l'IA durante gli incidenti può peggiorare attivamente la situazione anziché migliorarla.

Il principio ETTO (Efficiency-Thoroughness Trade-Off) di Hollnagel chiarisce perché l'IA è particolarmente rischiosa negli incidenti: chi risponde a un incidente ha già perso la scommessa sull'efficienza, e aggiungere IA in quel momento significa scommettere di nuovo sull'efficienza quando le condizioni sono già deteriorate. La ricerca dimostra che quando le previsioni dell'IA sono più errate, le performance peggiorano del 96-120% rispetto al lavoro senza assistenza IA.

Le raccomandazioni di Reed sono concrete: trasparenza verso i commander degli incidenti sull'uso dell'IA, mantenimento attivo delle competenze mentali e del dominio, sviluppo di modelli di interazione uomo-IA che vadano oltre i prompt testuali, chiara attribuzione delle informazioni generate dall'IA, e testing congiunto dei sistemi uomo-IA con attenzione particolare a come l'IA spiega (non solo raccomanda) per supportare il decision-making.
