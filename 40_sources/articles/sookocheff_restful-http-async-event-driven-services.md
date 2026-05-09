---
tags:
  - api
  - architecture
  - microservices
  - event-driven
  - distributed-systems
type: article
author: Kevin Sookocheff
source: https://sookocheff.com/post/api/marrying-restful-http-with-asynchronous-design/
date: 2020-11-24
---

# Marrying RESTful HTTP with Asynchronous and Event-Driven Services

## Sunto

L'articolo affronta un problema classico nell'architettura a microservizi: come integrare servizi indipendenti senza sacrificarne l'autonomia. Il punto di partenza è che **non esiste uno stile di comunicazione universale** — la scelta tra sincrono e asincrono, tra singolo e multiplo destinatario, dipende dal contesto. Più che lo stile, ciò che conta davvero è preservare l'indipendenza dei servizi.

Il principio cardine è: **minimizzare la comunicazione tra microservizi**, e in particolare evitare le *catene di chiamate sincrone*. L'anti-pattern delle "luci dell'albero di Natale" (AKF Partners) illustra il rischio: se un bulbo si rompe, tutta la catena si spegne. Le chiamate sincrone in serie creano failure cascading e accoppiamento temporale stretto. La comunicazione **asincrona** è la risposta pragmatica a questi problemi: i sistemi non devono essere disponibili contemporaneamente, si ottiene alta coesione e basso accoppiamento, e si abilita la composizione parallela dei servizi.

La proposta architetturale centrale è usare **HTTP/REST ai confini del sistema** (tramite API Gateway o *Backend for Frontend*) e comunicazione asincrona all'interno. I servizi interni possono trattare le proprie API come "classi private", mentre le API esterne devono essere stabili e documentate. Importante anche resistere alla tentazione di applicare il principio DRY ai modelli condivisi tra sistemi: avere rappresentazioni multiple della stessa entità (es. Customer in sistemi diversi) è desiderabile, non un problema. I punti di sovrapposizione sono i naturali *integration point*, non difetti da eliminare.

L'articolo distingue tra **code** ed **event stream**. Le code appartengono al servizio consumatore (non sono bus globali) e servono per buffer di picchi di carico, disaccoppiamento tra task submission e processing, e *queue-based load leveling*. Gli event stream sono log durabili ordinati (a differenza del publish-subscribe, che non permette il replay) da cui i consumer leggono da qualsiasi posizione. Lo streaming è ideale quando più consumer processano gli stessi eventi, si richiede bassa latenza, serve complex event processing o event sourcing.

L'autore descrive una **progressione incrementale** in 5 stadi — da singolo servizio REST a piattaforma composable — che mostra come aggiungere gradualmente code, BFF e stream senza rompere i contratti esterni. I principi architetturali risultanti della piattaforma composable sono: nessun single point of failure (code/topic localizzate per servizio, non bus globale), fault isolation tramite bounded context, design asincrono, frontend stateless, scale-out orizzontale, e API pensate come esterne fin dal principio.

---

## Classificazione dei sistemi di comunicazione

I servizi si classificano lungo due assi:

| | Singolo destinatario | Multipli destinatari |
|---|---|---|
| **Sincrono** | REST, RPC | — |
| **Asincrono** | Code (queue) | Message bus, Event stream |

---

## Progressione incrementale verso la piattaforma composable

| Stadio | Struttura | Comunicazione |
|---|---|---|
| 1 — Singolo servizio | Un servizio REST, opzionalmente un event stream in uscita | REST esterno |
| 2 — Microservizi | API REST esterna come facade, servizi interni con async/RPC | REST esterno + async interno |
| 3 — Consumer multipli | BFF (Browser, Mobile, Third-party) + sistema interno asincrono | BFF → REST → sistema |
| 4 — Load balancing | Code ai confini consumer per load leveling; l'API REST diventa asincrona | REST asincrono (202/polling) |
| 5 — Piattaforma composable | Due sistemi integrati via REST ai confini + event stream per aggiornamenti | REST + streaming |

> "The only API to your system is your RESTful frontend and your event stream." — Kevin Sookocheff

---

## Code vs Event Stream: confronto

| Aspetto | Coda (Queue) | Event Stream |
|---|---|---|
| **Consumer** | Singolo (ogni messaggio processato da uno solo) | Multipli (tutti ricevono tutti gli eventi) |
| **Replay** | No | Sì (log durevole, lettura da qualsiasi posizione) |
| **Owner** | Il servizio consumatore (non bus globale) | Infrastruttura condivisa ma log immutabile |
| **Uso ideale** | Buffer picchi di carico, load leveling | CEP, event sourcing, più consumer su stessi eventi |
| **Publish-subscribe** | No | Variante: no replay, new subscriber perde eventi passati |

---

## Principi della piattaforma composable

- **No single point of failure** — code e topic sono localizzate per servizio, non bus globali condivisi
- **Fault isolation** — i bounded context limitano il blast radius dei guasti
- **Asynchronous by design** — inerente all'approccio
- **Frontend stateless** — scalabilità naturale dei BFF
- **Scale out** — scaling orizzontale tramite BFF e load leveling
- **Think external-first** — le API vanno progettate come se fossero pubbliche, usando standard aperti

---

## Link esterni

- [Microsoft: Communication in Microservice Architecture](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/architect-microservice-container-applications/communication-in-microservice-architecture) — overview dei pattern di comunicazione nei microservizi .NET
- [AKF Partners: Christmas Tree Lights Anti-Pattern](https://akfpartners.com/growth-blog/microservice-anti-pattern-calls-in-series-the-xmas-tree-light-anti-pattern) — anti-pattern delle chiamate sincrone in serie
- [Enterprise Integration Patterns](https://www.enterpriseintegrationpatterns.com/patterns/messaging/) — catalogo di pattern di messaggistica (Hohpe & Woolf)
- [APIs You Won't Hate](https://apisyouwonthate.com/) — risorsa su API design pragmatico
- [Why Messaging Queues Suck (ProgrammableWeb)](https://www.programmableweb.com/news/why-messaging-queues-suck/analysis/2017/02/13) — articolo citato come contrappunto critico alle code

---

## Immagini

Nessuna immagine presente
