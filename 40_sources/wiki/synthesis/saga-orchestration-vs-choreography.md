---
title: Saga — Orchestration vs Choreography, criteri di scelta in sistemi DDD reali
type: synthesis
tags:
  - thread1-microservices
  - thread5-ddd
sources:
  - "[[saga-pattern]]"
  - "[[domain-event]]"
  - "[[event-driven]]"
  - "[[bounded-context]]"
  - "[[aggregate]]"
  - "[[async-workflow-patterns]]"
updated: 2026-04-18
related:
  - "[[patterns/saga-pattern]]"
  - "[[concepts/domain-event]]"
  - "[[patterns/event-driven]]"
  - "[[concepts/bounded-context]]"
  - "[[concepts/aggregate]]"
  - "[[concepts/async-workflow-patterns]]"
---

# Domanda originale

**Orchestration vs Choreography nella Saga: criteri di scelta in sistemi DDD reali, con esempi su quando ciascuna rompe?**

---

# Risposta sintetica

La scelta tra orchestration e choreography non è tecnica: è un trade-off tra **esplicitezza** e **disaccoppiamento**. L'orchestration rende il business process visibile ma concentra la responsabilità. La choreography distribuisce la responsabilità ma rende il business process implicito. In sistemi DDD, la scelta è guidata dalla struttura dei Bounded Context coinvolti e dalla complessità del processo.

---

## Prima domanda: serve davvero una Saga?

Prima di scegliere tra orchestration e choreography, verificare se la Saga è necessaria.

Le saga hanno **bassa scalabilità**: compensating transactions costose, coordination overhead elevato.

**Alternativa 1 — Aggregate singolo:** se i dati coinvolti appartengono allo stesso Bounded Context, si può modellare come un aggregate più grande. Forte consistenza locale, zero overhead distribuito. Preferire sempre questa opzione se possibile.

**Alternativa 2 — Eventual consistency senza compensazione:** se non è richiesto il rollback in caso di fallimento parziale, l'event-driven semplice è sufficiente. Il servizio pubblica eventi, gli altri reagiscono in modo autonomo.

**Usa la Saga solo quando:** la transazione attraversa più BC, richiede rollback coordinato su failure parziali, e nessuna delle due alternative sopra è applicabile.

---

## Orchestration: quando usarla e quando rompe

### Struttura

Un orchestratore centrale conosce tutti i passi della saga e dirige ogni servizio:

```
Orchestratore
    → OrderService.reserve()    → OK
    → PaymentService.charge()   → OK
    → ShippingService.ship()    → FAIL
    → PaymentService.refund()   (compensating)
    → OrderService.cancel()     (compensating)
```

### Quando sceglierla

- Il processo è **complesso** (molti passi, condizioni, branching)
- Il business process deve essere **comprensibile** in un punto solo (audit, debugging, compliance)
- Il team è piccolo o il processo è critico e non si può permettere comportamenti emergenti
- Il dominio è nuovo e il processo è ancora in evoluzione (più facile da modificare in un posto)

### Quando rompe

**1. L'orchestratore diventa un "god service"**
Se l'orchestratore accumula logica di dominio invece di solo coordinare, diventa un monolite distribuito. Segnale di allarme: l'orchestratore chiama direttamente i repository o contiene regole di business.

**2. Accoppiamento eccessivo**
L'orchestratore conosce tutti i servizi → qualsiasi cambio di interfaccia dei servizi partecipanti rompe l'orchestratore. In organizzazioni grandi con molti team, questo diventa un collo di bottiglia.

**3. Scalabilità dell'orchestratore**
L'orchestratore è un single point of failure e di scalabilità. Se diventa il bottleneck, tutta la saga si blocca.

---

## Choreography: quando usarla e quando rompe

### Struttura

Ogni servizio pubblica eventi e reagisce agli eventi degli altri, senza coordinamento centrale:

```
OrderService
    ──OrderCreated──► PaymentService
                          ──PaymentDone──► ShippingService
                                               ──ShipFailed──► PaymentService
                                                                    ──PaymentRefunded──► OrderService
```

### Quando sceglierla

- Il sistema ha **molti servizi** con team indipendenti (alta autonomia organizzativa)
- Il disaccoppiamento è prioritario rispetto all'esplicitezza del processo
- Il processo non ha molti step condizionali o branching complesso
- I servizi partecipanti devono poter evolvere indipendentemente

### Quando rompe

**1. Il business process diventa invisibile**
Nessuno sa dove guardare per capire lo stato di una transazione. Debugging di un failure richiede di tracciare manualmente la catena di eventi attraverso più log.

**2. Cicli impliciti**
Un evento di compensazione può scatenare altri eventi che tornano ad attivare il primo servizio. I cicli nella choreography sono difficili da rilevare a design time.

**3. Testing**
Testare un'intera choreography richiede di simulare la catena completa di eventi. I test di integrazione sono complessi e fragili.

**4. Cambio di requisiti**
Aggiungere un nuovo step a una choreography richiede di modificare potenzialmente tutti i servizi partecipanti, non un solo orchestratore.

---

## Criteri di scelta in sistemi DDD reali

| Criterio | Orchestration | Choreography |
|---|---|---|
| Numero di BC coinvolti | Fino a 3-4 | Molti |
| Complessità del processo | Alta (branching, condizioni) | Bassa/media |
| Requisiti di audit/compliance | ✅ (processo esplicito) | ❌ (difficile tracciare) |
| Autonomia dei team | Bassa | Alta |
| Frequenza di modifica del processo | Alta | Bassa |
| Tolleranza al debugging complesso | Bassa | Alta |

### Regola empirica

> Se un nuovo developer non riesce a capire il flusso del business process leggendo un singolo file o componente, la choreography è probabilmente troppo complessa per quel caso.

---

## Connessione con i Durable Execution Engine

I pattern di **Async Workflow** (Tier 1-3) si sovrappongono con la Saga:
- I durable execution engine (Temporal, Inngest) sono essenzialmente **orchestratori** con stato persistente, retry automatici e checkpoint
- Quando il numero di passi e le condizioni di retry/timeout diventano complessi, un durable execution engine è più affidabile di un orchestratore custom

**Debounce e Throttle** nella saga: se gli step di compensazione possono innescarsi molte volte (es. sistema di retry aggressivo), il Debounce (coalescenza di compensazioni ravvicinate) e il Throttle (rate limiting dei tentativi) si applicano anche alla Saga.

---

## Sorgenti utilizzate

- [[patterns/saga-pattern]] — struttura, orchestration vs choreography, trade-off
- [[concepts/domain-event]] — gli eventi come mezzo della choreography
- [[patterns/event-driven]] — substrate della choreography
- [[concepts/bounded-context]] — la struttura dei BC guida la scelta
- [[concepts/aggregate]] — alternativa alla Saga quando i dati sono locali
- [[concepts/async-workflow-patterns]] — durable execution engine come alternativa all'orchestratore custom

---

## Lacune / Argomenti da approfondire

- Outbox Pattern: garantire che l'evento venga pubblicato atomicamente con la scrittura DB
- Durable execution engine: Temporal, Conductor, Inngest — confronto e casi d'uso
- Saga state machine: come modellare e persistere lo stato di una saga orchestrata
- Compensating transaction design: idempotenza, retry safety, timeout
- Testing di saga distribuite: contract testing tra servizi partecipanti
- Observability delle saga: distributed tracing attraverso i passi della saga (correlation ID)
- Process Manager pattern: evoluzione dell'orchestratore che non accumula logica di dominio
- Dead letter queue nella choreography: gestione dei messaggi non processabili
