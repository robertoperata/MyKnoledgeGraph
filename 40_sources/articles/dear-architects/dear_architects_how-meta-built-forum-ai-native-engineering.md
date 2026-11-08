---
tags:
  - ai-native-engineering
  - ai-agents
  - software-engineering
  - verification
  - mobile-development
feature:
type: article
author: Meta Engineering
source: https://engineering.fb.com/2026/10/15/ml-applications/how-meta-built-forum-ai-native-engineering
date: 2026-11-08
---

# How Meta Built Forum With AI-Native Engineering

## Sunto

Forum è la nuova app standalone di Meta per le conversazioni approfondite nei Facebook Groups. Un team piccolo l'ha costruita da zero in otto settimane, trattando un agente AI come partecipante attivo nel processo ingegneristico — non come un modo più veloce di digitare codice. L'articolo descrive come il team ha adattato codebase, tooling e modello operativo per l'esecuzione agentiva.

La distinzione fondamentale del modello è tra **human-in-the-loop** (l'ingegnere guida ogni step e l'AI assiste) e **human-on-the-loop** (l'agente esegue il ciclo iterate-and-test mentre l'ingegnere imposta gli obiettivi, monitora e approva). Quest'ultimo è il target operativo del team. La metrica chiave non è il volume di codice prodotto, ma gli **outcome verificati e pronti per la produzione**: ogni modifica deve buildare, passare i test e sopravvivere alla review.

Il sistema di verifica è stato progettato come parte integrante del loop agentivo: build status, unit test e UI spec matching forniscono segnali pass/fail che l'agente usa per auto-validare il proprio lavoro prima che un ingegnere lo revisioni. Ogni modifica porta con sé i risultati di build, test e sessioni di verifica registrate. La trustworthiness del verifier è il limite della sicurezza: più il sistema di verifica è robusto, più lavoro può essere delegato al loop in sicurezza.

Un elemento distintivo è il **simulated user testing**: agenti di navigazione interagiscono con l'app live come farebbe un utente umano, leggendo la schermata e segnalando regressioni. Questo ha permesso di eseguire routinariamente stress test (scrolling aggressivo, navigazione rapida, caricamento ripetuto di media) su ogni build. In un caso, un agente ha rilevato un problema di memoria, isolato il repro, proposto una fix e verificato la soluzione — riducendo la crescita di memoria e i page fault circa del 50% prima del lancio.

L'operatività post-lancio è anch'essa agentiva: gli agenti analizzano i feedback degli utenti, investigano possibili regressioni e passano i findings agli ingegneri, che decidono il passo successivo. Questo chiude il ciclo dal deployment al triage.

## Modello operativo

| Componente | Descrizione |
|------------|-------------|
| **Rules** | Istruzioni permanenti del progetto. Ogni errore ripetuto 2+ volte viene codificato come regola per tutti |
| **Skills** | Convenzioni del team pacchettizzate come step riutilizzabili |
| **Agents** | Verifiche automatizzate che prima richiedevano review manuale |
| **Commands** | Workflow che concatenano skills e agents (complexity assessment, bug repro, planning, PR creation, review) |
| **Hooks** | Trigger automatici (es. linter dopo ogni turn agentivo, validazione test prima della PR) |

## Decisioni architetturali chiave

- **Leggibilità del codebase**: pattern semplici e pezzi chiave modularizzati. Dove il riuso era impraticabile, la squadra ha ricostruito da zero, privilegiando il throughput agentivo sul riuso.
- **Tools over descriptions**: il principale failure mode era la perdita di orientamento degli agenti nell'app. La soluzione è stata fornire strumenti per leggere la view hierarchy live, listare gli elementi a schermo e cercare i view path prima di agire.
- **Build-level isolation**: un build flavor separato per il sandbox condivide la codebase e il design system di produzione, ma compila il codice prototipo separatamente via plugin/socket architecture. Questo permette prototyping sicuro senza rischi di crash in produzione.

## Insight operativo

La qualità dell'output dipende più dal **contesto** che dalla formulazione del prompt. Dare agli agenti gli stessi riferimenti, query di esempio e design doc che ricevono i nuovi ingegneri migliora notevolmente la qualità delle investigazioni.
