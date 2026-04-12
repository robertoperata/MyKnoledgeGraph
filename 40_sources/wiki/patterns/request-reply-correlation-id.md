---
title: Request-Reply con Correlation ID
type: pattern
tags: [thread1-microservices, thread2-java]
sources:
  - "[[microservizi_pattern_summary]]"
  - "[[Microservices Collaboration]]"
updated: 2026-04-09
related:
  - "[[concepts/completable-future]]"
  - "[[concepts/virtual-threads]]"
  - "[[patterns/event-driven]]"
  - "[[technologies/kafka]]"
---

# Request-Reply con Correlation ID

## Problema che risolve

Un microservizio che riceve una richiesta HTTP deve raccogliere dati da altri servizi (che parlano via Kafka), aggregarli e rispondere entro un timeout. Il problema: come correlare una risposta asincrona Kafka con la richiesta HTTP originale?

## Struttura

```
Thread HTTP          HashMap<correlationId→Future>
     │ put(corrId, future) ──────────────────────┐
     │                                           │
     │ publish(corrId, data)                     │
     ▼                                           │
  Kafka                     Servizio B/C         │
  request topic ──consume──►(elaborazione)       │
  response topic ◄──────────(risposta)           │
     │                                           │
     │ consume response                          │
     ▼                                           │
  Consumer Thread  get(corrId)→future ───────────┘
                   future.complete()
     │
     ▼
CompletableFuture.allOf(futureB, futureC)
→ Thread HTTP si sveglia
```

## Implementazione critica

L'**ordine di esecuzione** è critico:

```java
// 1. Crea i future e li registra PRIMA di pubblicare
CompletableFuture<ResponseData> futureB = new CompletableFuture<>();
pendingRequests.put(correlationId + ":B", futureB);   // ① put in map

// 2. Solo DOPO pubblica su Kafka
kafkaTemplate.send("topic.B.request", new RequestMessage(correlationId, input));  // ② publish

// 3. Aspetta entrambi in parallelo (tempo totale = max, non somma)
CompletableFuture.allOf(futureB, futureC).get(3, TimeUnit.SECONDS);  // ③ wait
return aggregateResults(futureB.get(), futureC.get());
```

Se si inverte ① e ②, una risposta molto veloce può arrivare prima che il future esista nella mappa, andando persa.

## Il consumer che completa i future

```java
@KafkaListener(topics = "topic.B.response")
public void onResponseB(ResponseMessage msg) {
    CompletableFuture<ResponseData> future =
        pendingRequests.get(msg.getCorrelationId() + ":B");
    if (future != null) {
        future.complete(msg.getData()); // sveglia il thread HTTP
    }
}
```

## Implementazioni a confronto

| Aspetto | Platform Thread | Virtual Thread | WebFlux |
|---|---|---|---|
| Stile codice | Imperativo bloccante | Imperativo bloccante | Funzionale reattivo |
| Scalabilità | Pool ~200 thread | Molto alta | Massima |
| Modifica al codice | Nessuna | 1 riga config | Riscrittura |
| Debugging | Semplice | Semplice | Difficile |

**Virtual Thread**: `spring.threads.virtual.enabled: true` — nessuna altra modifica al codice. Con `future.get()` il virtual thread si smonta dal carrier, liberandolo per altre richieste.

## Trade-off

| Vantaggio | Svantaggio |
|---|---|
| Dati sempre freschi (strong consistency) | Latenza dipendente dai servizi remoti |
| Implementazione diretta | Gestione timeout + cleanup della mappa |
| Funziona senza duplicazione dati | Fragilità se un servizio è down |

## Quando usarlo

- Dati che devono essere freschi al momento esatto della richiesta (saldo, stock in tempo reale)
- Flussi sincroni dove il client attende una risposta immediata
- Casi dove la strong consistency è obbligatoria per regole di business

## Quando preferire alternative

- Letture frequenti degli stessi dati → **[[patterns/cqrs-read-model]]**
- Aggregazione semplice via REST → **[[patterns/api-composition-bff]]**
- Elaborazioni lunghe → **Event-Driven con 202 Accepted + Webhook**

## Connessioni

- [[concepts/completable-future]] — il meccanismo Java che implementa il pattern
- [[concepts/virtual-threads]] — abilitano alta concorrenza con codice invariato
- [[technologies/kafka]] — il broker per il trasporto dei messaggi request/response
- [[patterns/event-driven]] — questo pattern è event-driven applicato al request/response
