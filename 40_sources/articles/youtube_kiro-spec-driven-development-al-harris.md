---
tags:
  - ai
  - agentic
  - developer-experience
  - architecture
  - testing
type: article
author: Al Harris
source: https://www.youtube.com/watch?v=HY_JyxAZsiE
date: 2026-01-09
---

# Spec-Driven Development: Agentic Coding at FAANG Scale and Quality

**Speaker:** Al Harris (Principal Engineer, Amazon — team Kiro)  
**Evento:** AI Engineer World's Fair  
**Durata:** 1:03:49  
**Lingua originale:** inglese

---

## Sintesi

Al Harris, uno dei fondatori del piccolo team (3-4 persone) che ha costruito **Kiro** — un IDE agentioco di Amazon, fork di VS Code — illustra la filosofia e i meccanismi del Spec-Driven Development così come implementato in Kiro. Il talk è presentato come una sessione pratica con live demo, e ha il merito di andare in profondità su dettagli di implementazione che la documentazione pubblica non copre.

Il problema che Kiro cerca di risolvere è quello della **scalabilità e riproducibilità** dell'AI coding: il vibe coding funziona per task semplici, ma dipende interamente dall'operatore che fornisce i guardrail giusti. SDD è pensato per rappresentare l'intero SDLC in modo che un agente possa lavorare in modo affidabile su problemi complessi con meno supervisione umana. La struttura è: **Requirements → Design → Tasks** (tre documenti markdown), ognuno dei quali viene prodotto e validato prima di procedere al successivo.

Il contributo tecnico più originale del talk è il formato **EARS** (Easy Approach to Requirements Syntax) per i requisiti: una sintassi strutturata in linguaggio naturale ("When... the system shall...") che ha due funzioni distinte. Prima, serve come specifica leggibile da umani e consumabile dall'agente. Seconda — e questa è la novità annunciata al lancio GA di Kiro — viene tradotta automaticamente in **property-based tests**: invarianti del sistema che vengono verificate continuamente. L'obiettivo dichiarato è chiudere il loop tra requisiti scritti e codice prodotto: se le proprietà derivate dai requisiti passano, c'è alta confidenza che il sistema sia stato implementato correttamente.

Un concetto fondamentale è la distinzione tra **spec** e **steering**. La spec descrive una feature o un'area problematica specifica ed è un artefatto *vivente*: non si accumula indefinitamente, ma viene mutata quando la feature evolve. Lo steering è il memory bank del progetto — documenti di contesto su convenzioni, come deployare, come fare commit, etc. — che fornisce il contesto permanente del progetto all'agente. L'architettura interna di Kiro non è puramente LLM: usa tecniche di **automated reasoning** neurosimbolico per operazioni deterministiche come la validazione dei requisiti (rilevamento di ambiguità, requisiti in conflitto), mentre l'LLM viene usato per la generazione.

Il talk affronta anche le limitazioni pratiche: ogni task viene eseguito in una sessione separata inizializzata con la spec (nessun contesto condiviso tra task), il che richiede spec ben scritte. La gestione del contesto è ottimizzata per il **prompt caching** (90-95% hit rate) piuttosto che per la compressione. Per codebase grandi, l'agente lavora meglio con alta coesione e separazione delle responsabilità — una conferma pratica del principio "good code is AI-readable code".

---

## Il workflow SDD in Kiro

```
Prompt utente
    │
    ▼
[Requirements] — formato EARS — accettazione criteri + user stories
    │
    ▼
[Design] — documenti di design revisionabili; può includere wireframe ASCII, pseudo-code
    │
    ▼
[Properties] — invarianti estratte automaticamente dai requirements EARS (PBT)
    │
    ▼
[Tasks] — task list atomici; ognuno con test case espliciti
    │
    ▼
[Esecuzione] — ogni task = nuova sessione, seed = spec + steering
```

### Spec come feature, non come progetto

Una spec rappresenta una **feature o area problematica**, non il progetto intero. Con l'uso, il progetto accumula molte spec, una per feature. Quelle relative a feature stabili vengono mantenute come living docs; quelle di esperimenti possono essere cancellate.

---

## Il formato EARS (Easy Approach to Requirements Syntax)

EARS è una sintassi strutturata di linguaggio naturale per i requisiti:

```
When <condizione>, the system shall <comportamento>.
```

Esempio reale dal talk:
```
When the conversation is validated, the system shall verify
that each user input is either non-empty content or tool responses.
```

**Perché EARS invece di linguaggio naturale libero?**

La struttura permette di processare i requisiti con **automated reasoning non-LLM** invece di affidarsi al solo LLM. Questo dà risultati più deterministici per operazioni come:
- Rilevamento di **ambiguità** nei requisiti
- Identificazione di **requisiti in conflitto**
- Estrazione di **proprietà verificabili** (invarianti)

> "Our goal is to use the LLM for less and less over time. We want to use classic automated reasoning techniques to give you high quality results, not just whatever the latest model is going to tell you." — Al Harris

---

## Property-Based Testing dai requisiti

Novità lanciata con la GA di Kiro: i requisiti EARS vengono tradotti automaticamente in **property-based tests** (PBT).

**Come funziona:**
1. Kiro estrae le "correctness properties" dal documento dei requirements e dal design
2. Le proprietà diventano test che verificano gli invarianti del sistema (usando framework come Hypothesis/Python, fast-check/Node, spec/Clojure)
3. L'agente esegue i task solo se le proprietà passano

**Perché PBT invece di unit test tradizionali:**
- Un property test cerca il **contro-esempio** che falsifica l'invariante
- Se nessun contro-esempio viene trovato → alta confidenza che il sistema rispetti il requisito
- I PBT sono generativi: producono automaticamente casi di test che un umano non scriverebbe

> "If the properties of the code meet the initial requirements, we have a high degree of confidence that you have reliably shipped the software you expected to ship." — Al Harris

---

## Spec vs Steering: distinzione operativa

| | Spec | Steering |
|---|---|---|
| **Cosa descrive** | Una feature o area problematica specifica | Il contesto permanente del progetto |
| **Ciclo di vita** | Vivente — viene mutata quando la feature evolve | Stabile — cambia con l'architettura del progetto |
| **Esempi** | requirements.md, design.md, tasks.md | Come fare commit, come deployare, convenzioni di codice, come funziona un servizio esterno |
| **Analogo** | Specifica di feature | CLAUDE.md, cursor rules, memory bank |
| **Quando crearla** | Per ogni nuova feature significativa | Quando si scopre qualcosa di non ovvio sul progetto |

**Pattern "Kira, scrivi quello che hai imparato in uno steering doc":** dopo aver risolto un problema non ovvio (es. parametro specifico richiesto da un tool), chiedere a Kiro di documentarlo nello steering per preservare la conoscenza.

---

## MCP nei tre stadi del workflow

L'integrazione MCP è disponibile in ogni fase — non solo durante l'esecuzione:

| Fase | Uso degli MCP |
|---|---|
| **Requirements** | Importare task da Asana/Jira; leggere requisiti esistenti dal sistema di project management |
| **Design** | Ricerca su documentazione tecnica (AWS docs MCP, fetch MCP); proporre alternative |
| **Implementation** | Accesso a servizi runtime, database, API esterne |

**Consiglio pratico:** cambiare la configurazione MCP mid-session è una cache-busting operation — evitare se si è profondi in una sessione lunga.

---

## Gestione del contesto e sessioni

**Isolamento task:** ogni task della task list viene eseguito in una sessione separata, inizializzata con la spec + steering. Nessun contesto condiviso tra task. Pro: parallelizzabile (sub-agents, in roadmap). Contro: richiede spec complete.

**Prompt caching:** il team Kiro ottimizza per il **cache hit rate** (90-95%) piuttosto che per la compressione del contesto. Questo mantiene le interazioni veloci nonostante sessioni lunghe (200k token limit).

**Incremental disclosure:** l'agente viene avviato con il minimo contesto necessario e usa i tool (codebase search, MCP) per auto-scoprire il contesto del task — invece di ricevere l'intero codebase all'inizio.

> "The agent does better when given less context but given the tools to understand where to go find things." — Al Harris

---

## Backend neurosimbolico

Kiro non è puramente LLM-driven: usa **neurosymbolic reasoning** per operazioni che richiedono determinismo.

- **LLM**: generazione di requisiti, design, codice, chat libera
- **Automated reasoning classico**: validazione dei requisiti (ambiguità, conflitti), property extraction, requirements verification
- **Obiettivo a lungo termine**: ridurre progressivamente la dipendenza dall'LLM per le operazioni che possono essere rese deterministiche

---

## Spec come living documentation

Le spec non vengono accumulate indefinitamente ma vengono **mutate** quando le feature evolvono. Questo è il pattern che il team Kiro usa internamente per le proprie design review:

1. Developer genera una spec per la nuova feature
2. La spec viene postata sulla wiki interna (via MCP)
3. Il team la revisiona e commenta — **spec review** invece di design session
4. Quando la feature evolve, la spec esistente viene aggiornata con un diff

> "On the next time I open that doc, I want to be seeing: you've relaxed this previous requirement, you've added a requirement that actually has this implication on the design doc." — Al Harris

---

## Efficacia su codebase esistenti (brownfield)

L'agente lavora meglio su codebase con:
- Alta **separazione delle responsabilità**
- Moduli **coesi** — l'agente può capire cosa fa un modulo senza tenere 18 file in contesto
- **Test suite affidabili** — la spec può fare assunzioni sul comportamento verificabile

Codebase con elevato debito tecnico o entanglement rendono il lavoro dell'agente molto più difficile — la stessa difficoltà che avrebbe un developer che entra nel codice per la prima volta.

---

## Citazioni notevoli

> "Vibe coding is great, but vibe coding relies a lot on me as the operator getting things right. That is me giving guardrails to the system."  
> — Al Harris (motivazione per SDD)

> "If I spend an hour generating a design doc, reviewing it with my team, and then synthesizing from that — I want to get it right."  
> — Al Harris (su perché SDD vale l'investimento)

> "The structure is important because the structure lets us build reproducible tooling that is not just an LLM."  
> — Al Harris (sul backend neurosimbolico)

> "Anything you put in the prompt is effectively rounding the agent, for better or for worse."  
> — Al Harris (sul rischio di bias impliciti nelle specifiche)

---

## Riferimenti e risorse

- **Kiro IDE** ([kiro.dev](https://kiro.dev)) — IDE agentioco di Amazon con SDD nativo; fork di VS Code; disponibile GA da gennaio 2026
- **EARS format** (Easy Approach to Requirements Syntax) — sintassi strutturata per requisiti leggibili da umani e processabili da automated reasoning
- **Property-Based Testing** — Hypothesis (Python), fast-check (Node.js), clojure.spec — framework per test basati su invarianti invece di casi specifici
- **AWS Documentation MCP** — MCP server ufficiale AWS per grounding dell'agente sulla documentazione AWS
- **Tessl** — citato come alternativa per specs orientate a knowledge base ("as long as you've got the right grounding docs")
- **Kiro blog** ([kiro.dev/blog](https://kiro.dev/blog)) — benchmark su come property-based testing migliora la task accuracy
