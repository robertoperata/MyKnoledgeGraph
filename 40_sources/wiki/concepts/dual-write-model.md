---
title: Dual-Write Model
type: concept
tags: [thread1-microservices, thread5-ddd]
sources:
  - "[[youtube_building-bank-backend-revolut-yatsenko]]"
updated: 2026-05-08
related:
  - "[[patterns/event-driven]]"
  - "[[patterns/cqrs-read-model]]"
  - "[[concepts/domain-event]]"
  - "[[patterns/transactional-outbox]]"
---

# Dual-Write Model

## Definizione

Pattern in cui ogni operazione scrive **contemporaneamente** sia nell'event log (per audit trail e architettura event-driven) sia nello stato corrente (per query veloci senza ricostruzione). È il compromesso pragmatico tra event sourcing puro e architettura stateful classica.

```
Operazione di business
    │
    ├──► Event Log (immutable, append-only)   → per audit, replay, event-driven
    └──► Current State (mutable)              → per query veloci, decisioni immediate
```

## Motivazione

**Event sourcing puro** (solo event log, stato ricostruito dagli eventi) ha vantaggi teorici ma introduce latenza nella ricostruzione dello stato e complessità operativa. Per sistemi con vincoli di latenza stretti (es. autorizzazione carta: 500ms), ricostruire lo stato da eventi non è praticabile.

**Dual-write** offre il meglio dei due mondi:
- L'**event log** garantisce audit completo, possibilità di replay, e abilita l'architettura event-driven (i consumer si sottoscrivono agli eventi)
- Lo **stato corrente** permette decisioni immediate senza traversare la storia degli eventi

> "We have a dual model: we consistently write to the event log and at the same time write the new state. This allows quick decisions on state without recalculating, while events are used for audit log and event-driven architecture." — Vlad Yatsenko (Revolut)

## Differenza da Event Sourcing puro

| Aspetto | Event Sourcing puro | Dual-Write |
|---|---|---|
| **Fonte di verità** | Solo event log | Sia event log che stato corrente |
| **Ricostruzione stato** | Ricostruito da eventi (costoso) | Stato già disponibile (O(1)) |
| **Audit trail** | Sì, completo | Sì, completo |
| **Latenza lettura** | Alta (replay) o con snapshot | Bassa (lettura diretta) |
| **Complessità** | Alta | Medio-bassa |
| **Uso ideale** | Sistemi dove la storia è la fonte di verità | Sistemi con vincoli di latenza e necessità di audit |

## Differenza da CQRS

Il dual-write è complementare al CQRS ma opera a un livello diverso:
- **CQRS**: separa il write side (ownership del dato) dal read side (proiezione per query) — può essere inter-servizio
- **Dual-write**: opera all'interno dello stesso servizio/operazione — scrive in due destinazioni nello stesso momento

Il dual-write può essere visto come la strategia di sincronizzazione che alimenta il read model in CQRS: ogni write al command side aggiorna sia lo stato persistente che pubblica l'evento che il projector del read side consuma.

## Caso d'uso: pre-computation per decisioni time-critical

Il pattern è particolarmente potente quando combinato con la **pre-computation**: un consumer dell'event log calcola continuamente risultati (es. score di fraud detection, posizione di rischio) che saranno richiesti in futuro con vincoli di latenza stretti.

```
Transazione passata → Event Log → Consumer ML
                                    │ calcola score
                                    ▼
                              Pre-computed State
                                    │
                                    ▼ (quando arriva la prossima transazione)
                         Authorization Engine ← lookup O(1) → risposta in < 500ms
```

Revolut usa questo pattern per due casi critici:
- **Fraud detection** (Sherlock): score ML pre-calcolato per ogni utente, lookup istantaneo durante l'autorizzazione carta
- **Risk hedging**: posizione di esposizione valutaria aggiornata ad ogni evento, visibile al finance team in ~100ms

## Rischi

- **Inconsistenza transiente**: se il write allo stato corrente riesce ma la scrittura all'event log fallisce (o viceversa), i due si desincronizzano. Soluzione: [[patterns/transactional-outbox]] per garantire atomicità tra DB write e pubblicazione evento.
- **Duplicazione di storage**: i dati esistono in due forme. Accettabile se i benefici di latenza lo giustificano.

## Connessioni

- [[patterns/event-driven]] — il dual-write alimenta l'architettura event-driven: l'event log è il backbone degli eventi
- [[patterns/cqrs-read-model]] — il dual-write è il meccanismo di sincronizzazione tra write side e read side nel CQRS
- [[concepts/domain-event]] — gli eventi scritti nel log sono domain events
- [[patterns/transactional-outbox]] — garantisce atomicità tra la write allo stato corrente e la pubblicazione dell'evento nel log
