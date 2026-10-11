---
tags:
  - ai-sdlc
  - agentic-ai
  - software-development-lifecycle
  - atlassian
  - developer-productivity
feature:
type: article
author: Matthew Canham
source: https://www.atlassian.com/blog/ai-at-work/ai-sdlc-transformation-playbook
date: 2026-10-11
---

# The AI SDLC Transformation Playbook

## Sunto

L'articolo di Atlassian propone un framework per trasformare l'intero ciclo di vita dello sviluppo software (SDLC) in un'ottica AI-nativa, andando ben oltre la semplice adozione di coding agent. La tesi centrale è che man mano che la scrittura di codice diventa asincrona e multi-threaded grazie agli agenti, le fasi precedenti e successive al codice — pianificazione, design, review e manutenzione — diventano i principali colli di bottiglia e le maggiori opportunità di miglioramento.

L'approccio di Atlassian si fonda su due investimenti infrastrutturali. Il primo è il **context graph**: gli agenti hanno bisogno di contesto organizzativo connesso e permission-aware. Il "Teamwork Graph" di Atlassian attinge a oltre 50 sorgenti e ha prodotto risultati misurati: il 44% in più di accuratezza e il 48% in meno di token consumati rispetto agli agenti senza contesto strutturato. Il secondo investimento è la **misurazione**: serve un feedback quantitativo e qualitativo (tramite strumenti come DX) per separare i reali guadagni di produttività dall'hype.

Il playbook descrive la trasformazione workflow per workflow. Nella **pianificazione**, gli agenti sintetizzano le esigenze dei clienti e il contesto del codice in specifiche, con Jira Product Discovery che valuta le idee rispetto ai feedback reali. Nel **design**, il processo diventa un thread continuo con prototipi live; i designer possono contribuire direttamente al codice UI. Nello **sviluppo**, la qualità del codice generato dipende dalla qualità del contesto fornito all'AI, e la velocità dipende dall'orchestrazione degli agenti. Nella **review**, l'AI applica standard condivisi prima che i revisori umani esprimano il giudizio finale. Nella **manutenzione**, gli agenti gestiscono triage, risoluzione e documentazione del lavoro routine, escalando quando necessario.

I risultati riportati da Atlassian includono la riduzione del cycle time degli issue da 11,5 a 6,7 giorni e la risoluzione automatica del 51% delle potenziali vulnerabilità tramite loop agentivi in Jira. Il playbook si chiude con cinque raccomandazioni operative: ancorare tutto al contesto, investire presto nella misurazione, scegliere un workflow di partenza e andare in profondità prima di allargare, preservare la responsabilità umana per giudizio e governance, e costruire il modello operativo non solo il toolkit.
