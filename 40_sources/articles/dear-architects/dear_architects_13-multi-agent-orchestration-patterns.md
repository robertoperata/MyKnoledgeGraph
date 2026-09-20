---
tags:
  - multi-agent
  - orchestrazione
  - reactive-patterns
  - ai-agents
  - deterministico
feature:
type: article
author: Brad Murry
source: https://bradmurry.com/software/reactive-reducer-patterns/
date: 2026-09-20
---

# The 13 Multi-Agent Orchestration Patterns

## Sunto

Brad Murry analizza tredici pattern ricorrenti di orchestrazione multi-agente esaminando come ciascuno funziona quando implementato tramite una architettura **"reactive reducer"** piuttosto che il tradizionale message passing tra agenti. Il reactive reducer rimuove i modelli dal control flow attraverso tre principi: il contesto condiviso come unica fonte di verità (tutti le decisioni e gli artefatti esistono come campi tipizzati in uno store), gli agenti contribuiscono valori ma non richiedono transizioni (gli attori modificano il contesto, la macchina avanza da sola), e micro-step loop deterministici (la macchina si ferma in stati dichiarati, journaled e restart-safe).

I tredici pattern sono organizzati in sei categorie. I **Flow Patterns** (Sequential Pipeline, Parallel Fan-Out/Fan-In) gestiscono la progressione delle fasi con back-edge per revisione e decomposizione concorrente con aggregazione. Gli **Authority Patterns** (Supervisor/Router, Hierarchical Manager/Worker) implementano hub centralizzati di routing verso specialisti e delega multi-livello con responsabilità scoped. I **Delegation Patterns** (Peer Handoff/Swarm, Auction/Contract-Net, Dynamic Team Formation) gestiscono trasferimento dinamico del controllo, bidding competitivo per assignment di task, e assembramento adattivo del team basato sui requisiti della missione.

I **Reasoning Patterns** (Debate/Consensus, Quorum/Voting, Generator/Critic/Refiner) sono tra i più sofisticati: il primo usa valutazione multi-prospettiva con sintesi, il secondo valutazione indipendente con regole di aggregazione, il terzo cicli iterativi di miglioramento della qualità. Il **Planning Pattern** (Planner/Executor) separa decomposizione strategica ed esecuzione tattica. I **Coordination Patterns** (Blackboard/Shared-State, Event-Driven/Publish-Subscribe) gestiscono contribuzione asincrona a workspace comuni e reazione a cambiamenti di stato ed emissioni.

Tre idiomi tecnici chiave guidano l'implementazione corretta. **Map fields per contributor multipli**: N attori che contribuiscono contenuto simile usano un unico campo `map` tipizzato, con chiave per autore reale (imposta dal runtime), collassando logica di join, attribuzione e conteggio. **Consumo vs accumulo**: i trigger loop-re-arming vengono distrutti (`onTransition`) mentre le evidenze si accumulano in campi `list` o `map` via `exit` actions, prevenendo il riuso silenzioso di giudizi obsoleti. **Guard comparativi vs test di presenza**: poiché nessun dato accumulato viene distrutto, i test di freschezza diventano confronti di dimensione (es. "più draft che verdetti") piuttosto che check booleani di presenza.

Il vantaggio principale del reactive reducer rispetto all'autonomia dinamica degli agenti è la triade: esecuzione deterministica, costi bounded, e completa auditabilità. La stessa FSM (Finite State Machine) gira interattivamente o compilata su piattaforme di streaming, separando la logica di orchestrazione dal substrato di esecuzione. I sistemi reali tipicamente compongono pattern multipli simultaneamente.

## Codice

Struttura base di uno store condiviso per il reactive reducer:

```typescript
// Contesto condiviso come unica fonte di verità
interface OrchestratorContext {
  // Flow state
  currentPhase: 'planning' | 'execution' | 'review' | 'done';
  
  // Map field per contributi multipli (Debate pattern)
  perspectives: Map<AgentId, Perspective>;
  verdicts: Map<AgentId, Verdict>;
  
  // Accumulo di evidenze (non distrutto)
  drafts: Draft[];
  refinements: Refinement[];
  
  // Trigger consumabili (distrutti onTransition)
  pendingTasks: Task[];
}

// Guard comparativo per freschezza (non test di presenza)
const hasNewVerdicts = (ctx: OrchestratorContext) =>
  ctx.verdicts.size > ctx.perspectives.size - 1; // Più verdetti che prospettive precedenti
```

Pattern Generator/Critic/Refiner con reactive reducer:

```typescript
// Agenti contribuiscono valori, non richiedono transizioni
const generatorAgent = async (ctx: OrchestratorContext) => {
  const draft = await generateDraft(ctx.requirements);
  // L'agente modifica il contesto; la macchina avanza da sola
  return { drafts: [...ctx.drafts, draft] };
};

const criticAgent = async (ctx: OrchestratorContext) => {
  const latestDraft = ctx.drafts[ctx.drafts.length - 1];
  const critique = await critique(latestDraft);
  return { critiques: [...ctx.critiques, critique] };
};

// FSM avanza deterministicamente in base allo stato
const transitions = {
  generating: {
    guard: (ctx) => ctx.drafts.length > ctx.critiques.length,
    target: 'critiquing'
  },
  critiquing: {
    guard: (ctx) => ctx.critiques.length > ctx.refinements.length,
    target: ctx.critiques.last().score > THRESHOLD ? 'done' : 'refining'
  }
};
```
