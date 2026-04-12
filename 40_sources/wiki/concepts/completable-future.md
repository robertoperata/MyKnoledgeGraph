---
title: CompletableFuture
type: concept
tags: [thread2-java, thread1-microservices]
sources:
  - "[[Java Threads Basics]]"
  - "[[Java Threads Demystified]]"
  - "[[microservizi_pattern_summary]]"
updated: 2026-04-09
related:
  - "[[concepts/java-concurrency]]"
  - "[[concepts/virtual-threads]]"
  - "[[patterns/request-reply-correlation-id]]"
  - "[[patterns/api-composition-bff]]"
---

# CompletableFuture

## Definizione

`CompletableFuture<T>` è un `Future` con funzionalità aggiuntive per comporre operazioni asincrone in modo dichiarativo. Permette di modellare workflow concorrenti complessi senza thread espliciti.

## API core

```java
// Avvio asincrono
CompletableFuture<String> cf = CompletableFuture.supplyAsync(() -> fetchData());

// Trasformazione del risultato
cf.thenApply(data -> process(data))          // transform
  .thenCompose(r -> fetchMore(r))             // chain another async op
  .thenAccept(System.out::println);           // consume (void)

// Gestione errori
cf.exceptionally(ex -> defaultValue)
  .handle((result, ex) -> ex != null ? handleError(ex) : result);

// Combinazione di più future
CompletableFuture.allOf(cf1, cf2, cf3).join();  // aspetta tutti
CompletableFuture.anyOf(cf1, cf2, cf3).join();  // aspetta il primo
```

## Pattern: aggregazione parallela

Il pattern classico per aggregare chiamate parallele:
```java
CompletableFuture<Order> orderFuture = 
    CompletableFuture.supplyAsync(() -> orderService.get(id));
CompletableFuture<Customer> customerFuture = 
    CompletableFuture.supplyAsync(() -> customerService.get(id));

CompletableFuture.allOf(orderFuture, customerFuture).join();
// Tempo totale = max(latenzaOrder, latenzaCustomer), non la somma
return compose(orderFuture.get(), customerFuture.get());
```

## Connessione con Request-Reply e Correlation ID

Il blog post sui microservizi mostra un uso sofisticato di `CompletableFuture` nel pattern **Request-Reply con Correlation ID**:
1. Si crea il `CompletableFuture` e lo si inserisce in una mappa `correlationId → Future` **prima** di pubblicare su Kafka
2. Il thread HTTP aspetta su `future.get()` (o `CompletableFuture.allOf(...)`)
3. Il consumer Kafka, alla ricezione della risposta, chiama `future.complete(data)` svegliando il thread HTTP

```java
// Ordine critico: ① crea future ② metti in mappa ③ pubblica su Kafka
CompletableFuture<ResponseData> futureB = new CompletableFuture<>();
pendingRequests.put(correlationId + ":B", futureB);  // PRIMA
kafkaTemplate.send("topic.B.request", new RequestMessage(correlationId, input));  // POI

// Completion lato consumer
@KafkaListener(topics = "topic.B.response")
public void onResponseB(ResponseMessage msg) {
    CompletableFuture<ResponseData> future = pendingRequests.get(msg.getCorrelationId() + ":B");
    if (future != null) future.complete(msg.getData());
}
```

> [!warning] Deprecazione parziale con Virtual Thread: con l'avvento dei virtual thread (Java 21+), molti usi di CompletableFuture per evitare il blocking possono essere semplificati usando codice bloccante classico su virtual thread. Ma `CompletableFuture.allOf()` per l'aggregazione parallela rimane utile.

## BFF con CompletableFuture

```java
// BFF puro: aggrega in parallelo
CompletableFuture<Order> orderFuture = 
    CompletableFuture.supplyAsync(() -> orderClient.getOrder(orderId));
CompletableFuture<Customer> customerFuture = 
    CompletableFuture.supplyAsync(() -> customerClient.getCustomer(orderId));
CompletableFuture.allOf(orderFuture, customerFuture).join();
return compose(orderFuture.get(), customerFuture.get());
```

## Connessioni

- [[concepts/java-concurrency]] — CompletableFuture fa parte dell'Executor Framework
- [[concepts/virtual-threads]] — con VT il codice bloccante può spesso sostituire CompletableFuture per la semplicità
- [[patterns/request-reply-correlation-id]] — uso avanzato di CompletableFuture come meccanismo di correlazione
- [[patterns/api-composition-bff]] — aggregazione parallela di chiamate REST
