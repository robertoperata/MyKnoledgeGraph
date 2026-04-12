# Comunicazione tra Microservizi
## Pattern architetturali per aggregazione dati e orchestrazione

*Summary — Architettura Event-Driven*

---

## 1. Il problema: aggregare dati da più microservizi

In un'architettura a microservizi ogni servizio è owner esclusivo dei propri dati e dispone di un database indipendente. Quando un microservizio deve elaborare una richiesta che richiede informazioni appartenenti ad altri domini, non può accedere direttamente ai loro database: deve coordinarsi attraverso meccanismi espliciti di comunicazione.

Questo problema si presenta in modo diverso a seconda di come il microservizio viene ingaggiato.

### Tipi di ingaggio

- **API sincrona (HTTP/REST/gRPC):** un client chiama un endpoint e attende una risposta immediata. Il microservizio deve aggregare i dati e rispondere entro il timeout della connessione.
- **Evento Kafka o coda (SQS, RabbitMQ):** il microservizio viene svegliato da un messaggio su un topic. Non c'è un chiamante in attesa — il risultato può essere pubblicato su un altro topic o persistito.
- **Batch schedulato (cron, Spring Batch):** il microservizio viene eseguito periodicamente. Ha tempo sufficiente per aggregare dati, ma deve gestire volumi potenzialmente elevati.
- **Evento di sistema (saga step, workflow engine):** il microservizio riceve un comando nell'ambito di un flusso orchestrato più ampio (es. Temporal, AWS Step Functions).

### La sfida

In tutti questi casi il microservizio si trova davanti allo stesso nodo: come ottenere dati da altri servizi senza creare accoppiamento forte, senza violare la separazione dei domini, e mantenendo latenze accettabili?

---

## 2. I pattern disponibili

Esistono quattro famiglie di pattern per risolvere questo problema, ciascuna con caratteristiche diverse in termini di consistenza dei dati, latenza, complessità implementativa e scalabilità.

- **Request-Reply con Correlation ID:** il microservizio pubblica richieste su topic Kafka, aspetta le risposte usando CompletableFuture, e le collega tramite un identificatore univoco.
- **CQRS + Read Model:** il microservizio mantiene una copia locale denormalizzata dei dati altrui, aggiornata in tempo reale dagli eventi Kafka. Nessuna chiamata remota a runtime.
- **API Composition / BFF:** un livello intermedio aggrega chiamate REST verso più servizi e restituisce una risposta composta.
- **Event-Driven con risposta asincrona (202 Accepted + Webhook/SSE):** per operazioni lunghe, il servizio accetta la richiesta e notifica il client quando il risultato è pronto.

---

## 3. Descrizione dettagliata dei pattern

### 3.1 Request-Reply con Correlation ID

Il microservizio pubblica messaggi su topic Kafka dedicati alle richieste, includendo in ogni messaggio un `correlationId` univoco (UUID). Salva nella propria memoria una mappa `correlationId → CompletableFuture`. Un consumer thread separato ascolta i topic di risposta: quando arriva un messaggio, cerca il future corrispondente nella mappa e lo completa. Il thread HTTP che aveva avviato la richiesta era sospeso su `future.get()` e si risveglia automaticamente.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     Request-Reply con Correlation ID                        │
│                                                                             │
│  ┌─────────────┐    put(corrId,future)   ┌──────────────┐                  │
│  │ Thread HTTP │ ──────────────────────► │   HashMap    │                  │
│  │handleRequest│                         │corrId→Future │                  │
│  └─────────────┘                         └──────────────┘                  │
│         │                                                                   │
│         │ publish(corrId, data)                                             │
│         ▼                                                                   │
│  ┌─────────────┐     consume msg         ┌──────────────┐  ┌────────────┐  │
│  │    Kafka    │ ──────────────────────► │   Servizio B │  │ Servizio C │  │
│  │request topic│                         └──────────────┘  └────────────┘  │
│  │response tpc │ ◄──────────────────────────────────────────────────────── │
│  └─────────────┘     response(corrId)         (asincrono)                  │
│         │                                                                   │
│         │ consume response                                                  │
│         ▼                                                                   │
│  ┌─────────────┐  get(corrId)→future     ┌──────────────┐                  │
│  │  Consumer   │ ──────────────────────► │   HashMap    │                  │
│  │   Thread    │    future.complete()    └──────────────┘                  │
│  └─────────────┘                                                            │
│         │                                                                   │
│         │ future completato → Thread HTTP si sveglia                        │
│         ▼                                                                   │
│  ┌────────────────────────────────────────────────┐                        │
│  │   CompletableFuture.allOf(futureB, futureC)    │                        │
│  │   tempo totale = max(latenza B, latenza C)     │                        │
│  └────────────────────────────────────────────────┘                        │
│                                                                             │
│  ⚠ Ordine critico: ① crea future  ② metti in mappa  ③ pubblica su Kafka   │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### Meccanismo del Correlation ID

L'ordine di esecuzione è critico: il future deve essere inserito nella mappa **prima** di pubblicare il messaggio Kafka, altrimenti una risposta molto rapida potrebbe arrivare prima che il future esista, andando persa.

```java
// 1. Crea i future e li registra nella mappa
CompletableFuture<ResponseData> futureB = new CompletableFuture<>();
CompletableFuture<ResponseData> futureC = new CompletableFuture<>();
pendingRequests.put(correlationId + ":B", futureB);
pendingRequests.put(correlationId + ":C", futureC);

// 2. Solo DOPO pubblica su Kafka
kafkaTemplate.send("topic.B.request", new RequestMessage(correlationId, input));
kafkaTemplate.send("topic.C.request", new RequestMessage(correlationId, input));

// 3. Aspetta entrambi in parallelo (non in serie)
CompletableFuture.allOf(futureB, futureC).get(3, TimeUnit.SECONDS);
return aggregateResults(futureB.get(), futureC.get());
```

#### Il consumer che completa i future

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

#### Pregi e difetti

| Pregi | Difetti |
|---|---|
| Dati sempre freschi al momento della chiamata | Latenza dipendente dai servizi remoti |
| Implementazione relativamente diretta | Gestione timeout e cleanup della mappa |
| Funziona senza duplicazione di dati | Scalabilità limitata dai thread HTTP in attesa |
| Timeout esplicito gestibile | Fragilità se un servizio è down |

#### Quando usarlo

- Dati che devono essere freschi al momento esatto della richiesta (es. saldo, stock in tempo reale)
- Flussi sincroni dove il client attende una risposta immediata
- Casi dove la consistenza forte è obbligatoria per regole di business

---

### 3.2 CQRS + Read Model

Il microservizio mantiene localmente una proiezione denormalizzata dei dati che appartengono ad altri domini. Questa proiezione viene aggiornata in tempo reale ascoltando gli eventi Kafka pubblicati dai servizi owner. Quando arriva una richiesta, il microservizio legge dal suo database locale — nessuna chiamata remota, latenza zero a runtime.

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                            CQRS + Read Model                                 │
│                                                                              │
│   Write side (owner del dato)    │    Read side (consumer)                  │
│                                  │                                           │
│  ┌────────────┐                  │   ┌─────────────────┐   ┌─────────────┐  │
│  │ Servizio A │ ──pubblica──►  Kafka ──consume──► │   Servizio B    │   │   Client   │  │
│  │ owner dato │   evento         │   │  Projector      │◄──│  request   │  │
│  └────────────┘                  │   └─────────────────┘   └─────────────┘  │
│        │                         │           │                    │          │
│        ▼                         │           ▼                    │          │
│  ┌────────────┐                  │   ┌─────────────────┐          │          │
│  │  DB svc A  │                  │   │  Read Model DB  │          │          │
│  │normalizzato│                  │   │ denormalizzato  │◄─legge───┘          │
│  │fonte verità│                  │   │latenza zero     │                     │
│  └────────────┘                  │   └─────────────────┘                     │
│                                  │                                           │
│  ⚠ Trade-off: Eventual Consistency — il read model è aggiornato              │
│    con leggero ritardo rispetto alla fonte di verità                         │
└──────────────────────────────────────────────────────────────────────────────┘
```

#### Cosa significa denormalizzato

La struttura del read model non è progettata per eliminare la ridondanza, ma per soddisfare esattamente la query o il calcolo che il microservizio deve eseguire. Tutti i dati necessari sono già aggregati in un unico documento.

```java
// Read model ottimizzato per business logic (pricing)
public class CustomerPurchaseProfile {
    String customerId;
    int totalOrdersLast90Days;
    BigDecimal totalSpentLast12Months;
    String customerTier; // BRONZE, SILVER, GOLD
}

// Il Projector aggiorna il read model ascoltando gli eventi
@KafkaListener(topics = "orders.events")
public void onOrderEvent(OrderEvent event) {
    if (event.getType() == ORDER_COMPLETED) {
        CustomerPurchaseProfile profile = repo.findByCustomerId(event.getCustomerId())
            .orElse(new CustomerPurchaseProfile());
        profile.incrementOrderCount();
        profile.addToTotalSpent(event.getTotalAmount());
        profile.recalculateTier();
        repo.save(profile);
    }
}
```

#### Pregi e difetti

| Pregi | Difetti |
|---|---|
| Latenza zero a runtime — lettura locale | Consistenza eventuale — il dato può essere leggermente in ritardo |
| Isolamento completo da downtime degli altri servizi | Complessità del projector e bootstrap iniziale |
| Scalabilità elevatissima in lettura | Storage aggiuntivo per ogni read model |
| Adatto sia per UI che per business logic pura | Richiede strategia di replay eventi in caso di bug |

#### Quando usarlo

- Letture frequenti di dati che appartengono ad altri domini
- Business logic che non richiede dati freschi al millisecondo (pricing, raccomandazioni, fraud detection)
- Quando si vuole isolare il microservizio da downtime temporanei dei servizi dipendenti

---

### 3.3 API Composition / BFF

Un livello intermedio — API Gateway o Backend For Frontend (BFF) — aggrega chiamate REST o gRPC verso più servizi e restituisce al client una risposta composta. I microservizi non si conoscono tra loro. Il BFF puro è stateless e non ha un database proprio. Nella variante avanzata, il BFF mantiene un read model locale costruito con CQRS.

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                          API Composition / BFF                               │
│                                                                              │
│                    ┌─────────────────────────────────┐                      │
│                    │   BFF — Backend For Frontend    │                      │
│  ┌─────────┐       │  ┌─────────────┐ ┌───────────┐ │   ┌────────────┐     │
│  │ Client  │──────►│  │  API Layer  │►│Aggregator │ │──►│ Servizio A │     │
│  │Web/Mob. │◄──────│  │routing,auth │ │allOf(A,B) │ │   └────────────┘     │
│  └─────────┘       │  └─────────────┘ └───────────┘ │                      │
│                    │                                 │──►┌────────────┐     │
│                    │  ┌─────────────────────────┐   │   │ Servizio B │     │
│                    │  │ Read Model (variante)   │   │   └────────────┘     │
│                    │  │ Projector Kafka          │   │                      │
│                    │  │ dati già aggregati local │   │──►┌────────────┐     │
│                    │  └─────────────────────────┘   │   │ Servizio C │     │
│                    └─────────────────────────────────┘   └────────────┘     │
│                                                                              │
│  BFF puro: stateless, ogni request chiama i servizi → semplicità            │
│  BFF + Read Model: resiliente a downtime, latenza zero, eventual consistency│
└──────────────────────────────────────────────────────────────────────────────┘
```

```java
// BFF puro: aggrega in parallelo con CompletableFuture
public OrderDetailResponse getOrderDetail(String orderId) {
    CompletableFuture<Order> orderFuture =
        CompletableFuture.supplyAsync(() -> orderClient.getOrder(orderId));
    CompletableFuture<Customer> customerFuture =
        CompletableFuture.supplyAsync(() -> customerClient.getCustomer(orderId));

    CompletableFuture.allOf(orderFuture, customerFuture).join();
    return compose(orderFuture.get(), customerFuture.get());
}
```

#### Pregi e difetti

| Pregi | Difetti |
|---|---|
| Semplicità implementativa (BFF puro) | Componente aggiuntivo da gestire e scalare |
| I microservizi restano disaccoppiati tra loro | BFF puro: latenza dipendente dai servizi remoti |
| Facile adattare la risposta a diversi client | Rischio di accumulare logica di dominio nel BFF |
| Con read model: resiliente ai downtime | Con read model: consistenza eventuale |

#### Quando usarlo

- Aggregazione di dati per client con esigenze diverse (mobile vs web vs third-party)
- Quando i microservizi hanno già API REST ben definite
- Per adattare il formato dei dati senza modificare i servizi di dominio

---

### 3.4 Event-Driven con risposta asincrona (202 Accepted + Webhook/SSE)

Quando la computazione è lunga e non è possibile tenere aperta una connessione HTTP, il microservizio accetta la richiesta con un `202 Accepted`, restituisce immediatamente un `jobId`, e notifica il client tramite webhook o Server-Sent Events quando il risultato è pronto.

```
┌──────────────────────────────────────────────────────────────────────────────┐
│              Event-Driven con risposta asincrona (202 + Webhook/SSE)         │
│                                                                              │
│   Client          Microservizio        Kafka / Queue     Servizi dipendenti  │
│     │                   │                   │                   │            │
│     │ POST /compute      │                   │                   │            │
│     │──────────────────►│                   │                   │            │
│     │◄──────────────────│ 202 Accepted       │                   │            │
│     │  { jobId: "xyz" } │                   │                   │            │
│     │                   │ pubblica evento    │                   │            │
│     │                   │──────────────────►│                   │            │
│     │                   │                   │ elaborazione      │            │
│     │ GET /jobs/xyz      │                   │ asincrona...      │            │
│     │──────────────────►│                   │──────────────────►│            │
│     │◄── PENDING ───────│                   │◄── response ──────│            │
│     │                   │                   │                   │            │
│     │                   │◄── risultato pronto ─────────────────              │
│     │◄── Webhook/SSE ───│ notifica client                                    │
│     │                   │                                                    │
│     │ GET /jobs/xyz      │                                                    │
│     │──────────────────►│                                                    │
│     │◄── COMPLETED ─────│ { result: ... }                                    │
│                                                                              │
│  Strategie di notifica: Polling · Webhook · Server-Sent Events · WebSocket  │
└──────────────────────────────────────────────────────────────────────────────┘
```

```java
// Controller: accetta e risponde subito
@PostMapping("/compute")
public ResponseEntity<JobResponse> startComputation(@RequestBody Input input) {
    String jobId = UUID.randomUUID().toString();
    jobRepository.save(new Job(jobId, PENDING));
    eventPublisher.publish(new ComputationRequestedEvent(jobId, input));
    return ResponseEntity.accepted().body(new JobResponse(jobId));
}

// Polling: il client controlla lo stato
@GetMapping("/jobs/{jobId}")
public JobStatusResponse getStatus(@PathVariable String jobId) {
    return jobRepository.findById(jobId).map(this::toResponse).orElseThrow();
}
```

#### Pregi e difetti

| Pregi | Difetti |
|---|---|
| Adatto a computazioni lunghe senza timeout HTTP | Il client deve implementare polling o webhook |
| Client non rimane bloccato in attesa | Complessità nella gestione dello stato del job |
| Scalabilità disaccoppiata dal tempo di elaborazione | Non adatto quando il risultato è necessario immediatamente |
| La richiesta non va persa anche se il servizio si riavvia | Richiede storage per lo stato dei job |

#### Quando usarlo

- Elaborazioni che durano più di 2-3 secondi
- Batch processing, generazione di report, elaborazione di file
- Quando il client può ricevere notifiche asincrone (mobile push, webhook)

---

## 4. Modalità di implementazione

### 4.1 Platform Thread (approccio classico)

L'approccio standard con Spring MVC e un pool di thread Tomcat. Ogni richiesta HTTP occupa un thread del pool per tutta la sua durata — incluso il tempo di attesa delle risposte Kafka.

```java
// Spring MVC classico: bloccante
CompletableFuture.allOf(futureB, futureC).get(3, TimeUnit.SECONDS);
// Il thread Tomcat è bloccato per tutta la durata dell'attesa
// Pool tipico: 200 thread → max ~200 richieste concorrenti in attesa
```

**Pregi:** semplicità del codice, stack trace leggibili, nessuna dipendenza aggiuntiva, debugging diretto.

**Difetti:** scalabilità limitata dal pool size, thread OS tenuti occupati durante l'attesa, non adatto ad alta concorrenza.

---

### 4.2 Virtual Thread (Java 21+)

I Virtual Thread sono gestiti dalla JVM, non dall'OS. Sono leggerissimi (pochi KB ciascuno) e quando si bloccano su un'operazione di attesa vengono smontati dal platform thread sottostante (carrier), che diventa libero di eseguire altri virtual thread. Il codice rimane identico a quello bloccante classico.

```yaml
# application.yml — una sola riga per abilitare in Spring Boot 3.2+
spring:
  threads:
    virtual:
      enabled: true
```

```java
// Il codice non cambia rispetto al platform thread
CompletableFuture.allOf(futureB, futureC).get(3, TimeUnit.SECONDS);
// Con virtual thread: .get() smonta il VT dal carrier OS
// Migliaia di richieste concorrenti con pochissimi carrier thread

// ATTENZIONE al pinning — evitare synchronized con operazioni bloccanti:

// DA EVITARE
synchronized (this) { future.get(); }  // pinning: carrier OS bloccato

// PREFERIRE
ReentrantLock lock = new ReentrantLock();
lock.lock();
try { future.get(); } finally { lock.unlock(); }
```

**Pregi:** scalabilità molto alta, codice identico al bloccante classico (nessuna riscrittura), debugging semplice, stack trace leggibili.

**Difetti:** richiede Java 21+, attenzione al pinning con `synchronized`, librerie legacy potrebbero causare pinning internamente.

---

### 4.3 WebFlux / Reactor (approccio reattivo)

Spring WebFlux usa un numero fisso di thread non bloccanti (pari ai core CPU). Tutto è basato su pipeline dichiarative di `Mono` e `Flux`. Offre il throughput massimo teorico ma richiede una riscrittura completa del codice.

```java
public Mono<AggregatedResponse> handleRequest(String input) {
    String correlationId = UUID.randomUUID().toString();

    Mono<ResponseData> monoB = registerPendingMono(correlationId + ":B");
    Mono<ResponseData> monoC = registerPendingMono(correlationId + ":C");

    kafkaTemplate.send("topic.B.request", new RequestMessage(correlationId, input));
    kafkaTemplate.send("topic.C.request", new RequestMessage(correlationId, input));

    return Mono.zip(monoB, monoC)
               .map(tuple -> aggregateResults(tuple.getT1(), tuple.getT2()))
               .timeout(Duration.ofSeconds(3));
}
```

**Pregi:** throughput massimo teorico, nessun thread bloccato mai, ideale per latenze ultra-basse ad altissima concorrenza.

**Difetti:** curva di apprendimento alta, codebase completamente diverso da Spring MVC, stack trace difficili da leggere, tutte le librerie devono essere reattive.

---

### 4.4 Confronto tra le implementazioni

| Aspetto | Platform Thread | Virtual Thread | WebFlux / Reactor |
|---|---|---|---|
| Stile del codice | Imperativo bloccante | Imperativo bloccante | Funzionale reattivo |
| Modifica al codice | Nessuna | Quasi nulla (1 riga config) | Riscrittura completa |
| Scalabilità | Limitata (pool size) | Molto alta | Massima |
| Debugging | Semplice | Semplice | Difficile |
| Java minimo | 8+ | 21+ | 8+ (Spring 5+) |
| Librerie compatibili | Tutte | Tutte (attenzione pinning) | Solo reattive |
| Quando sceglierlo | Semplicità, codice esistente | Nuovo codice, alta concorrenza | Latenza ultra-bassa, team esperto |

---

## 5. Tabella riassuntiva dei pattern

| Pattern | Quando usarlo | Consistenza | Complessità | Scalabilità |
|---|---|---|---|---|
| **Request-Reply + Correlation ID** | Dati freschi obbligatori, risposta sincrona | Forte | Media | Alta |
| **CQRS + Read Model** | Letture frequenti da domini esterni, latenza zero | Eventuale | Media-Alta | Molto alta |
| **API Composition / BFF** | Aggregazione semplice, servizi REST veloci | Forte | Bassa | Media |
| **BFF + Read Model** | Molte letture aggregate, isolamento da downtime | Eventuale | Alta | Alta |
| **202 Accepted + Webhook/SSE** | Computazioni lunghe, client asincrono | Forte | Media | Alta |
| **Saga Pattern** | Transazioni distribuite multi-servizio (solo scrittura) | Eventuale | Alta | Bassa |

---

*In sistemi reali questi pattern coesistono. Un microservizio può usare CQRS per i dati che legge frequentemente, Request-Reply per i pochi dati che devono essere freschi al momento della computazione, e 202 Accepted per esporre operazioni lente verso il client. La scelta dipende sempre da: quanto devono essere freschi i dati, quanto può aspettare il chiamante, e quale throughput deve sostenere il sistema.*
