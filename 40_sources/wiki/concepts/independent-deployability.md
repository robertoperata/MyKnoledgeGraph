---
title: Independent Deployability
type: concept
tags: [thread1-microservices]
sources:
  - "[[Microservices Fundamental]]"
  - "[[Microservices Collaboration]]"
updated: 2026-04-09
related:
  - "[[concepts/information-hiding]]"
  - "[[concepts/bounded-context]]"
  - "[[patterns/event-driven]]"
---

# Independent Deployability

## Definizione

La capacità di rilasciare una singola unità di software in produzione senza richiedere il deployment coordinato di altri servizi. È la proprietà definitoria dei microservizi: senza independent deployability non si tratta di microservizi, ma di un distributed monolith.

## Come funziona

L'independent deployability si ottiene attraverso:

1. **Backwards compatibility**: ogni modifica pubblica a un servizio non deve rompere i consumer esistenti. È la responsabilità primaria di chi sviluppa il servizio.
2. **Information hiding**: esporre il minimo indispensabile dell'interfaccia pubblica. Tutto ciò che non è esposto può cambiare liberamente.
3. **Database per servizio**: nessun database condiviso tra servizi. La condivisione del database crea accoppiamento implicito che impedisce i deploy indipendenti.
4. **Schema esplicito**: ogni endpoint ha un contratto formale. L'implicit schema è un rischio perché non rilevabile automaticamente.

## Quando è minacciata

- **Shared database**: due servizi accedono alla stessa tabella → un DDL change richiede coordinamento
- **Shared libraries incompatibili**: se una libreria deve essere aggiornata su tutti i servizi contemporaneamente → coupling
- **Distributed monolith**: deployment di un servizio richiede il deployment di altri → si perde il beneficio principale

> [!quote] "A lot of the challenges of microservices are the challenges of any distributed system dealing with the network." — Sam Newman

## Esempi pratici dalle sorgenti

**Microservices Fundamental (Sam Newman):**
- La soluzione al coupling da shared library è: se il codice deve cambiare su tutti i servizi contemporaneamente, crea un nuovo servizio invece di una libreria condivisa.
- Duplicazione del codice tra servizi è accettabile se evita il coupling delle librerie.

**Microservices Collaboration (Sam Newman):**
- L'information hiding ("hide information/behavior that changes behind a stable boundary") è il meccanismo principale per garantire backwards compatibility.
- "You always have a schema — but is it explicit or implicit?" — rendere lo schema esplicito tramite OpenAPI permette di verificare la compatibilità automaticamente (openapi-diff).

## Connessioni

- [[concepts/information-hiding]] — meccanismo principale per garantire l'independent deployability
- [[concepts/bounded-context]] — le boundary del bounded context definiscono dove creare il confine del servizio
- [[patterns/event-driven]] — la comunicazione event-driven è naturalmente più decoupled rispetto al request/response sincrono
- [[concepts/twelve-factor]] — il principio 5 (Build/Release/Run) e il principio 4 (Stateless Processes) supportano la deployability
