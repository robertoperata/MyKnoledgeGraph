---
tags:
  - event-driven
  - architecture
  - cloud
  - aws
  - distributed-systems
  - streaming
type: article
author: Teena Idnani
source: https://www.infoq.com/articles/multi-cloud-event-driven-architectures/
date: 2025-11-19
---

# Building Distributed Event-Driven Architectures across Multi-Cloud Boundaries

## Sunto

L'articolo affronta le sfide architetturali dell'implementazione di sistemi *event-driven* che attraversano i confini di più cloud provider. Il dato di partenza è significativo: l'86% delle organizzazioni opera già in ambienti multi-cloud, solo il 12% mantiene un deployment su singolo cloud, e il 70% usa architetture ibride che combinano sistemi on-premise con più provider. Il multi-cloud non è più un'eccezione ma lo standard.

Il caso di studio è **FinBank**, una banca centenaria fittizia in fase di modernizzazione: i sistemi core banking rimangono on-premise, il risk management migra su AWS, analytics e BI si spostano su Azure. Questo scenario realistico serve da filo conduttore per esplorare quattro sfide critiche che ogni architettura event-driven multi-cloud deve affrontare.

**1. Ottimizzazione della latenza** — La latenza inter-cloud richiede ottimizzazioni a livello di codice, non solo di networking. Le tecniche chiave includono: compressione dei messaggi (Snappy), batching ottimizzato (riduzione del 40-60% della latenza), timeout calibrati, e partitioning per account per un caching efficace.

**2. Resilienza** — La resilienza non riguarda solo la gestione dei fallimenti in corso, ma soprattutto il **recovery dopo le interruzioni**. Gli strumenti principali sono: *event store* per la persistenza persistente (Outbox Pattern, Kafka retention), circuit breaker per prevenire fallimenti a cascata, e meccanismi di replay sistematico per il recovery automatico.

**3. Ordinamento degli eventi** — Il multi-cloud rende l'ordinamento non banale. Serve un approccio a più livelli: numeri di sequenza strettamente crescenti lato publisher, verifica delle sequenze e processing differito lato subscriber, partitioning per account per ordinamento intrinseco, e una scelta esplicita dello spettro di consistenza (forte vs. eventuale).

**4. Gestione dei duplicati** — Serve una strategia di difesa a quattro livelli: (1) il publisher garantisce eventi univoci con schema CloudEvents, (2) la configurazione del producer Kafka abilita la modalità idempotente, (3) il subscriber controlla i duplicati via tabelle di processo, (4) l'handler è implementato in modo idempotente.

L'articolo introduce il framework **DEPOSITS** come principi operativi: Design for failure, Embrace event stores, Prioritize regular reviews, Observability first, Start small, Invest in robust event backbone, Team education, Success through continuous refinement. Le considerazioni trasversali includono security & compliance (IAM systems diversi tra provider), schema evolution con release cycle indipendenti, observability con distributed tracing cross-cloud, e il trade-off cloud-native vs. cloud-agnostic (performance/integrazione vs. flessibilità/portabilità).

---

## Esempi pratici

**Configurazione producer Kafka ottimizzata per latenza inter-cloud**

```csharp
var producerConfig = new ProducerConfig
{
    CompressionType = CompressionType.Snappy,
    BatchSize = 32768,
    LingerMs = 20,
    SocketTimeoutMs = 30000,
    DeliveryTimeoutMs = 30000,
    SocketNagleDisable = true
};
```

**Retry con backoff esponenziale per resilienza cross-cloud**

```csharp
await _resiliencePolicy
    .WaitAndRetryAsync(
        5,
        attempt => TimeSpan.FromSeconds(Math.Pow(2, attempt)),
        onRetry: (ex, timeSpan, attempt, ctx) =>
        {
            _logger.LogWarning(ex, "Retry {Attempt} publishing transaction event", attempt);
        })
    .ExecuteAsync(async () =>
    {
        var deliveryResult = await _kafkaProducer.ProduceAsync(
            topic, transactionEvent, key, producerConfig);
    });
```

---

## Link esterni

- [Flexera 2025 State of the Cloud Report](https://info.flexera.com/CM-REPORT-State-of-the-Cloud) — dati sull'adozione multi-cloud citati nell'articolo
- [CloudEvents Specification](https://cloudevents.io/) — standard per la struttura degli eventi usato per garantire unicità
- [Azure ExpressRoute](https://learn.microsoft.com/en-us/azure/expressroute/expressroute-introduction) — connettività privata verso Azure
- [AWS Direct Connect](https://docs.aws.amazon.com/directconnect/latest/UserGuide/Welcome.html) — connettività privata verso AWS
- [Azure Cosmos DB Consistency Levels](https://learn.microsoft.com/en-us/azure/cosmos-db/consistency-levels) — riferimento per lo spettro di consistenza
- [Video: Event-Driven Multi-Cloud (QCon)](https://www.infoq.com/presentations/event-driven-multi-cloud/) — presentazione correlata degli autori

---

## Immagini

Nessuna immagine presente
