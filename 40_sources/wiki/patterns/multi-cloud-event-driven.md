---
title: Multi-Cloud Event-Driven Architecture
type: pattern
tags: [thread1-microservices, thread3-cloud]
sources:
  - "[[infoq_multi-cloud-event-driven-architectures]]"
updated: 2026-04-17
related:
  - "[[patterns/event-driven]]"
  - "[[technologies/kafka]]"
  - "[[concepts/independent-deployability]]"
---

# Multi-Cloud Event-Driven Architecture

## Problema che risolve

L'86% delle organizzazioni opera già in ambienti multi-cloud. Le architetture event-driven devono attraversare i confini tra provider (AWS, Azure, on-premise) mantenendo latenza bassa, resilienza e ordinamento degli eventi.

## Quattro sfide critiche

### 1. Ottimizzazione della latenza
La latenza inter-cloud richiede ottimizzazioni a livello di codice, non solo di networking:

```csharp
var producerConfig = new ProducerConfig
{
    CompressionType = CompressionType.Snappy,  // riduzione banda
    BatchSize = 32768,
    LingerMs = 20,  // batching: -40-60% latenza
    SocketNagleDisable = true
};
```

### 2. Resilienza (recovery, non solo availability)
- **Event store** per persistenza (Outbox Pattern, Kafka retention)
- **Circuit breaker** per prevenire fallimenti a cascata
- **Replay sistematico** per recovery automatico dopo interruzioni

### 3. Ordinamento degli eventi
Approccio a più livelli:
- Publisher: numeri di sequenza strettamente crescenti
- Subscriber: verifica sequenze + processing differito
- Partitioning per account per ordinamento intrinseco
- Scelta esplicita dello spettro di consistenza (forte vs. eventuale)

### 4. Gestione dei duplicati — difesa a 4 livelli
1. Publisher garantisce eventi univoci (schema CloudEvents)
2. Producer Kafka in modalità idempotente
3. Subscriber con tabelle di processo per duplicate checking
4. Handler idempotente

## Framework DEPOSITS

| Lettera | Principio |
|---|---|
| **D** | Design for failure |
| **E** | Embrace event stores |
| **P** | Prioritize regular reviews |
| **O** | Observability first |
| **S** | Start small, scale gradually |
| **I** | Invest in robust event backbone |
| **T** | Team education |
| **S** | Success through continuous refinement |

## Trade-off: cloud-native vs cloud-agnostic

| Approccio | Vantaggio | Svantaggio |
|---|---|---|
| Cloud-native | Performance, integrazione profonda | Lock-in al provider |
| Cloud-agnostic | Portabilità, flessibilità | Overhead, meno ottimizzato |

## Connessioni

- [[patterns/event-driven]] — questo pattern estende l'EDA ai confini multi-cloud
- [[technologies/kafka]] — Kafka come backbone cross-cloud con configurazione ottimizzata
- [[concepts/independent-deployability]] — il multi-cloud richiede release cycle indipendenti tra provider
