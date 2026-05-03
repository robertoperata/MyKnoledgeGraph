---
title: DDD per la decomposizione di un monolite in microservizi
type: synthesis
tags:
  - thread1-microservices
  - thread5-ddd
sources:
  - "[[bounded-context]]"
  - "[[aggregate]]"
  - "[[domain-event]]"
  - "[[saga-pattern]]"
  - "[[cqrs-read-model]]"
  - "[[legacy-modernization]]"
  - "[[ubiquitous-language]]"
updated: 2026-04-18
related:
  - "[[concepts/bounded-context]]"
  - "[[concepts/aggregate]]"
  - "[[concepts/domain-event]]"
  - "[[patterns/saga-pattern]]"
  - "[[patterns/cqrs-read-model]]"
  - "[[concepts/legacy-modernization]]"
---

# Domanda originale

**Come si usa il DDD per decomporre un monolite in microservizi: dalla scoperta dei Bounded Context alla scelta del pattern di comunicazione (Saga, CQRS, Event-Driven)?**


---

# Risposta sintetica

Il DDD fornisce i criteri sia per *dove tagliare* il monolite sia per *come far comunicare* i pezzi risultanti. I due livelli sono distinti ma dipendenti: scegliere male le boundary rende sbagliato qualsiasi pattern di comunicazione successivo.

---

## Fase 1 — Trovare i confini: Strategic Modeling

### L'Ubiquitous Language come rivelatore di confini

Il primo segnale di un confine sbagliato è l'ambiguità linguistica: se il termine "Customer" significa cose diverse nel contesto degli ordini e in quello della fatturazione, ci sono due Bounded Context distinti. L'**Ubiquitous Language** non è solo convenzione: è lo strumento diagnostico principale.

Tecnica pratica: **Event Storming** — mappare gli eventi di dominio su una timeline rivela naturalmente dove il linguaggio cambia e dove le responsabilità si separano.

### Bounded Context: la regola dei microservizi

> Un microservizio non dovrebbe mai attraversare il confine di due Bounded Context diversi.

Un BC può diventare uno o più microservizi, mai il contrario. La violazione più comune: creare servizi per layer tecnici (un servizio "database", un servizio "API") invece che per dominio di business.

La struttura del BC tende a essere più stabile della struttura tecnica perché riflette il business — che cambia più lentamente del codice. Questo rende le boundary DDD più durature delle boundary tecniche.

### Aggregate: l'unità di transazionalità

All'interno di ogni BC, gli **Aggregate** determinano le unità di consistenza. La regola di Vaughn Vernon:

> Se due concetti devono essere sempre consistenti insieme, appartengono allo stesso aggregate (e probabilmente allo stesso servizio).

Gli aggregate piccoli sono preferibili: aggregate grandi portano a lock di lunga durata e problemi di performance. Il principio "riferirsi ad altri aggregate solo per ID" è già la preparazione alla comunicazione distribuita.

---

## Fase 2 — Come estrarre incrementalmente: Strangler Fig

Il Big Bang Rewrite è quasi sempre sbagliato: nessun valore consegnato fino al termine, decisioni non validate fino al go-live.

Il pattern **Strangler Fig** ("scolpire pezzi dal monolite"):
1. Identificare un BC che può diventare servizio indipendente
2. Disintricarlo — può richiedere mesi, ma ogni passo è validabile in produzione
3. Gestire la **duplicazione transitoria dei dati**: le colonne rimangono nel DB originale come repliche read-only mentre il nuovo servizio diventa owner. Situazione temporanea, con eventual consistency accettata per quel periodo.

---

## Fase 3 — Scegliere il pattern di comunicazione

Una volta estratti i servizi, la scelta del pattern dipende da due variabili: **direzione dell'accoppiamento** e **requisiti di consistenza**.

### Quando usare Event-Driven + Domain Events

Quando il publisher non deve sapere chi reagisce ai suoi eventi. I Domain Event ("OrderPlaced", "PaymentReceived") comunicano tra BC senza accoppiamento diretto: l'intento della collaborazione è nel destinatario, non nel mittente.

**Caso tipico:** un BC pubblica eventi, altri BC costruiscono proiezioni locali (→ CQRS).

### Quando usare CQRS + Read Model

Quando un servizio ha bisogno di dati di un altro dominio per letture frequenti. Il Projector ascolta gli eventi del servizio owner e mantiene aggiornata una copia locale denormalizzata. Latenza runtime = zero.

**Trade-off:** eventual consistency. Accettabile per pricing, raccomandazioni, fraud detection. Non accettabile per saldo bancario o stock in tempo reale.

### Quando usare Saga

Quando una transazione di business attraversa più servizi e richiede rollback in caso di fallimento parziale.

- **Orchestration:** preferibile quando il processo è complesso e deve essere comprensibile in un punto solo. Rischio: orchestratore che accumula logica di dominio.
- **Choreography:** preferibile quando i servizi sono molti e il disaccoppiamento è prioritario. Rischio: il business process non è esplicito da nessuna parte.

**Attenzione:** le saga hanno bassa scalabilità. Prima di usarle, verificare se il problema si risolve con un singolo aggregate (forte consistenza locale) o con eventual consistency senza compensazione.

---

## La connessione trasversale

```
Ubiquitous Language    →  dove sono i confini (strategic)
Bounded Context        →  confini stabili per i microservizi
Aggregate              →  unità transazionale → quale servizio possiede cosa
Domain Event           →  mezzo di comunicazione tra BC
Event-Driven           →  disaccoppiamento tra publisher e subscriber
CQRS + Read Model      →  letture efficienti di dati cross-domain
Saga                   →  transazioni distribuite con compensazione
```

---

## Sorgenti utilizzate

- [[concepts/bounded-context]] — criteri per le boundary dei microservizi
- [[concepts/ubiquitous-language]] — linguaggio come strumento diagnostico
- [[concepts/aggregate]] — unità di consistenza e implicazioni per il design
- [[concepts/domain-event]] — comunicazione tra BC
- [[patterns/saga-pattern]] — transazioni distribuite
- [[patterns/cqrs-read-model]] — proiezioni locali cross-domain
- [[concepts/legacy-modernization]] — estrazione incrementale dal monolite

---

## Lacune / Argomenti da approfondire

- Context Mapping patterns (Shared Kernel, Customer-Supplier, Conformist, Anti-Corruption Layer, Open Host Service)
- Event Storming: metodologia completa per la discovery dei Bounded Context
- Subdomain types: Core Domain, Supporting Domain, Generic Subdomain
- Domain Services vs Application Services vs Infrastructure Services
- Aggregate design avanzato: invarianti complesse, lazy loading, snapshot pattern
- Outbox Pattern per garantire la consistenza tra scrittura DB e pubblicazione evento
- Testing del domain model: unit test per gli aggregate, integration test per i domain events
- Versioning degli eventi di dominio (schema evolution, Tolerant Reader Pattern)
