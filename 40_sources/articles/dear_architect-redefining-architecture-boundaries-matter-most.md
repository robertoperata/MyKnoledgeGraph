---
tags:
  - architecture
  - ai
  - autonomous-systems
  - agentic
  - boundaries
type: article
author: Dear Architects
source: https://www.infoq.com/podcasts/redefining-architecture-boundaries-matter-most/
date: 2026-04-12
---

# Ridefinire l'Architettura: Perché i Confini Contano più di Tutto

## Sunto

In questo podcast (InfoQ, marzo 2026), Jesper Lowgren argomenta che l'AI generativa non è una semplice estensione dell'automazione tradizionale: è un cambiamento di paradigma che richiede un approccio architetturale radicalmente diverso. La distinzione fondamentale è tra **automazione** (esecuzione di passi predeterminati) e **autonomia** (comportamento emergente e non deterministico). "Inserire l'AI nei flussi procedurali" è l'errore più comune — autonomia e logica procedurale sono "olio e acqua".

Il cambio di paradigma centrale è il passaggio dalla *definizione di passi* alla *definizione di confini*. Un architetto che lavora con sistemi autonomi deve specificare cosa gli agenti *non possono* fare, strutturando sette dimensioni: scope, obiettivi, autorità, diritti decisionali, policy, rischio e ontologia/semantica. Questi confini diventano il meccanismo di governance, non le regole procedurali.

Lowgren identifica cinque livelli di maturità per i sistemi AI autonomi: dal Level 1 (assistenti AI ad hoc) al Level 3 (multi-agent con autonomia piena), con il salto al Level 3 descritto come un "passo davvero grande" che richiede nuovi linguaggi di design e un nuovo operating model. Un aspetto critico a questo livello è la **sicurezza ontologica**: se cinque agenti interpretano diversamente il termine "completato", il sistema "deriva e allucinera immediatamente". Ontologie condivise e meccanismi di context-sharing diventano misure di sicurezza strutturali.

Un pattern innovativo descritto nel podcast è il *processo di design LLM-assistito*: invece di lavagne e post-it, i team definiscono prima i confini in modo completo, poi lasciano che un LLM progetti i processi *all'interno* di quei confini. In un caso reale questo approccio ha generato automaticamente 27-33 agenti, validati successivamente tramite testing adversariale piuttosto che documentazione tradizionale.

Il messaggio pratico è chiaro: governance e design devono essere "uniti all'anca" fin dall'inizio — non si può aggiungere la governance a posteriori su sistemi autonomi in produzione. I team devono inoltre valutare onestamente la propria maturità organizzativa: la maggior parte opera ai livelli 1-2 e saltare direttamente al multi-agent autonomo senza fasi intermedie crea rischi insostenibili.
