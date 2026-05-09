---
title: Sync/Async Hybrid Architecture
type: concept
tags: [thread1-microservices]
sources:
  - "[[sookocheff_restful-http-async-event-driven-services]]"
updated: 2026-05-08
related:
  - "[[patterns/event-driven]]"
  - "[[patterns/api-composition-bff]]"
  - "[[concepts/independent-deployability]]"
  - "[[concepts/bounded-context]]"
  - "[[concepts/async-workflow-patterns]]"
---

# Sync/Async Hybrid Architecture

## Definizione

Approccio architetturale che combina **HTTP/REST ai confini del sistema** (verso client e sistemi esterni) con **comunicazione asincrona all'interno** (tra microservizi). Non sceglie uno stile di comunicazione unico — riconosce che sincrono e asincrono hanno ruoli diversi.

> "It is naïve to think that we can only choose one communication style to solve all problems." — Kevin Sookocheff

## Anti-pattern: catene di chiamate sincrone

Il rischio principale nella comunicazione inter-servizio è creare **catene sincrone**, dove un servizio chiama un altro che ne chiama un altro ancora. L'AKF Partners chiama questo il **Christmas Tree Lights Anti-Pattern**: se un bulbo si rompe, tutta la catena si spegne.

Conseguenze:
- **Failure cascading**: il guasto di un servizio si propaga a tutti i chiamanti
- **Temporal coupling**: tutti i servizi della catena devono essere up contemporaneamente
- **Latenza additiva**: il tempo totale è la somma delle latenze, non il massimo

**Regola fondamentale:** evitare il chaining di chiamate sincrone tra microservizi.

## Il principio: REST ai confini, async internamente

```
[Client / Sistemi esterni]
         │
         ▼  REST / HTTP (stabile, documentato)
[API Gateway / BFF]
         │
         ▼  Async (message broker, RPC, event stream)
[Microservizi interni]
```

- Le **API esterne** (verso client, third-party, altri sistemi) devono essere stabili, ben documentate, progettate come se fossero pubbliche.
- Le **API interne** tra microservizi possono essere trattate come "private" — cambiano più liberamente.

> "The only API to your system is your RESTful frontend and your event stream." — Kevin Sookocheff

## Queue ownership: la coda appartiene al consumer

Le code non devono essere bus globali condivisi. Ogni coda **appartiene al servizio consumatore** che la utilizza per:
- Bufferare picchi di carico intermittenti
- Disaccoppiare la submission del task dalla sua esecuzione
- Permettere al servizio di processare al proprio ritmo

Questo è il **Queue-Based Load Leveling**: il consumer controlla il proprio backpressure senza imporre pressione al producer.

Benefici del queue ownership:
- Nessun single point of failure (la coda è scoped al servizio)
- Il producer non conosce né controlla la velocità di processing del consumer
- Scalabilità indipendente per ogni servizio

## Resist DRY inter-sistema: le entità condivise sono integration point

Un principio controintuitivo ma importante: **non applicare DRY alle entità condivise tra sistemi diversi**. Se Sistema A e Sistema B hanno entrambi un concetto di `Customer`, è corretto che abbiano rappresentazioni separate.

I punti di sovrapposizione sono **integration point** — vanno trattati come tali (es. con [[patterns/anti-corruption-layer]]), non eliminati attraverso una struttura condivisa che creerebbe coupling tra sistemi.

Questo rinforza il principio di [[concepts/bounded-context]]: ogni contesto ha il proprio modello, anche se descrive la stessa realtà del mondo esterno.

## Queue vs Event Stream: distinzione

| Aspetto | Coda | Event Stream |
|---|---|---|
| Consumer | Singolo (ogni messaggio → un solo consumer) | Multipli (tutti ricevono tutti gli eventi) |
| Replay | No | Sì (log durevole, lettura da qualsiasi posizione) |
| Ownership | Consumer (locale al servizio) | Infrastruttura condivisa |
| Pub-sub | No | Sì (con o senza replay a seconda dell'impl.) |
| Uso ideale | Load leveling, task queue | CEP, event sourcing, fan-out |

## Progressione verso piattaforma composable

L'architettura si costruisce incrementalmente:
1. Singolo servizio REST + event stream opzionale
2. Microservizi con facade REST esterna + async interno
3. BFF multipli (browser, mobile, third-party)
4. Code ai confini consumer (load leveling) → l'API REST diventa asincrona
5. Sistemi separati integrati via REST + event stream

## Connessioni

- [[patterns/event-driven]] — la comunicazione asincrona interna è implementata tramite event-driven
- [[patterns/api-composition-bff]] — il BFF è il layer REST ai confini del sistema verso i client
- [[concepts/independent-deployability]] — l'approccio async evita il temporal coupling necessario per il deploy indipendente
- [[concepts/bounded-context]] — il "resist DRY inter-sistema" è un'applicazione del principio che ogni bounded context ha il proprio modello
- [[patterns/anti-corruption-layer]] — i punti di sovrapposizione tra sistemi sono integration point da gestire con ACL
- [[concepts/async-workflow-patterns]] — le code consumer si inseriscono nel modello di maturità dei workflow asincroni
