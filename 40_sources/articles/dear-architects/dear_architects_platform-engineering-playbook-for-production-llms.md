---
tags:
  - platform-engineering
  - llm
  - production-ai
  - hallucination-prevention
  - prompt-registry
  - observability
feature:
type: article
author: Aditya Mulik
source: https://www.infoq.com/articles/platform-engineering-playbook-production-llms/
date: 2026-10-11
---

# The Platform Engineering Playbook for Production LLMs

## Sunto

L'articolo descrive come un team di platform engineering ha ridotto il tasso di risposte allucinatorie da un agente LLM in produzione dal 15% all'1,5% in sei mesi. Il contesto è un sistema di raccomandazione inventario per un grande retailer. La lezione principale è che trattare lo stack LLM come infrastruttura platform condivisa — anziché come codice per-applicazione — è la svolta che rende sostenibile la produzione.

L'architettura si articola attorno a un **gateway** come singolo ingress, un **root coordinator** basato sul Google Agent Development Kit (ADK) che delega a agenti specializzati, e **server MCP per-team** che espongono gli strumenti. LiteLLM rende il provider del modello intercambiabile, mentre un modello Ollama locale supporta lo sviluppo a costo zero. Gli **output strutturati con Pydantic** vengono applicati al boundary della piattaforma, con campi `grounded` e `out_of_scope` che fungono da segnali di allucinazione. Il **prompt registry** archivia i prompt con versioning, referenziati per ID e risolti a runtime, permettendo rollback senza redeploy.

Il meccanismo di **retry orchestration** classifica i fallimenti in tre categorie — schema, allucinazione, infrastruttura — ognuna con una risposta diversa. Un **gate di intent classification** basato su regole YAML filtra le richieste prima della costosa chiamata al modello, restituendo "unclassified" invece di indovinare. L'**autenticazione e autorizzazione** usa JWT stateless con role claims; ogni server MCP applica una policy default-deny direttamente, non solo al gateway.

Per l'**osservabilità**, l'autore argomenta che l'APM tradizionale non può rilevare la degradazione semantica: occorre tracciare tassi di allucinazione, deriva dei prompt e costi per-team fin dalla prima richiesta. Il sistema di **evaluation** usa un golden set di casi eseguiti attraverso l'evaluator di ADK, con una soglia di superamento (tool-trajectory average score ≥ 0.8) applicata in CI. Il punto decisionale per adottare una piattaforma condivisa è la seconda applicazione LLM, non la decima.

## Codice

Schema Pydantic per output strutturato con segnali di allucinazione:

```python
from pydantic import BaseModel, Field

class DocsOutput(BaseModel):
    """Result of a grounded documentation lookup."""

    answer: str = Field(
        description="The answer, grounded entirely in the fetched documentation."
    )
    sources: list[DocSource] = Field(
        default_factory=list,
        description="Documentation sources cited; empty only when nothing was fetched.",
    )
    grounded: bool = Field(
        description="True if every claim is backed by the fetched docs; "
        "False if the docs did not cover the question.",
    )
    out_of_scope: bool = Field(
        default=False,
        description="True if the request fell outside documentation lookup.",
    )
```

Loop di retry con classificazione degli errori nell'agente custom ADK:

```python
from google.adk.agents import LlmAgent

class CustomLlmAgent(LlmAgent):

    def __init__(self, *args, max_validation_retries: int = 3, **kwargs):
        super().__init__(*args, **kwargs)
        self._max_validation_retries = max_validation_retries

    async def _run_async_impl(self, ctx):
        for attempt in range(self.max_validation_retries):
           try:
               async for event in super()._run_async_impl(ctx):
                   yield event
               return
           except ValidationError as exc:
               is_last_attempt = attempt == self.max_validation_retries - 1
               logger.warning(
                   "[%s] VALIDATION ERROR (attempt %d/%d): %s%s",
                   self.name, attempt + 1, self.max_validation_retries,
                   exc.errors(include_url=False),
                   " -- retrying..." if not is_last_attempt else " -- max retries exceeded",
               )
               metrics.record_hallucination(get_current_user())
               if is_last_attempt:
                   raise
```

Gate di classificazione dell'intent basato su regole YAML — restituisce "unclassified" invece di fare default a un agente fisso:

```python
def classify_intent(user_input: str) -> Dict:
    user_input_lower = user_input.lower()
    agent_scores = {}
    for agent_name, agent_rules in INTENT_RULES.get("agents", {}).items():
        keywords = agent_rules.get("keywords", [])
        weight = agent_rules.get("weight", 1.0)
        score = sum(1 for kw in keywords if kw in user_input_lower) * weight
        agent_scores[agent_name] = score

    if not agent_scores or all(score == 0 for score in agent_scores.values()):
        return {
            "status": "unclassified",
            "agent": None,
            "reason": "No intent rule matched the request.",
            "available_agents": list(INTENT_RULES.get("agents", {}).keys()),
            "next_action": "ask_user_about_web_search",
        }

    best_agent = max(agent_scores, key=agent_scores.get)
    return {"status": "classified", "agent": best_agent, "score": agent_scores[best_agent]}
```

Middleware di autorizzazione MCP con policy default-deny per singolo server:

```python
from fastmcp.server.middleware import Middleware, MiddlewareContext

class AuthzMiddleware(Middleware):
    async def on_call_tool(self, context: MiddlewareContext, call_next):
        tool_name = context.message.name
        auth_header = get_http_headers(include={"authorization"}).get("authorization")
        try:
            claims = decode_token(token_from_header(auth_header))
        except AuthError as exc:
            raise ToolError(f"Unauthorized: {exc}")

        role = claims.get("role", "")
        if not policy_engine.can_access_tool(role, tool_name):
            raise ToolError(
                f"Forbidden: role '{role}' is not authorized to use tool '{tool_name}'"
            )
        return await call_next(context)
```

Configurazione della soglia di superamento per la valutazione in CI:

```json
{ "criteria": { "tool_trajectory_avg_score": 0.8 } }
```
