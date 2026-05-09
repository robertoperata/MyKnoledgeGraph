---
tags:
  - microservices
  - event-driven
  - architecture
  - distributed-systems
type: article
author: Chris Richardson
source: https://www.youtube.com/watch?v=kyNL7yCvQQc
date: 2019-01-01
---

# Not Just Events: Developing Asynchronous Microservices

**Speaker:** Chris Richardson (autore di *Microservices Patterns*, creatore di eventuate.io)  
**Evento:** GOTO 2019  
**Durata:** 50:37  
**Lingua originale:** inglese

---

## Sintesi

Chris Richardson smonta tre equivoci comuni sull'architettura a microservizi: (1) i microservizi non sono solo REST, (2) non sono equivalenti agli eventi, (3) non sono equivalenti all'event sourcing. Il talk illustra le soluzioni ai due problemi fondamentali dei microservizi — **gestione delle transazioni distribuite** e **query che attraversano più servizi** — e mostra che entrambi richiedono messaggistica asincrona, ma con pattern di comunicazione radicalmente diversi: eventi domain (pub/sub) per la choreography, e request/reply asincrono per l'orchestration.

Il cuore del talk è la **Saga pattern**: la transazione distribuita diventa una sequenza di transazioni locali coordinate tramite messaggistica. Richardson descrive i due stili di coordinamento — choreography (basata su eventi, decentralizzata) e orchestration (basata su command/reply, centralizzata con un Saga Orchestrator) — con i rispettivi trade-off. Per le query, introduce **CQRS**: repliche dedicate ai diversi pattern di lettura, aggiornate tramite subscription agli eventi.

La fondazione tecnica che rende tutto questo possibile è il **Transactional Outbox pattern**: aggiornare il database e pubblicare messaggi in modo atomico, senza XA/2PC, usando una tabella outbox nella stessa transazione locale e un processo separato per la pubblicazione (tramite transaction log tailing o polling).

---

## Il problema: transazioni e query nei microservizi

Ogni servizio ha il suo **database privato**. Questo è il principio fondamentale che genera tutti i problemi.

```
Customer Service → Customer DB (schema privato)
Order Service    → Order DB    (schema privato)
```

**Problema transazioni**: per creare un ordine bisogna verificare il credito del cliente → richiede coordinamento tra Order Service e Customer Service. In un monolite è una semplice transazione ACID con SQL. In microservizi, non si può fare.

**Problema query**: trovare "ordini di un dato cliente" richiede join tra tabelle in database separati e privati. Non si può fare direttamente.

La soluzione a entrambi passa per la **messaggistica asincrona**.

---

## Saga Pattern — Transazioni distribuite

### Definizione

Una Saga è una sequenza di transazioni locali, una per ogni servizio partecipante. Se un passo fallisce, vengono eseguite **transazioni compensatorie** per annullare i passi precedenti.

```
Saga "Create Order":
  Step 1: Order Service   → crea ordine in stato PENDING  (semantic lock)
  Step 2: Customer Service → riserva credito
  Step 3: Order Service   → approva ordine (stato APPROVED)

Se Step 2 fallisce:
  Compensazione Step 1: Order Service → rigetta ordine (stato REJECTED)
```

### Proprietà delle Saga

Le Saga sono **ACD**, non ACID — manca la proprietà di **Isolation**. L'esecuzione di passi di saga diverse può interleave, aprendo a anomalie (lost updates, fuzzy reads). Soluzioni: **countermeasures** come il semantic lock (stato PENDING).

### Perché NON usare REST/sincronous per le Saga

- **Accoppiamento temporale**: entrambi i servizi devono essere up simultaneamente
- **Failure mode pericoloso**: se Order Service invia richiesta a Customer Service e poi crasha prima di ricevere la risposta, il sistema è in stato inconsistente
- Con messaggistica asincrona + message broker con **at-least-once delivery**, la Saga è garantita a completarsi eventualmente anche in caso di failure parziali

### Requisiti del message broker

1. **At-least-once delivery** — il broker riprova la consegna se il consumer è temporaneamente down
2. **Ordered delivery** — i messaggi per la stessa entità devono arrivare in ordine
3. **Scalabilità con ordering** — es. Kafka consumer groups

---

## Choreography vs Orchestration

### Choreography (basata su eventi, decentralizzata)

```
Client → Order Service
  Order Service → pubblica "OrderCreated" → order-events channel
    Customer Service ← subscribe → riserva credito → pubblica "CreditReserved"
      Order Service ← subscribe → approva ordine
```

**Vantaggi:**
- Semplice, naturale, loose coupling temporale
- Ottimo fit con event sourcing

**Svantaggi:**
- Logica distribuita: per capire una Saga devi leggere il codice di N servizi
- Dipendenze cicliche: Customer Service deve conoscere gli eventi di Order Service
- Non puoi "dire cosa fare" — puoi solo "annunciare cosa hai fatto" (comunicazione passivo-aggressiva)

### Orchestration (basata su command/reply, centralizzata)

```
Order Service crea CreateOrderSaga (oggetto persistente = state machine)
  CreateOrderSaga → invia command "ReserveCredit" → customer-commands channel
    Customer Service → esegue → invia reply "CreditReserved" → saga-reply channel
      CreateOrderSaga ← riceve reply → approva ordine
```

Il `CreateOrderSaga` è un **oggetto persistente** nel database (state machine). Il ciclo è:
1. Carica la Saga dal DB
2. Riceve il reply → transizione di stato
3. Determina il prossimo partecipante da invocare
4. Invia command message
5. Salva la Saga nel DB
6. Riparte dal punto 1

**Vantaggi:**
- Logica centralizzata in una classe (`CreateOrderSaga`)
- Riduce il coupling: Customer Service espone solo API per la gestione del credito (non conosce gli eventi di Order Service)
- Elimina le dipendenze cicliche

**Svantaggi:**
- Richiede un framework di Saga orchestration (es. Eventuate Tram)
- Rischio di mettere business logic nella Saga invece che nei service

> **Nota critica**: la Saga NON è un processo BPM di lunga durata. Deve completarsi in decine di millisecondi, non ore/giorni. Usare un workflow engine per le Saga è probabilmente sbagliato.

---

## CQRS — Query che attraversano più servizi

### API Composition (soluzione semplice)

Un API Composer (es. API Gateway) invoca i servizi coinvolti e assembla i risultati in memoria. Funziona bene per query semplici.

**Limiti**: query complesse (es. "clienti recenti con ordini > $X già spediti") richiedono join tra dataset potenzialmente grandi → inefficiente o impossibile con semplice API composition.

### CQRS — Command Query Responsibility Segregation

Mantenere una **replica read-optimized** dei dati, aggiornata subscribendo agli eventi dei servizi.

```
Customer Service → pubblica CustomerEvents
Order Service    → pubblica OrderEvents
                              ↓
                 CustomerOrderView Service
                 (replica MongoDB con dati denormalizzati)
                 → query ottimizzata per "ordini per cliente"
```

**Caratteristiche:**
- Il view store è **eliminabile e ricostruibile** (è una replica)
- Può usare il database più adatto: MongoDB per JSON, Elasticsearch per full-text, Neo4j per grafi
- In un sistema reale: N view services, ognuno ottimizzato per un pattern di query

**Trade-off principale — Replication Lag**: c'è un ritardo (es. 20ms) tra l'aggiornamento del command side e l'aggiornamento del view side. Un client che aggiorna e poi subito rilegge potrebbe ottenere i dati vecchi. Soluzione lato UI: aggiornare il modello client-side ottimisticamente dopo una command request riuscita.

---

## Transactional Outbox Pattern — La fondazione tecnica

### Il problema

Ogni passo di una Saga deve:
1. Aggiornare il suo database locale
2. Pubblicare un messaggio/evento al message broker

Questi due passi devono essere **atomici**. Non si può usare 2PC/XA (è esattamente quello che si cerca di evitare con le Saga).

### Soluzioni errate

- **Publish first, then update DB**: non funziona perché il service non può leggere le proprie scritture nella stessa transazione
- **Update DB, then publish**: se il processo crasha tra i due passi, il messaggio non viene mai pubblicato

### Transactional Outbox Pattern

```sql
BEGIN TRANSACTION;
  UPDATE orders SET status = 'APPROVED' WHERE id = ?;
  INSERT INTO outbox (aggregate_id, type, payload) VALUES (?, 'OrderApproved', ?);
COMMIT;
```

Un processo separato legge la tabella `outbox` e pubblica i messaggi al broker. I messaggi già pubblicati vengono marcati come tali (o eliminati).

**Risultato**: atomicità garantita dalla transazione locale ACID. Il service può leggere e scrivere nella stessa transazione locale.

### Due approcci per leggere l'outbox

| Approccio | Come funziona | Pro | Contro |
|---|---|---|---|
| **Transaction Log Tailing** | Legge il WAL/binlog del DB (Postgres WAL, MySQL binlog, DynamoDB Streams) | Zero latency, efficiente | Database-specific |
| **Polling** | `SELECT * FROM outbox WHERE published = false` ogni N ms | Universale (qualsiasi JDBC DB) | Polling frequency trade-off; possibile doppia pubblicazione in caso di crash |

Il doppio publish è gestibile: i message handler devono essere **idempotent** (ignorare duplicati), e i message broker comunque possono consegnare lo stesso messaggio più volte.

---

## Event Sourcing — Alternativa al Transactional Outbox

L'event sourcing è un approccio **event-centric** alla business logic e alla persistenza: le entità di dominio (Order, Customer) vengono persistite come **sequenza di eventi** invece che come stato corrente.

```
orders_events table:
  (order_id=1, event='OrderCreated', payload={...})
  (order_id=1, event='OrderApproved', payload={...})
  (order_id=1, event='OrderShipped', payload={...})
```

Per ricostruire lo stato corrente dell'ordine: caricare gli eventi e **riprodurli** (replay).

**Vantaggi**: naturale fit con choreography-based Saga, preserva la storia, auditing, time-travel queries.

**Svantaggi**:
- Approccio radicalmente diverso di programmazione
- Schema evolution degli eventi complessa
- Forza l'uso di CQRS (non si può fare query dirette sullo stato)
- Implementare orchestration-based Saga è più complesso
- **Kafka NON è un event store**: Kafka è eccellente come message broker ma non permette di recuperare eventi per ID (requisito fondamentale di un event store)

---

## Riepilogo: quando usare cosa

| Pattern | Problema | Comunicazione |
|---|---|---|
| **Choreography Saga** | Transazione distribuita semplice | Pub/sub su domain events |
| **Orchestration Saga** | Transazione distribuita complessa, multi-step | Async request/reply |
| **API Composition** | Query semplice su N servizi | Invocazione sincrona o asincrona |
| **CQRS** | Query complessa su N servizi, join tra grandi dataset | Subscription a eventi → view store |
| **Transactional Outbox** | Atomicità aggiornamento DB + pubblicazione messaggio | — (fondazione tecnica) |
| **Event Sourcing** | Persistenza event-centric + storia completa | — (approccio alternativo) |

---

## Citazioni notevoli

> "Microservices are more than REST. Microservices are more than events. Microservices are not equivalent to event sourcing. And Kafka is not an event store, despite claims to the contrary."  
> — Chris Richardson

> "With choreography, you can only say what you have done — not tell someone what to do. It's a very passive-aggressive way of communicating."  
> — Chris Richardson (sui limiti della choreography)

> "A Saga is not a long-running business process that lasts hours or days. It should complete in tens of milliseconds. If you're using a BPM engine to describe your Sagas, you're probably doing it wrong."  
> — Chris Richardson

> "The views in CQRS are disposable. They're replicas. If you need to change the schema, just throw them away and rebuild from scratch."  
> — Chris Richardson

---

## Riferimenti e risorse

- **[Microservices Patterns](https://www.manning.com/books/microservices-patterns)** — libro di Chris Richardson (Manning), con il 40% di sconto al link `go.manning.com/goto2019-microservices`
- **[microservices.io](https://microservices.io)** — sito di Chris Richardson con pattern language, articoli, codice di esempio
- **[Eventuate](https://eventuate.io)** — framework open source di Chris Richardson per Saga orchestration e Transactional Outbox
- **Saga Pattern** — [paper originale del 1987](https://www.cs.cornell.edu/andru/cs711/2002fa/reading/sagas.pdf) (Hector Garcia-Molina, Kenneth Salem)
- **Esempio di codice**: implementazioni di Saga (choreography + orchestration + event sourcing) su GitHub, linkate nelle slide del talk