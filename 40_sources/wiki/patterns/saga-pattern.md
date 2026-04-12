---
title: Saga Pattern
type: pattern
tags: [thread1-microservices, thread5-ddd]
sources:
  - "[[Microservices Collaboration]]"
  - "[[microservizi_pattern_summary]]"
updated: 2026-04-09
related:
  - "[[concepts/domain-event]]"
  - "[[patterns/event-driven]]"
  - "[[concepts/aggregate]]"
---

# Saga Pattern

## Problema che risolve

In un sistema a microservizi, una transazione di business può attraversare più servizi (ognuno col proprio database). Non è possibile usare le transazioni ACID tradizionali (2-Phase Commit) perché i database sono separati e il 2PC è fragile e costoso in un sistema distribuito. La Saga è la soluzione per gestire transazioni distribuite con compensazione in caso di failure.

## Struttura

Una saga è una sequenza di transazioni locali. Se una transazione fallisce, le precedenti vengono annullate tramite **compensating transactions**.

Componenti necessari:
1. **Execution Log**: registro di ciò che è già accaduto
2. **Execution Coordinator**: componente che decide cosa fare dopo

## Orchestration vs Choreography

### Orchestration (command & control)
Un orchestratore centrale dirige ogni passo della saga.

```
Orchestratore
    │ → OrderService.reserve()
    │ ← OK
    │ → PaymentService.charge()
    │ ← OK
    │ → ShippingService.ship()
    │ ← FAIL
    │ → PaymentService.refund()    (compensating)
    │ → OrderService.cancel()     (compensating)
```

| Vantaggi | Svantaggi |
|---|---|
| Rappresentazione esplicita del business process | Può diventare troppo accoppiato |
| Errori visibili inline | Rischio di orchestratore "god service" |
| Adatto a scenari complessi | Problematico per organizzazioni grandi |

### Choreography (hippi drum circle)
Ogni servizio pubblica eventi e reagisce agli eventi altrui, senza coordinatore centrale.

```
OrderService ──OrderCreated──► PaymentService ──PaymentDone──► ShippingService
ShippingService ──ShipFailed──► PaymentService (refund) ──PaymentRefunded──► OrderService (cancel)
```

| Vantaggi | Svantaggi |
|---|---|
| Altamente decoupled | Business process non esplicito da nessuna parte |
| Intelligenza distribuita | Capire lo stato completo è complesso |
| Nessun single point of failure | Difficile da debuggare |

## Quando usarlo

Transazioni distribuite multi-servizio che richiedono:
- Atomicità cross-service senza distributed transactions
- Rollback su failure parziali

## Quando non usarlo

Le saga hanno **bassa scalabilità** rispetto ad altri pattern (compensating transactions costose, coordination overhead). Non usarle quando si può risolvere con:
- Un singolo aggregate nel servizio (se i dati sono locali)
- Eventual consistency senza compensazione

## Connessioni

- [[concepts/aggregate]] — ogni passo della saga è una transazione locale su un aggregate
- [[concepts/domain-event]] — la choreography saga si basa sui domain events
- [[patterns/event-driven]] — il substrate su cui operano le saga a choreography
- [[patterns/cqrs-read-model]] — spesso usato per il "read side" in sistemi che usano saga per il write side
