# Spec-Driven Development – Adoption at Enterprise Scale

**Fonte:** [InfoQ](https://www.infoq.com/articles/enterprise-spec-driven-development/)  
**Data lettura:** 2026-04-11  
**Tipo:** Article  
**Tag:** #sdd #spec-driven-development #ai-coding #enterprise #architecture #context-engineering #agentic-ai

---

## Key Takeaways

- Con l'esecuzione agente ininterrotta che sostituisce sempre più il prompting interattivo, l'articolazione dell'intent diventa ancora più critica per usare efficacemente i coding agents.
- Lo Spec-Driven Development (SDD) aiuta a ingegnerizzare il contesto efficacemente, ma a scala enterprise gli strumenti attuali presentano gap significativi.
- Nel breve periodo, l'adozione richiede integrazione con i workflow esistenti, supporto per progetti brownfield, e capacità di abilitare progressivamente tecniche sofisticate.
- Nel lungo periodo, man mano che ci spostiamo in ruoli review-centrici, dobbiamo sviluppare comprensione intuitiva per usare gli strumenti SDD efficacemente, gestendo il contesto per evitare di essere sopraffatti da feedback loop addizionali.
- Quando implementato efficacemente, SDD migliora la collaborazione tra stakeholder. Adottarlo esclusivamente come processo tecnico perde questo beneficio principale.

---

## Evoluzione dell'Intent Articulation: da Istruzioni a Dialogo

### 1. Vibe Coding — Interazione Istruzionale

Il "vibe coding" (iterare con l'AI finché il codice funziona, senza pianificazione upfront deliberata) ha mantenuto interazioni largamente istruzionali con l'AI, un prompt alla volta. L'implementazione stessa serviva da contesto per ulteriori aggiustamenti.

![Figure 1: Vibe coding workflow](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/enterprise-spec-driven-development/en/resources/1Figure-1-1771319819572.jpg)
*Figure 1: Vibe coding workflow*

### 2. Plan Mode — Preparazione Migliore

Il Plan Mode (dove l'AI bozza un piano di esecuzione per review umana prima di scrivere codice) ha segnato una grande evoluzione. Richiedendo a umani e AI di deliberare su una lista di task prima dell'implementazione, ha esteso i tempi di esecuzione indipendente. È stata la prima "kick-off ceremony" con l'AI.

Tuttavia, i piani tipicamente non persistono oltre l'esecuzione, quindi l'implementazione rimane il contesto primario per raffinamenti successivi.

![Figure 2: Plan Mode Workflow](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/enterprise-spec-driven-development/en/resources/1Figure-2-1771319819572.jpg)
*Figure 2: Plan Mode Workflow*

### 3. Spec-Driven Development — Dialogo Umano-AI

SDD è emerso quando i modelli AI hanno iniziato a dimostrare focus prolungato su task complessi. Le specifiche facilitano il **dialogo** tra umani e AI, piuttosto che fungere da manuali di istruzioni.

![Figure 3: Spec-Driven Development high-level workflow](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/enterprise-spec-driven-development/en/resources/1Figure-3-1771319819572.jpg)
*Figure 3: Spec-Driven Development high-level workflow*

---

## Adozione Enterprise: Perché è Importante (e non è un Rollout Tecnico)

### Conversazioni invece di Istruzioni

Quando senior engineer collaborano, la comunicazione è conversazionale, non unidirezionale. SDD facilita questo stesso pattern tra umani e agenti AI, dove gli agenti aiutano a pensare le soluzioni, sfidare assunzioni, e raffinare l'intent prima dell'esecuzione.

![Figure 4: Spec-Driven Development detailed workflow](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/enterprise-spec-driven-development/en/resources/1Figure-4-1771319819572.jpg)
*Figure 4: Spec-Driven Development detailed workflow*

### Breakdown delle Specifiche (3 livelli)

Con SDD, i team possono decomporre le feature in aspetti che guidano l'esecuzione autonoma:

| Fase | Domanda | Chi |
|------|---------|-----|
| **Discover** (What) | Contesto di business che definisce il use case | Product |
| **Design** (How) | Approccio tecnico che mappa il use case all'architettura | Architecture |
| **Tasks** | Piani di esecuzione su cui gli agenti agiscono | Engineering |

### Collaborative Context > Modelli più Intelligenti

I team che costruiscono contesto di esecuzione attraverso spec cross-funzionali sono superiori ai singoli che ottimizzano prompt o inseguono modelli più smart. Le spec fungono da **translation layer** che cattura il dialogo evolutivo degli stakeholder.

### Il Rischio del "SpecFall" / "Markdown-Monster"

Adottare workflow spec-driven senza cambiare come product, architecture, engineering e QA lavorano effettivamente insieme rischia di creare un "markdown monster": strati di documentazione obsoleta generata all'arrivo. Il rischio è lo stesso visto nell'enterprise agile adoption con lo "Scrumerfall".

> **Punto chiave**: Le spec devono essere sia il medium di conversazione tra stakeholder che il context engine per gli agenti AI.

---

## Gap nell'Adozione SDD a Scala

### 1. Tooling Developer-Centrico
La maggior parte dei tool SDD vive in Git repos e CLI, creando barriere per la collaborazione cross-funzionale. Product manager e business analyst — che dovrebbero definire il "What" — faticano a partecipare.

### 2. Focus Mono-Repo
I tool attuali co-locano spec e codice in un singolo repository. Ma le architetture enterprise spannano microservizi, librerie condivise, infra repos e componenti di piattaforma.

### 3. Mancanza di Separation of Concerns
Le decisioni architetturali (schema design) e il contesto di business (acceptance criteria) hanno audience e ritmi evolutivi diversi rispetto alla task list tecnica. Ma la maggior parte dei tool organizza tutto in folder per feature.

### 4. Starting Point poco chiari
La maggior parte delle enterprise ha backlog qualificati in Jira o Azure DevOps. Ma i tool SDD non si integrano con questi sistemi, rendendo difficile non duplicare il lavoro.

### 5. Collaboration Pattern non definiti
Chi contribuisce in quale fase? Quando inizia e finisce il contributo di ogni stakeholder? Come avvengono le approval?

### 6. Range ampio di Spec Style e Granularità

I tool principali usano approcci molto diversi:

| Tool | Approccio |
|------|-----------|
| GitHub SpecKit | Structured user stories + acceptance criteria |
| [Kiro](https://kiro.dev/docs/specs/concepts/#requirements) | Pattern EARS, organizzazione a livello feature |
| Tessl | Test embedded nelle spec |
| OpenSpec | Approccio incrementale, fasi proposal/application/archival |

### 7. Alignment Spec-to-Implementation
Il processo ha due transizioni distinte:
- **Intent-to-specification**: avviene tramite dialogo e review
- **Specification-to-implementation**: validazione più difficile, dipende dallo stile della spec

### 8. Brownfield Adoption Path poco chiaro
Creare spec retroattive per sistemi legacy è impraticabile su larga scala. Eccede i limiti di contesto e crea spec enormi difficili da revieware.

---

## Abilitare l'Adozione SDD Senza "Bollire l'Oceano"

### Integrazione con Backlog Esistenti via MCP

Gli MCP server forniscono un layer di integrazione pratico. I developer possono fare pull di storie da Jira, Linear o Azure DevOps direttamente nel loro workflow SDD, con aggiornamenti di stato che fluiscono automaticamente.

**Esempio con [OpenSpec](https://github.com/Fission-AI/OpenSpec)**:

Il workflow di OpenSpec prevede tre fasi: **proposal → application → archival**.

![Figure 5-a: OpenSpec Spec-Driven Development workflow](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/enterprise-spec-driven-development/en/resources/1Figure-5-a-1771319819572.jpg)
*Figure 5-a: [OpenSpec Spec-Driven Development workflow](https://blog.harikrishnan.io/2025-11-09/spec-driven-development-openspec-source-truth)*

Nel workflow modificato, integrato con il backlog via MCP:
- **Proposal** → Issue in "Todo"
- **Application** → Issue in "In Progress"
- **Archival** → Issue in "Done"

![Figure 5-b: OpenSpec modified SDD workflow integrated with product backlog through MCP](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/enterprise-spec-driven-development/en/resources/1Figure-5-b-1771319819572.jpg)
*Figure 5-b: OpenSpec modified SDD workflow integrato con product backlog tramite MCP*

### Orchestrazione Multi-Repository

Per feature che toccano più repo, la separazione tra contesto di business e dettagli di implementazione tecnica è cruciale.

![Figure 6: Product owner, architect, and dev collaboration facilitated through SDD workflow](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/enterprise-spec-driven-development/en/resources/1Figure-6-1771319819572.jpg)
*Figure 6: Collaborazione tra Product Owner, Architect e Dev tramite SDD workflow*

**Discover Phase**: Il product owner lavora con AI per articolare l'intent di business ("what" e "why").

**Design Phase**: L'architetto determina l'approccio tecnico e decompone la storia parent in sub-issue per repository. Le sub-issue rimangono nel backlog per la visibilità cross-team.

**Task Phase**: Il developer lavora su ogni sub-issue nel suo repository, dettagliando i task di implementazione.

Gli architetti possono documentare i confini dei repository, i pattern di integrazione e i vincoli architetturali, permettendo agli agenti di generare automaticamente sub-issue appropriate per ogni repo coinvolto.

![Figure 7: Front end and back end sub issues being generated based on architecture context](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/enterprise-spec-driven-development/en/resources/1Figure-7-1771319819572.jpg)
*Figure 7: Claude Code che genera sub-issue frontend/backend basandosi sul contesto architetturale*

### Contributi Role-Specific

Infrastructure specialists, performance expert, security professional: ognuno stabilisce il proprio "context harness" con vincoli e pattern specifici del dominio. Gli agenti specializzati applicano automaticamente la propria expertise:
- Infrastructure agents → deployment constraints
- Performance agents → optimization needs
- Security agents → compliance requirements

### Spec Style e Validazioni: Criteri di Scelta

- **Top-Level Steering Ability**: supporto per documenti che influenzano le spec cross-feature (es. Constitution in SpecKit, Steering Docs in Kiro)
- **Top-Level Spec View**: capacità di estrarre una visione dello stato attuale dell'applicazione
- **Granularità ragionevole**: spec "human reviewable" in termini di dimensione; evitare artifact quasi identici al codice reale

### Adozione Brownfield

Approccio incrementale: partire dall'area di cambiamento attuale, non dall'intero sistema. La copertura delle spec cresce organicamente con ogni modifica. Bug fix, feature addition, refactoring diventano opportunità per aggiungere spec al codice toccato.

> Analogo al TDD durante il refactoring legacy: coprire la logica esistente attorno all'area di cambiamento, poi isolare quell'area efficacemente.

---

## Direzione di Lungo Periodo

### Spec come Prima Interfaccia (non Documentazione Secondaria)

In SDD maturo, **ogni cambiamento** (anche bug fix minori) deve fluire attraverso le spec. Le spec sono l'interfaccia primaria per dirigere l'esecuzione degli agenti.

Analogia storica: abbiamo rimosso l'accesso diretto ai server perché le modifiche dirette venivano sovrascritte al prossimo deploy. Allo stesso modo, le modifiche dirette al codice AI-generato ampliano il gap con la spec, e quel gap continua a riemergere ogni volta che il codice viene rigenerato.

### Harness Governance

Come indicato nell'articolo InfoQ "[Spec Driven Development: When Architecture Becomes Executable](https://www.infoq.com/articles/spec-driven-development/)" di Leigh e Ray (concetto SpecOps), l'authoring delle spec diventa una **first-class engineering surface**.

Quando emergono bug, identificarne l'origine è essenziale:

| Tipo di Gap | Causa | Soluzione |
|-------------|-------|-----------|
| **Spec-to-Implementation Gap** | La spec era chiara, ma l'implementazione ha deviato | Rafforzare i meccanismi di validazione nel harness |
| **Intent-to-Spec Gap** | Il dettaglio del use case è stato perso durante la deliberazione | Migliorare il processo di elicitation delle spec |

Questi gap non sono solo classificazioni di bug: sono **metriche di qualità per il context harness**.

![Figure 8: Harness feedback loop](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/enterprise-spec-driven-development/en/resources/1Figure-8-1771319819572.jpg)
*Figure 8: Harness feedback loop — il ciclo di miglioramento continuo*

Il ruolo del Quality Engineering evolve: da validare implementazioni a **validare e migliorare gli harness** che guidano l'esecuzione degli agenti.

---

## Conclusione

Come ha inquadrato [Adrian Cockcroft a QCon SF](https://qconsf.com/presentation/nov2025/directing-swarm-agents-fun-and-profit), stiamo imparando a dirigere sciami di agenti. Il bottleneck non è più la velocità di scrittura del codice — è l'efficacia nell'articolare l'intent.

| Approccio | Benefici ottenuti |
|-----------|------------------|
| SDD come rollout tecnico | Migliore token usage, run agente più lunghe, meno hallucination |
| SDD come trasformazione organizzativa | Capacità di dirigere agent swarm, creatività umana liberata per problem-solving strategico |

La trasformazione non è uno stato futuro: **è disponibile ora**, per le organizzazioni pronte a fare il salto.

---

## Immagini dell'articolo

| Figure | Descrizione | URL |
|--------|-------------|-----|
| Fig. 1 | Vibe coding workflow | [link](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/enterprise-spec-driven-development/en/resources/1Figure-1-1771319819572.jpg) |
| Fig. 2 | Plan Mode Workflow | [link](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/enterprise-spec-driven-development/en/resources/1Figure-2-1771319819572.jpg) |
| Fig. 3 | SDD high-level workflow | [link](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/enterprise-spec-driven-development/en/resources/1Figure-3-1771319819572.jpg) |
| Fig. 4 | SDD detailed workflow | [link](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/enterprise-spec-driven-development/en/resources/1Figure-4-1771319819572.jpg) |
| Fig. 5-a | OpenSpec SDD workflow | [link](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/enterprise-spec-driven-development/en/resources/1Figure-5-a-1771319819572.jpg) |
| Fig. 5-b | OpenSpec + MCP backlog integration | [link](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/enterprise-spec-driven-development/en/resources/1Figure-5-b-1771319819572.jpg) |
| Fig. 6 | PO + Architect + Dev collaboration | [link](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/enterprise-spec-driven-development/en/resources/1Figure-6-1771319819572.jpg) |
| Fig. 7 | Sub-issues generati da contesto architetturale | [link](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/enterprise-spec-driven-development/en/resources/1Figure-7-1771319819572.jpg) |
| Fig. 8 | Harness feedback loop | [link](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/enterprise-spec-driven-development/en/resources/1Figure-8-1771319819572.jpg) |

---

## Link citati nell'articolo

- [EARS (Easy Approach to Requirements Syntax) — Kiro docs](https://kiro.dev/docs/specs/concepts/#requirements)
- [OpenSpec — GitHub](https://github.com/Fission-AI/OpenSpec)
- [OpenSpec SDD workflow — blog post](https://blog.harikrishnan.io/2025-11-09/spec-driven-development-openspec-source-truth)
- [Spec Driven Development: When Architecture Becomes Executable — InfoQ](https://www.infoq.com/articles/spec-driven-development/)
- [Directing Swarm Agents for Fun and Profit — Adrian Cockcroft @ QCon SF](https://qconsf.com/presentation/nov2025/directing-swarm-agents-fun-and-profit)

---

## Analisi e Riflessioni

### Punti di forza dell'articolo:
- Ottima progressione narrativa: Vibe Coding → Plan Mode → SDD
- Molto concreto sulle **difficoltà enterprise** reali (brownfield, multi-repo, stakeholder diversi)
- La distinzione tra rollout tecnico e trasformazione organizzativa è fondamentale e spesso ignorata
- Il concetto di "Harness Governance" è maturo e sistemico

### Concetti chiave da ricordare:
- **"SpecFall"**: il rischio di fare SDD come si faceva Scrumerfall con Agile
- **"Markdown Monster"**: spec generate ma mai mantenute né usate
- **Context Harness**: il layer di vincoli e pattern domain-specific che guidano gli agenti
- **Feedback loop harness**: ogni bug è un'opportunità per migliorare il harness, non solo patchare il codice

### Connessioni con altri concetti:
- Si collega direttamente a **CALM** (articolo precedente): entrambi puntano a rendere la complessità gestibile tramite artefatti strutturati
- Connessione con **Platform Engineering**: il harness è di fatto una Internal Developer Platform per agenti AI
- Relazione con **TDD/BDD**: l'approccio brownfield incrementale è esplicitamente analogo al refactoring test-driven
- L'evoluzione del ruolo QA verso "harness quality" è un cambiamento radicale che merita approfondimento
- Il tema **MCP come integration layer** emerge come pattern architetturale ricorrente nell'ecosistema AI
