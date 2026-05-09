---
title: Testing Pyramid per Microservizi
type: pattern
tags:
  - thread1-microservices
sources:
  - "[[chris_richardson_cubes-hexagons-triangles-yow2019]]"
updated: 2026-05-07
related:
  - "[[concepts/independent-deployability]]"
  - "[[concepts/hexagonal-architecture]]"
  - "[[technologies/istio]]"
---

# Testing Pyramid per Microservizi

## Problema che risolve

I microservizi aumentano la complessità del testing: ogni servizio ha interfacce esterne (API, eventi), dipendenze da altri servizi, e comportamenti distribuiti. Senza una strategia deliberata, si finisce con una suite di test end-to-end lenta, fragile e costosa che rallenta il delivery — contraddicendo il principio di independent deployability.

> "If you attempt to do microservices without automated testing, it is self-defeating and it is risky. 72% of companies do not extensively do automated testing, and ironically 90% want to do DevOps — there's a total disconnect." — Chris Richardson

## Struttura

```
         ╱─────╲
        ╱ E2E   ╲         Pochi — lenti, fragili, costosi
       ╱─────────╲
      ╱ Component ╲       Molti — testano il servizio in isolamento
     ╱─────────────╲
    ╱   Contract    ╲     Il confine critico: consumer-driven contract testing
   ╱─────────────────╲
  ╱   Unit Tests       ╲  Moltissimi — veloci, affidabili, economici
 ╱─────────────────────╲
```

Più si scende nella piramide:
- Più i test sono veloci da eseguire
- Più sono affidabili (non falliscono casualmente)
- Più sono semplici da scrivere e mantenere
- Più sono economici

## I livelli nel dettaglio

### Unit Tests (base)
- Testano le classi e i componenti della business logic in isolamento
- I mock rimpiazzano le dipendenze esterne (database, altri servizi)
- Nell'[[concepts/hexagonal-architecture]], si invocano i **port** direttamente
- Devono essere la maggioranza assoluta della suite

### Contract Tests (livello 2)
- Il confine **critico** per i microservizi: testano la compatibilità tra un consumer e un provider senza deploy end-to-end
- **Consumer-driven contract testing** (Spring Cloud Contract): il consumer definisce le sue aspettative (il "contratto"), il provider verifica che le rispetti
- Sostituiscono la maggior parte dei test di integrazione cross-servizio
- Rilevano breaking changes prima del deploy

### Component Tests (livello 3)
- Testano il servizio completo in isolamento: il servizio è reale, le sue dipendenze esterne sono mock/stub
- Verificano il comportamento end-to-end **del singolo servizio** (non del sistema)
- Relativamente lenti ma molto più stabili degli E2E

### End-to-End Tests (apice)
- Testano il sistema completo con tutti i servizi reali
- Obiettivo: **eliminare o minimizzare** — sono lenti, fragili, costosi
- Usarli solo per i happy path più critici
- Sostituiti in gran parte dai component + contract tests

## Canary Deployment come test in produzione

Richardson distingue tra **deployment** (codice in produzione) e **release** (traffico instradato). Questa separazione è una forma di "test in produzione" con safety net:

```
v1 ── 100% traffico
v2 (deployed) → test interni → 5% → 20% → 100%
                                    ↕
                monitoring automatico (latency, 500s)
                                    ↕
                rollback automatico se soglie superate
```

Tool: service mesh ([[technologies/istio]]), traffic routing intelligente, [[concepts/observability]] integrata.

## Trade-off

| Approccio | Vantaggio | Svantaggio |
|---|---|---|
| Più unit tests | Veloci, feedback immediato | Non testano integrazione |
| Consumer-driven contracts | Testano il confine senza deploy completo | Richiedono collaborazione tra team |
| Canary deployment | Test reale con traffico reale | Richiede monitoring maturo, rischio su alcuni utenti |

## Connessioni

- [[concepts/independent-deployability]] — la testing pyramid abilita il deploy indipendente con confidenza
- [[concepts/hexagonal-architecture]] — l'architettura hexagonal rende i unit test e component test naturali (porte = punti di test)
- [[technologies/istio]] — il service mesh abilita il canary deployment e il traffic splitting per i test in produzione
- [[concepts/observability]] — il monitoring automatico nel canary deployment si basa sulla piattaforma di osservabilità
