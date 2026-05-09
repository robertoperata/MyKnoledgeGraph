---
title: Agentic Patterns
type: concept
tags:
  - thread4-ai
sources:
  - "[[vercel_agent-responsibly]]"
  - "[[AI Talks - Lada Kesseler Augmented Coding]]"
  - "[[Claude Code and Large-Context Reasoning]]"
  - "[[tessl_context-development-lifecycle]]"
  - "[[youtube_ai-coding-workflow-matt-pocock]]"
updated: 2026-05-09
related:
  - "[[concepts/context-management]]"
  - "[[concepts/architectural-governance]]"
  - "[[concepts/harness-engineering]]"
  - "[[concepts/spec-driven-development]]"
  - "[[concepts/autonomous-systems-architecture]]"
  - "[[concepts/deep-modules]]"
  - "[[patterns/testing-pyramid]]"
---

# Agentic Patterns

## Definizione

Pattern ricorrenti nell'uso efficace di agenti AI per il coding e la produzione di software. La distinzione fondamentale è tra **relying** (delegare il giudizio all'agente) e **leveraging** (usare l'agente mantenendo piena ownership).

## Pattern fondamentali (Lada Kesseler)

### Anti-pattern: Context Rot
Il contesto si degrada nel tempo. Soluzione: creare `knowledge.md` con le risposte buone, ricominciare conversazioni nuove con il contesto rilevante.

### Anti-pattern: Distracted Agent
L'agente si disperde su troppi task contemporaneamente.

### Pattern: Focused Agent
Scope più piccolo = risultati migliori. **Una cosa alla volta.** Documenti separati per stile (`code_style.md`), processo (`tdd_process.md`).

### Pattern: Knowledge Composition
Comporre contesti diversi: `CLAUDE.md` a livello utente o progetto per le ground rules.

### Pattern: Semantic Zoom
- **Zoom in**: "tell me more about", "how do you write", "explain better"
- **Zoom out**: "make it shorter", "describe architecture at high level", "give me a TLDR"

### Pattern: Noise Cancellation
Combattere la verbosità default degli LLM: dare istruzioni esplicite per la brevità, eliminare senza pietà (`delete mercilessly`).

### Pattern: Knowledge Checkpoint
`save → commit → implement`. Il commit git è il checkpoint: permette di tornare a uno stato noto.

### Pattern: Parallel Implementations
Esplorare più implementazioni in parallelo prima di scegliere.

## Relying vs Leveraging (Vercel)

> "Il futuro appartiene agli ingegneri che mantengono un giudizio spietato su ciò che rilasciano, non a quelli che generano più codice."

| Approccio | Comportamento |
|---|---|
| **Relying** | Ship se i test passano; PR enormi che i reviewer non capiscono davvero |
| **Leveraging** | Piena ownership, comprensione dell'impatto in produzione, prontezza per un incident |

Test pratico: *saresti a tuo agio nell'essere responsabile di un incident legato a questo codice?*

## Tre linee di difesa per deployment sicuro

1. **Self-driving deployments**: canary release + rollback automatico
2. **Continuous validation**: load test e chaos experiment come pratica continua
3. **Executable guardrails**: la conoscenza operativa codificata come strumenti (non documentazione)

## Context Window e Large-Context Reasoning

La context window non è FIFO: i LLM mantengono più strettamente il materiale iniziale e quello più recente. Il materiale in mezzo è più soggetto all'eviction.

Implicazione pratica: le istruzioni critiche (`CLAUDE.md`, system prompt) vanno all'inizio; i dettagli rilevanti al task corrente vanno verso la fine della conversazione.

## Multi-Agent Architecture (Spotify Ads)

Pattern da Spotify per sistemi multi-agent:

**Tool Grounding**: i modelli LLM hanno accesso strutturato a dati reali tramite function calling — gli agenti ragionano su *cosa* fare, i tool forniscono i *fatti*. Previene le allucinazioni.

**Separazione intent/esecuzione**: gli agenti interpretano l'intenzione di business e orchestrano dinamicamente le chiamate — coerenza su superfici diverse (UI, Salesforce, Slack) senza duplicare la logica.

**Prompt engineering come software engineering**: i prompt sono versioned, testati e iterati con la stessa disciplina del codice. Piccole variazioni di formulazione impattano significativamente la consistenza dell'output.

**Regola per i confini degli agenti**: un agente per skill distinta o sorgente di dati. Troppi agenti → overhead di coordinazione; troppo pochi → prompt monolitici.

Risultati Spotify: tempo creazione media plan 15-30min → 5-10sec; 20+ campi di form → 1-3 messaggi in linguaggio naturale.

---

## Workflow AI-Human: il framework di Matt Pocock

Un workflow completo che mette insieme tutti i pattern agentic in un sistema coerente, partendo da due vincoli fondamentali degli LLM: la smart zone/dumb zone e la natura smemorata del modello.

### Human-in-the-loop vs AFK

Due categorie di task nell'era AI:

| Categoria | Caratteristiche | Esempi |
|---|---|---|
| **Human-in-the-loop** | Richiede un umano che risponda, valuti, decida | Grilling session, review PRD, QA, code review |
| **AFK (Away From Keyboard)** | L'umano può essere assente | Implementazione, automated review, migrate DB |

**Regola chiave:** la pianificazione è sempre human-in-the-loop — non può essere automatizzata perché richiede decisioni che solo un umano può prendere. L'implementazione può diventare completamente AFK se il backlog è ben preparato.

Metafora del **day shift e night shift**: la pianificazione è il day shift umano (preparare il backlog). L'implementazione è il night shift dell'AI (esecuzione AFK).

### Grilling session (design concept)

**Trigger:** arriva un'idea o un brief.

**Problema:** gli allineamenti silenziosi — quando si passa direttamente all'implementazione, l'AI (e spesso anche il developer) fa assunzioni implicite che portano a divergenze costose da scoprire tardi.

**Soluzione:** la **grill me skill** — un prompt che chiede all'AI di intervistare il developer su ogni aspetto del piano, camminando lungo ogni ramo dell'albero delle decisioni, una domanda alla volta, con la propria raccomandazione:

```
"Interview me relentlessly about every aspect of this plan
until we reach a shared understanding. Walk down each branch
of the decision tree resolving dependencies one by one.
For each question, provide your recommended answer.
Ask the questions one at a time."
```

Il concetto teorico è il **design concept** di Frederick Brooks (*The Design of Design*): l'idea condivisa tra tutti i partecipanti al progetto. Non si ha bisogno di un piano — si ha bisogno di essere sulla stessa lunghezza d'onda dell'AI.

**Output:** ~25k token di conversazione gold che documenta il design concept. Non va mai compattato — rimane come asset. Può essere alimentata con trascrizioni di meeting con domain expert.

### PRD come destination document

Una volta raggiunto l'allineamento, si crea il **PRD (Product Requirements Document)** che descrive *dove* si vuole arrivare, non *come* arrivarci.

Struttura: problem statement, solution overview, user stories, implementation decisions, testing decisions, **out of scope** (essenziale per la definition of done).

**Doc Rot:** i PRD non vanno lasciati nel repo a lungo termine. Dopo un mese, il codice è cambiato abbastanza da rendere il PRD fuorviante per l'AI che esplora il repo. Preferibile marcarli come closed nelle GitHub issues invece di tenerli come file aperti.

### Kanban board con DAG — Vertical Slices

Il PRD descrive la destinazione. Per arrivarci serve un piano di viaggio: un **Kanban board di issue indipendenti con relazioni di blocking**.

Il motivo per preferire il Kanban board a un piano sequenziale è la **parallelizzazione**: un piano sequenziale può essere eseguito da un solo agente. Un board con dependency graph (DAG — Directed Acyclic Graph) permette di avviare agenti in parallelo su issue indipendenti.

Ogni issue è classificata come **AFK** o **human-in-the-loop** e ha blocking relationships esplicite.

**Vertical slices vs horizontal coding** (dal *Pragmatic Programmer* — traceable bullets):

L'AI tende a codare **orizzontalmente** — prima tutto il database layer, poi tutto l'API layer, poi tutto il frontend. Il risultato è che non si ha un sistema integrabile e testabile fino alla fine.

La soluzione è codare **verticalmente**: ogni issue attraversa tutti i layer necessari e produce qualcosa di funzionante e visibile alla sua conclusione.

> Esempio: invece di "crea il gamification service" (orizzontale), la prima issue diventa "award points for lesson completion, visible on dashboard" — include schema change, service, e rendering minimo sul frontend.

### AFK implementation loop (Ralph pattern)

Una volta preparato il backlog, l'implementazione è completamente delegabile.

Pattern di esecuzione:
1. Carica tutte le issue dal backlog
2. Prende gli ultimi N commit per contesto
3. Avvia l'agente in modalità autonoma

Il **Ralph prompt** lavora sul backlog con priorità: critical bug fixes → development infrastructure → traceable bullets → polishing/refactors. Per ogni task: esplora il repo, usa TDD, esegue i feedback loop. Se non ci sono più issue AFK, termina.

### TDD come feedback loop strutturale per AI

> "Se la tua codebase non ha feedback loop, non otterrai mai output decenti dall'AI. Il soffitto dei tuoi feedback loop è il soffitto dell'AI."

Il TDD non è opzionale per l'AI — è strutturale:

- **Senza TDD:** l'AI scrive l'implementazione e poi i test. I test tendono a essere shallow o a "barare" (testano il comportamento esatto del codice, non il comportamento inteso).
- **Con TDD (red-green-refactor):** l'AI scrive un test failing prima, poi l'implementazione. È molto più difficile barare perché deve instrumentare il codice *prima* di scriverlo.

**Automated review:** dopo l'implementazione, si può far rivedere il codice da un agente separato — in una **nuova context window** (non alla fine della sessione di implementazione, dove si è nella dumb zone). Modello più capace per il review, più veloce/economico per l'implementazione.

### Push vs Pull per gli standard di codice

**Push:** le istruzioni vengono sempre inviate all'agente (es. `CLAUDE.md`). Costo fisso in token. Utile per vincoli che l'agente deve sempre rispettare.

**Pull:** le istruzioni stanno nel repo con un header descrittivo, e l'agente le recupera quando ne ha bisogno (skills/slash commands). Costo zero se non invocate.

Regola pratica:
- **Implementatore** → coding standards disponibili via **pull** (recupera se ha dubbi)
- **Reviewer automatico** → coding standards inviati via **push** (deve averli sempre per fare confronti)

### Sandcastle — Parallelizzazione degli agenti

Per eseguire issue in parallelo in modo strutturato:
1. Un **planner agent** analizza il backlog, identifica le issue parallelizzabili rispettando le blocking relationships
2. Per ogni issue, crea una sandbox (Git worktree + Docker container) e avvia un **implementer agent**
3. Gli agenti producono commit su branch isolati
4. Un **reviewer agent** per ciascuno e un **merger agent** che mergia i branch e risolve eventuali conflitti

La parallelizzazione è strutturalmente sicura perché le issue sono progettate per essere indipendenti (DAG).

---

## Connessioni

- [[concepts/context-management]] — la gestione del contesto è il tema centrale dell'era agentica
- [[concepts/architectural-governance]] — governance dichiarativa applicabile anche agli agenti
- [[concepts/harness-engineering]] — i sistemi intorno agli agenti per garantire la qualità del codice generato
- [[concepts/spec-driven-development]] — SDD come paradigma per governare i coding agent; il PRD è una forma di spec
- [[concepts/autonomous-systems-architecture]] — architettura dei confini per sistemi multi-agent autonomi
- [[concepts/deep-modules]] — prerequisito architetturale: codebase AI-friendly con feedback loop efficaci
- [[patterns/testing-pyramid]] — TDD come feedback loop strutturale per l'AI; vertical slices → component test
- [[technologies/kubernetes]] — deployment sicuro con canary release è infrastruttura K8s
