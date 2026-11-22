---
tags:
  - ai-agents
  - durable-execution
  - temporal
  - long-horizon-agents
  - workflow-orchestration
feature:
type: article
author: Kiran Prakash
source: https://martinfowler.com/articles/durable-execution-agents.html
date: 2026-11-22
---

# Long-horizon agents and durable execution

## Sunto

Kiran Prakash, Director of Data and AI presso Thoughtworks, affronta un problema fondamentale degli agenti AI moderni: come gestire task che richiedono giorni o settimane, coinvolgono sistemi esterni e persone reali, e devono sopravvivere a crash, errori di rete e comportamenti imprevisti del mondo reale. L'articolo usa come esempio ricorrente "Stock Replenisher", un agente che rifornisce i componenti di un produttore: ottiene preventivi dai fornitori, richiede approvazione al manager, emette ordini di acquisto e traccia le consegne.

Il cuore della soluzione è la **durable execution**: ogni chiamata a sistemi esterni (chiamate LLM, ERP, messaggi ai fornitori) viene eseguita come uno "step" atomico il cui risultato viene registrato in un write-ahead log. In caso di crash, il programma riparte, salta gli step già registrati e ricostruisce lo stato. Il codice tra gli step deve essere deterministico, poiché viene rieseguito durante il replay. Questo pattern è implementato con Temporal (framework di workflow orchestration), che fornisce primitive come `workflow.execute_activity` con `activity_id` per garantire idempotenza.

Un'architettura chiave è il modello entity/process workflow: un workflow "entità" di lunga durata esiste per ogni SKU e instrada i messaggi al processo di rifornimento attivo. I workflow sospesi non consumano CPU né memoria — la misurazione mostra riattivazione in millisecondi per workflow in-memory e circa due secondi per quelli ricostruiti dal log su Cloudflare. I segnali (come l'approvazione del manager) vengono inviati direttamente all'ID del workflow dell'SKU, che li inoltravail processo attivo.

La semantica at-least-once delle retry può generare scritture duplicate. La soluzione è un **idempotency key** (il numero dell'ordine di acquisto) costante attraverso i retry. Per le azioni già eseguite scoperte tardivamente, si usa un `cancel_order` tool — analogo alle compensating transaction del pattern Saga. Le decisioni "stantie" (prese prima di una lunga attesa) si gestiscono con date di scadenza, ri-verifica dei fatti prima di azioni irreversibili (pattern optimistic offline lock), e approvando range di prezzi invece di valori singoli.

La durabilità non equivale a correttezza: il codice tra gli step deve essere deterministico e le regole di business devono essere versionabili per permettere ai workflow in-flight di completarsi sotto le regole con cui sono partiti. Temporal e Cloudflare si comportano diversamente quando una decisione registrata viola una regola aggiornata. Per la sicurezza, `allowed_tools` limita le azioni del modello a livello di codice, non di prompt, e le approvazioni sono accettate solo da fonti autorizzate. La protezione da prompt injection rimane aperta.

## Immagini

![Diagramma dell'agentic loop - due turni del loop](https://martinfowler.com/articles/durable-execution-agents/agentic-loop.svg)

![Come funziona - ogni chiamata esterna è uno step](https://martinfowler.com/articles/durable-execution-agents/how-it-works-1.svg)

![Write-ahead log - dettaglio di uno step](https://martinfowler.com/articles/durable-execution-agents/how-it-works-2.svg)

![Replay dopo un crash - gli step registrati non vengono rieseguiti](https://martinfowler.com/articles/durable-execution-agents/replay.svg)

![Agente e workflow - un agente per SKU, un workflow alla volta](https://martinfowler.com/articles/durable-execution-agents/agent-and-workflows.svg)

![Wait - il timer e il messaggio vengono entrambi registrati nel log](https://martinfowler.com/articles/durable-execution-agents/wait.svg)

## Codice

Il loop agentivo di base: selezione degli tool consentiti, esecuzione o attesa, applicazione dei risultati allo stato.

```python
class Replenish:
    def run(self, state: State) -> None:
        while not state.done():
            allowed = [t for t in ALL_TOOLS if t.allowed(state)]
            tool_call = llm.select_tool(state, allowed)
            if tool_call.is_wait():
                state.wait(tool_call.duration)
            else:
                result = tool_call.execute(state)
                state.apply(result)
```

Il tool `PlaceOrder` con idempotency key e verifica del prezzo approvato:

```python
class PlaceOrder(Tool):
    def allowed(self, state: State) -> bool:
        return (state.approved_price_ceiling is not None
                and state.best_quote() <= state.approved_price_ceiling)

    def execute(self, state: State) -> Result:
        # Il PO number è costante attraverso i retry
        return erp.place_order(
            sku=state.sku,
            supplier=state.best_supplier(),
            price=state.best_quote(),
            idempotency_key=state.po_number  # garantisce esattamente una scrittura
        )
```

Loop durevole con Temporal — ogni chiamata esterna come activity con `activity_id`:

```python
@workflow.defn
class ReplenishWorkflow:
    @workflow.run
    async def run(self, state: State) -> None:
        while not state.done():
            allowed = [t for t in ALL_TOOLS if t.allowed(state)]
            tool_call = await workflow.execute_activity(
                select_tool,
                args=[state, allowed],
                activity_id=f"select-{state.step_count}"
            )
            if tool_call.is_wait():
                await workflow.wait_condition(
                    lambda: state.has_signal(),
                    timeout=tool_call.duration
                )
            else:
                result = await workflow.execute_activity(
                    tool_call.name,
                    args=[state],
                    activity_id=f"{tool_call.name}-{state.step_count}"
                )
                state.apply(result)
```

Regola di business con versioning per workflow in-flight:

```python
class DraftPurchaseOrder(Tool):
    def allowed(self, state: State) -> bool:
        if state.rules_version >= 2:
            # Nuova regola: richiede 4 preventivi
            return len(state.quotes) >= 4
        else:
            # Regola originale: basta 1 preventivo
            return len(state.quotes) >= 1
```
