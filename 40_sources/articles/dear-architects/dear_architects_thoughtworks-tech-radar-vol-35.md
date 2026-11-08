---
tags:
  - technology-radar
  - ai-agents
  - harness-engineering
  - software-architecture
  - platform-engineering
feature:
type: article
author: Thoughtworks
source: https://www.thoughtworks.com/radar
date: 2026-11-08
---

# Technology Radar Vol. 35 — Thoughtworks (Novembre 2026)

## Sunto

Il Technology Radar di Thoughtworks, giunto al volume 35, è uno snapshot semestrale degli strumenti, tecniche, piattaforme, linguaggi e framework rilevanti nell'ecosistema software. Questo numero è dominato da quattro temi principali legati all'intelligenza artificiale e allo sviluppo agentivo, con una particolare attenzione alla maturazione dell'harness engineering come disciplina autonoma.

Il primo tema è **harness engineering su larga scala**: l'ambiente costruito attorno agli agenti AI — memoria, loop di feedback, guardrail, contesto — viene riconosciuto come un asset vivente che richiede manutenzione continua. Sta emergendo un ecosistema di plugin per agenti, e la gestione dell'harness diventa una responsabilità ingegneristica di primo livello.

Il secondo tema riguarda il superamento dei **modelli frontier**: i modelli open-weight e locali stanno chiudendo il divario con i modelli proprietari per molti task. Thoughtworks incoraggia un approccio di routing deliberato: non usare sempre il modello più costoso, ma selezionare il modello più appropriato per ogni caso d'uso.

Il terzo tema è la **parallelizzazione degli agenti**: i team stanno eseguendo sessioni agentive parallele e build notturne non presidiate. La capacità di review e le pipeline di delivery stanno diventando i colli di bottiglia.

Il quarto tema riguarda la costruzione di una **AI factory**: Kubernetes emerge come base per piattaforme AI industrializzate. I costi in token dipendono dalle scelte infrastrutturali: routing dei modelli, batching, movimento dei dati e storage.

## Principali blip per anello

**Adopt (Adotta):**
- *Tecniche*: agent sandboxing, mutation testing, path-to-production mapping, post-quantum cryptography, progressive context disclosure, threat modeling
- *Piattaforme*: VictoriaMetrics
- *Strumenti*: mise, OpenAI Codex, OpenCode
- *Linguaggi/Framework*: Open Data Contract Standard (ODCS)

**Trial (Sperimenta):**
- *Tecniche*: agent plugin marketplace, agent-maintained knowledge bases, feedback flywheel, managed agents runtime, open-weight models, risk-tiered human oversight, small language models, stateful guardrails, subagents for context management, unattended overnight agentic builds
- *Piattaforme*: KServe, LiteLLM, Temporal, Restate, Talos Linux, vCluster, VictoriaLogs
- *Strumenti*: Linear, Git AI, Beads, ast-grep, Chaos Mesh, Pi, Sourcebot
- *Linguaggi/Framework*: Deep Agents, Kagent, OpenFGA, Presidio, Promptfoo, BoTorch

**Assess (Valuta):**
- *Tecniche*: enterprise harness, local coding models, MITRE ATLAS, programmatic tool calling, tests as data over tests as code
- *Piattaforme*: Agent Client Protocol (ACP), Claude Managed Agents, llm-d, agentgateway, AWS DevOps Agent
- *Strumenti*: GLM, Herdr, Entire CLI, Hunk, AgentsView, Vale, zizmor, Varlock
- *Linguaggi/Framework*: TLA+, Quint, SolidJS, CALM, Hindsight

**Caution (Attenzione):**
- blanket frontier model usage (uso indiscriminato dei modelli frontier)
- human-in-the-loop theater (presenza umana nel loop solo per apparenza)
- coding throughput as a measure of productivity
- agent instruction bloat (gonfiore delle istruzioni degli agenti)
- naive context compression
- stacked pull requests
- AI-accelerated shadow IT
- AI-driven documentation sprawl
- meat proxy (umano come semplice relay tra due sistemi AI)
