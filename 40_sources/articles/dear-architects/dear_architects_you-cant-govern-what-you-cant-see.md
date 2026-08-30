---
tags:
  - architectural-governance
  - ai-agents
  - fitness-functions
  - observability
  - architettura-software
  - opentelemetry
feature:
type: article
author: Luca Mezzalira
source: https://www.linkedin.com/pulse/you-cant-govern-what-see-luca-mezzalira-quige/
date: 2026-08-30
---

# You Can't Govern What You Can't See

## Sunto

Luca Mezzalira affronta in questo articolo un problema emergente nell'era dell'AI agentica: quando gli agenti di coding accelerano la velocità di sviluppo, la deriva architetturale avviene più velocemente di quanto la revisione umana possa intercettarla. Il monitoraggio tradizionale copre la salute operativa (uptime, latenza) e la correttezza del codice (test, tipi, linter), ma lascia un punto cieco critico: "è ancora il sistema che abbiamo progettato?" Questo gap si allarga drammaticamente con il volume di codice generato dagli agenti.

L'articolo illustra il problema con un esempio concreto: dopo quattro mesi di migrazione assistita da agenti, tutte le metriche apparivano sane, eppure un developer scoprì credenziali email hardcoded sepolte in 400 righe di codice — una violazione del confine del servizio di notifica stabilito. Il punto è che non è solo "cosa" viene violato, ma "la velocità con cui arriva" che supera la percezione umana. Mezzalira identifica tre categorie di segnali, ognuna cieca in direzioni diverse: le Rules (catturano violazioni esplicite, ma hanno scope ristretto), la Structure (misura i grafi di dipendenza, ma è silenziosa sull'intenzione), e il Runtime (mostra il comportamento reale del sistema, ma manca le violazioni statiche). Nessun segnale singolo è sufficiente; le combinazioni rivelano ciò che le metriche individuali nascondono.

Il sistema che Mezzalira ha costruito combina cinque componenti: Deterministic Scoring (funzione pura con pesi pubblicati, su Cloud Run, indipendente dal ragionamento LLM), Architecture Tests (controlli CI-level contro i vincoli architetturali), Agent Reasoning (Gemini Pro 2.5 per contestualizzare i risultati e spiegarne le conseguenze), Memory Bank (traccia i pattern storici per l'analisi della traiettoria), e Runtime Tracing (cattura il comportamento reale del sistema via OpenTelemetry). La scoperta più preziosa è emersa dal confronto tra l'analisi a livello di sorgente e il comportamento a runtime: un servizio chiamava l'inventario via HTTP nonostante il requisito progettuale di messaggistica asincrona — invisibile agli strumenti statici, ma immediatamente evidente nelle tracce.

Un aspetto critico dell'articolo riguarda i limiti degli agenti: "Il modello non troverà in modo affidabile ciò che gli strumenti non possono vedere." Mezzalira nota che il modello ha correttamente identificato i contratti come fuori dal proprio ambito di analisi, ma poi ha problematicamente dichiarato il sistema "pulito" — una falsa rassicurazione più pericolosa di un'ammissione esplicita dei limiti. Per scalare oltre 20 servizi, l'architettura consigliata evolve verso agenti per team con scoring deterministico condiviso e un orchestratore che gestisce le preoccupazioni cross-service. L'implementazione completa è disponibile su github.com/lucamezzalira/gcp-agents-architecture.

## Codice

Le regole architetturali dimostrative del sistema applicano:

```python
# Esempio di regola architetturale: ownership singola per provider esterno
architectural_rules = [
    Rule(
        name="single_service_per_external_provider",
        description="Each external provider must be owned by exactly one service",
        check=lambda services: all(
            count_owners(provider, services) == 1
            for provider in get_external_providers(services)
        )
    ),
    Rule(
        name="message_based_inter_service_communication",
        description="Services must communicate via messages, not HTTP calls",
        check=lambda traces: not any(
            is_synchronous_http(call)
            for call in get_inter_service_calls(traces)
        )
    ),
    Rule(
        name="layer_dependency_direction",
        description="Dependencies must flow inward (API → Domain → Infrastructure)",
        check=lambda service: validate_dependency_direction(service)
    )
]

# Scoring deterministico (funzione pura, nessun LLM)
def compute_architecture_score(service, rules):
    violations = [r for r in rules if not r.check(service)]
    weights = {r.name: r.weight for r in rules}
    score = 1.0 - sum(weights[v.name] for v in violations)
    return ArchitectureScore(score=score, violations=violations)
```
