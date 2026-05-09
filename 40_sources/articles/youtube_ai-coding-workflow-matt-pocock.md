---
tags:
  - ai
  - agent
  - workflow
  - software-engineering
  - tdd
  - architecture
type: article
author: Matt Pocock
source: https://www.youtube.com/watch?v=-QFHIoCo-Ko
date: 2026-05-09
---

# Full Walkthrough: Workflow for AI Coding

**Speaker:** Matt Pocock (TypeScript educator, creatore di Total TypeScript)  
**Evento:** Workshop live (conferenza non specificata)  
**Durata:** 1:36:30  
**Lingua originale:** inglese

---

## Sintesi

Matt Pocock presenta il suo workflow completo per lavorare con agenti AI nel software development, costruito attorno a una tesi centrale: **i principi fondamentali dell'ingegneria del software funzionano anche con l'AI**. Non è un approccio "specs to code" dove si ignora il codice — è un approccio dove il codice rimane il campo di battaglia e la comprensione condivisa con l'AI è il prerequisito.

Il workshop parte da due vincoli fondamentali degli LLM — la **smart zone/dumb zone** e la natura smemorata del modello — e costruisce un intero workflow su di essi: grilling session per raggiungere allineamento, PRD come documento di destinazione, Kanban board con vertical slices come piano di viaggio, loop di implementazione AFK, TDD come feedback loop obbligatorio, e deep modules come struttura architetturale che rende possibile tutto il resto.

La parte più preziosa del talk non è la sequenza delle fasi, ma la filosofia sottostante: **separa il lavoro human-in-the-loop dal lavoro AFK**, mantieni il controllo del tuo stack, e costruisci la codebase in modo che l'AI possa lavorarci bene. Una codebase con feedback loop deboli produce output AI scadenti — il soffitto dell'AI è il soffitto della tua codebase.

---

## Vincoli fondamentali degli LLM

### Smart zone e dumb zone

Il concetto viene da Dex Hardy (Human Layer): ogni LLM ha una **smart zone** (inizio della conversazione, attention relationships poco stressate) e una **dumb zone** (dopo ~100k token, il modello degrada progressivamente).

L'attenzione scala quadraticamente: ogni token aggiunto crea nuove relazioni con tutti i token precedenti. Il risultato pratico è che intorno ai 100k token — indipendentemente dalla context window dichiarata (200k, 1M) — il modello inizia a fare scelte sbagliate.

> La 1M context window di Claude Code non ha ampliato la smart zone: ha semplicemente aggiunto più dumb zone. È utile per il retrieval (cercare cose in testi lunghi), non per il coding.

Implicazione pratica: **le task vanno dimensionate per stare nella smart zone**. Non si può fare una task enorme e sperare che l'AI la gestisca per intero.

### LLM come il protagonista di Memento

Gli LLM dimenticano continuamente. Ogni sessione ricomincia dallo stesso stato base (system prompt). Questo sembra un limite, ma Pocock lo vede come un **vantaggio strutturale**: lo stato iniziale è sempre lo stesso, sempre predictable.

Due strategie per gestire il contesto pieno:
- **Clear**: si azzerano tutti i token, si torna al system prompt. Lo stato è identico ogni volta.
- **Compact**: si comprime la conversazione in un sommario e si continua. Il problema è che il sommario porta "sedimento" — informazioni imprecise o obsolete che influenzano la sessione successiva.

Pocock preferisce nettamente **clear** a **compact**: massimizza la predictability e, se il workflow è ottimizzato per ricominciare pulito, il clear ha un costo vicino a zero.

---

## Il workflow: fase per fase

### Fase 1 — Grilling session (human in the loop)

**Trigger:** arriva un'idea o un brief (es. un messaggio Slack da un cliente).

Il problema che vuole risolvere è quello degli **allineamenti silenziosi**: quando si passa direttamente all'implementazione, l'AI (e spesso anche lo sviluppatore) fa assunzioni implicite che portano a divergenze costose da scoprire tardi.

La soluzione è la **grill me skill**: un prompt che chiede all'AI di intervistare lo sviluppatore relentlessly su ogni aspetto del piano, camminando lungo ogni ramo dell'albero delle decisioni, una domanda alla volta, con la propria raccomandazione per ogni domanda.

```
"Interview me relentlessly about every aspect of this plan
until we reach a shared understanding. Walk down each branch
of the decision tree resolving dependencies one by one.
For each question, provide your recommended answer.
Ask the questions one at a time."
```

Il concetto teorico dietro la skill è quello di **design concept** di Frederick P. Brooks (*The Design of Design*): l'idea condivisa tra tutti i partecipanti al progetto. Non si ha bisogno di un piano — si ha bisogno di essere sulla stessa lunghezza d'onda dell'AI.

**Output della grilling session:** ~25k token di conversazione gold che documenta il design concept raggiunto. Questo non viene mai compattato — rimane come asset.

La grilling session è **non automatizzabile**: richiede un umano che risponda. È human-in-the-loop per definizione. Può durare a lungo (40, 80, 100 domande), ma il tempo è ben speso perché elimina ambiguità prima dell'implementazione. Può essere alimentata anche con trascrizioni di meeting con domain expert o stakeholder.

---

### Fase 2 — PRD: documento di destinazione (human review)

Una volta raggiunti l'allineamento nella grilling session, si crea il **PRD (Product Requirements Document)** usando la `write a PRD` skill.

Struttura del PRD:
- Problem statement (il problema dell'utente)
- Solution overview
- User stories (es. 18 storie per una feature gamification)
- Implementation decisions prese durante il grilling
- Testing decisions
- **Out of scope**: lista esplicita di ciò che non viene fatto — essenziale per la definition of done

Il PRD è il **documento di destinazione** (*destination document*): descrive dove si vuole arrivare, non come arrivarci.

**Importante:** Pocock non legge il PRD dopo averlo generato. Il motivo è che l'LLM è ottimo nella summarization, e dopo il grilling il design concept condiviso è già nella conversazione. Leggere il PRD non aggiunge nulla — si sta solo verificando la capacità di summarization dell'AI.

**Doc rot:** i PRD non vanno lasciati nel repo a lungo termine. Dopo un mese, il codice è cambiato abbastanza da rendere il PRD fuorviante per l'AI che esplora il repo. Pocock li marca come closed nelle GitHub issues invece di tenerli come file aperti.

---

### Fase 3 — Kanban board: documento di viaggio (human review)

Il PRD descrive la destinazione. Per arrivarci serve un piano di viaggio: **un Kanban board di issue indipendenti con relazioni di blocking**.

Il motivo per cui Pocock preferisce il Kanban board a un piano sequenziale (fase 1, fase 2, fase 3) è la **parallelizzazione**: un piano sequenziale può essere eseguito solo da un agente alla volta. Un Kanban board con dependency graph (DAG — Directed Acyclic Graph) permette di avviare agenti in parallelo su issue indipendenti.

La skill `PRD to issues` crea le issue come markdown file locali (o GitHub issues), con:
- Tipo di task: **AFK** (eseguibile da agente senza supervisione) o **human in the loop**
- Blocking relationships: quali issue devono essere completate prima

#### Vertical slices vs horizontal coding

Il concetto più importante di questa fase viene dal *Pragmatic Programmer*: **traceable bullets** (o vertical slices).

Il problema: l'AI tende a codare **orizzontalmente** — prima tutto il database layer, poi tutto l'API layer, poi tutto il frontend. Il risultato è che non si ha un sistema integrabile e testabile fino alla fine della fase 3.

La soluzione è codare **verticalmente**: ogni issue deve attraversare tutti i layer necessari e produrre qualcosa di funzionante e visibile alla sua conclusione. Come le tracce fosforescenti sui proiettili anti-aerei: feedback immediato sulla traiettoria.

Esempio concreto dal workshop: invece di "crea il gamification service" (orizzontale), la prima issue diventa "award points for lesson completion, visible on dashboard" — che include schema change, service, e rendering minimo sul frontend.

---

### Fase 4 — Implementazione AFK (agente senza supervisione)

Questa è l'unica fase che può essere completamente delegata. Lo sviluppatore si allontana dalla tastiera.

Lo script `once.sh` nel repo del workshop mostra la meccanica essenziale:
1. Carica tutte le issue dal backlog (file markdown)
2. Prende gli ultimi 5 commit per contesto
3. Avvia Claude Code in modalità `--accept-edits` con il prompt dell'implementatore

Il **Ralph prompt** (chiamato così in riferimento al personaggio dei Simpson) lavora sul backlog:
- Priorità: critical bug fixes → development infrastructure → traceable bullets → polishing/refactors
- Per ogni task: esplora il repo, usa TDD per l'implementazione, esegue i feedback loops

Il pattern di esecuzione è un loop: ogni iterazione prende la prossima issue disponibile, la implementa, esegue i test, e marca la issue come done. Se non ci sono più issue AFK, termina.

---

### Fase 5 — TDD come feedback loop (essenziale per l'AI)

> "Se la tua codebase non ha feedback loop, non otterrai mai output decenti dall'AI. Il soffitto dei tuoi feedback loop è il soffitto dell'AI."

Il TDD non è optional per l'AI — è strutturale. Il motivo:

- Senza TDD, l'AI scrive l'intera implementazione e poi i test. I test tendono a essere shallow o a "barare" (testano il comportamento esatto del codice, non il comportamento inteso).
- Con TDD (red-green-refactor), l'AI scrive un test failing prima, poi l'implementazione che lo fa passare. È molto più difficile barare perché deve instrumentare il codice *prima* di scriverlo.

Il workflow di implementazione include sempre i feedback loop: NPM run test + NPM run type check. Se passa, la task è done. Se c'è un errore (es. type error TypeScript), l'AI lo risolve nella stessa sessione.

**Automated review:** dopo l'implementazione, prima che lo sviluppatore faccia QA, si può far rivedere il codice da un agente separato — in una **nuova context window** (non alla fine della sessione di implementazione, dove si sarebbe nella dumb zone). Pocock usa Opus per il review (più "intelligente") e Sonnet per l'implementazione (più veloce/economico).

---

### Fase 6 — QA e code review (human in the loop)

Il QA manuale è dove lo sviluppatore **impone il proprio gusto al codice**. Non è automatizzabile.

Pocock è esplicito su questo: chi prova ad automatizzare tutto (idea, QA, ricerca, prototype) produce app che "mancano di gusto" o semplicemente non funzionano come inteso. Il tocco umano è necessario.

Il QA produce nuove issue che vengono aggiunte al Kanban board. Il ciclo riprende. Il Kanban board può ricevere issue indefinitamente — è un backlog continuo, non un piano chiuso.

---

## Architettura della codebase: deep vs shallow modules

Ispirandosi a **John Ousterhout** (*A Philosophy of Software Design*), Pocock distingue:

**Shallow modules** — molti file piccoli, ognuno con poche responsabilità, molte dipendenze reciproche.
- Difficili da navigare per l'AI (deve tracciare manualmente il grafo delle dipendenze)
- Difficili da testare (dove si tracciano i confini dei test?)
- L'AI tende a produrre questo tipo di codebase se non viene guidata

**Deep modules** — pochi moduli con interfacce piccole e semplici, ma con molta logica interna.
- Facili da testare: si wrappa il test boundary attorno al modulo e si cattura molto comportamento
- L'AI può navigarli facilmente: vede l'interfaccia, non ha bisogno di tracciare tutto il grafo interno
- Lo sviluppatore mantiene il senso della codebase lavorando sulle interfacce, delegando l'implementazione interna

**Mental trick:** con i deep modules, lo sviluppatore può trattare i moduli come **gray boxes** — conosce il comportamento, il contratto dell'interfaccia, non necessariamente ogni dettaglio implementativo. Questo permette di muoversi velocemente delegando l'implementazione interna all'AI, mantenendo la padronanza dell'architettura.

La skill `improve codebase architecture` analizza il repo e propone candidati per approfondire i moduli, trovando cluster di file correlati che potrebbero essere testati come unità con un'interfaccia unica.

---

## Push vs Pull per gli standard di codice

Come fare in modo che l'AI rispetti gli standard del codice senza caricare tutto nel system prompt?

**Push:** le istruzioni vengono sempre inviate all'agente (es. contenuto del CLAUDE.md). Ha un costo fisso in token. Utile per vincoli che l'agente deve sempre rispettare.

**Pull:** le istruzioni stanno nel repo con un header descrittivo, e l'agente le recupera quando ne ha bisogno (skills/slash commands). Il costo in token è zero se non vengono invocate.

Regola di Pocock:
- **Implementatore** → coding standards disponibili via **pull** (l'agente li recupera se ha dubbi)
- **Reviewer automatico** → coding standards inviati via **push** (il reviewer deve averli sempre davanti per fare confronti)

---

## Sandcastle: parallelizzazione degli agenti

Pocock ha costruito **Sandcastle**, una libreria TypeScript per eseguire agenti in parallelo in modo strutturato.

Il flusso parallelo:
1. Un **planner agent** analizza il backlog, identifica le issue parallelizzabili (rispettando le blocking relationships)
2. Per ogni issue, crea una sandbox (Git worktree + Docker container) e avvia un **implementer agent**
3. Se gli agenti producono commit, avvia un **reviewer agent** per ciascuno
4. Un **merger agent** prende tutti i branch, li mergia, risolve eventuali conflitti di type/test

Ogni agente lavora su un branch isolato. La parallelizzazione è strutturalmente sicura perché le issue sono progettate per essere indipendenti (Kanban board con DAG).

---

## Human in the loop vs AFK

Un framework concettuale chiave del talk: **due categorie di task nell'era AI**.

| Categoria | Caratteristiche | Esempi |
|---|---|---|
| **Human in the loop** | Richiede un umano che risponda, valuti, decida | Grilling session, review PRD, QA, code review |
| **AFK (Away From Keyboard)** | L'umano può essere assente | Implementazione, automated review, migrate DB |

La pianificazione (idea → grilling → PRD → Kanban board) è sempre **human in the loop** — non può essere loopata perché richiede decisioni che solo un umano può prendere.

L'implementazione può diventare **completamente AFK** se il backlog è ben preparato.

Metafora del **day shift e night shift**: la fase di pianificazione è il day shift umano (preparare il backlog, prendere decisioni). La fase di implementazione è il night shift dell'AI (esecuzione AFK mentre l'umano dorme o fa altro).

---

## Punti chiave

- **Non esiste "specs to code"**: ignorare il codice mentre si lavora sulle specifiche porta al vibe coding. Il codice è il campo di battaglia — va tenuto presente durante tutta la pianificazione.

- **Il design concept precede il piano**: prima di avere un PRD o un piano, si deve raggiungere una shared understanding con l'AI. Il grilling session è il modo per farlo.

- **Smart zone: ~100k token**: dimensionare le task per stare in questo range. La 1M context window espande la dumb zone, non la smart zone.

- **Clear > compact**: lo stato iniziale pulito è predictable. Il sedimento del compact introduce rumore difficile da controllare.

- **Vertical slices > horizontal coding**: ogni issue deve essere visibile e testabile alla sua conclusione. L'AI tende naturalmente al coding orizzontale — va corretta esplicitamente.

- **TDD è strutturale per l'AI**: senza TDD, l'AI bara nei test. Con TDD, il feedback loop è integrato nell'implementazione.

- **Deep modules > shallow modules**: la qualità dei feedback loop è il soffitto dell'AI. I deep modules rendono i feedback loop più efficaci.

- **Doc rot è un rischio reale**: i PRD e la documentazione di pianificazione non vanno lasciati nel repo a lungo termine. Influenzano negativamente l'AI quando il codice è già cambiato.

- **Parallelizzazione richiede un DAG**: un piano sequenziale può essere eseguito da un solo agente. Un Kanban board con blocking relationships può essere eseguito da N agenti in parallelo.

- **Il QA è il luogo del gusto**: automatizzare tutto il processo produce slop. Il tocco umano nel QA è dove si impone la qualità.

---

## Citazioni notevoli

> "We all think AI is a new paradigm, but we forget that software engineering fundamentals — the stuff that's crucial for working with humans — also works super well with AI."  
> — Matt Pocock

> "Bad codebases make bad agents. If you have a garbage codebase, you're going to get garbage out of the agent working in that codebase."  
> — Matt Pocock

> "I didn't need an asset, I didn't need a plan — I needed to be on the same wavelength as the AI. That's a design concept."  
> — Matt Pocock (citando Frederick P. Brooks)

> "Planning has to be human in the loop. Has to be. Implementation can be AFK. Planning can't."  
> — Matt Pocock

> "The ceiling of your feedback loops is the ceiling of your AI. If your codebase doesn't have feedback loops, you're never ever going to get decent output from AI."  
> — Matt Pocock

> "QA is how I impose my opinions back onto the codebase, how I impose my taste. Teams trying to automate everything end up with apps that lack taste and are bad."  
> — Matt Pocock

> "With AI you're working harder than ever before, but knowing your codebase less well. Deep modules are my way to retain a sense of the codebase while preserving my sanity."  
> — Matt Pocock

---

## Il workflow completo (schema)

```
IDEA
  ↓
GRILLING SESSION          ← human in the loop
  ↓
[opzionale: ricerca, prototipo]
  ↓
PRD (destination doc)     ← human review
  ↓
KANBAN BOARD (issues)     ← human review
  ↓
IMPLEMENTAZIONE AFK       ← agente (loop/parallelo)
  ↓
AUTOMATED REVIEW          ← agente (nuova context window)
  ↓
QA + CODE REVIEW          ← human in the loop
  ↓
nuove issue nel Kanban → torna all'implementazione
```

---

## Riferimenti e risorse

- **Total TypeScript / AI Hero** ([totaltypescript.com](https://totaltypescript.com) / [aihero.dev](https://aihero.dev)) — corsi e articoli di Matt Pocock su TypeScript e AI workflow
- **Sandcastle** — libreria TypeScript di Pocock per loop di agenti paralleli con Docker sandbox e Git worktrees
- **Ralph Wiggum loop** — pattern di implementazione AFK basato su backlog di issue: l'agente sceglie la prossima issue, implementa, esegue test, itera
- **Grill Me skill** — prompt per intervistare relentlessly lo sviluppatore e raggiungere shared understanding prima della pianificazione
- **HDR Histogram / Dex Hardy (Human Layer)** — origine del concetto smart zone/dumb zone
- **Frederick P. Brooks** — *The Design of Design*: il concetto di "design concept" come idea condivisa tra i partecipanti
- **John Ousterhout** — *A Philosophy of Software Design*: deep modules vs shallow modules
- **The Pragmatic Programmer** — traceable bullets / vertical slices: feedback immediato attraverso tutti i layer del sistema
- **Martin Fowler** — *Refactoring*: "don't bite off more than you can chew", citato per le task piccole
- **BEADS framework** (Steve) — framework alternativo per gestione Kanban board con agenti, citato come interessante ma non testato da Pocock
