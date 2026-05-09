---
title: Information Hiding
type: concept
tags: [thread1-microservices]
sources:
  - "[[Microservices Fundamental]]"
  - "[[Microservices Collaboration]]"
updated: 2026-04-09
related:
  - "[[concepts/independent-deployability]]"
  - "[[concepts/bounded-context]]"
---

# Information Hiding

## Definizione

Principio introdotto da Parnas nel 1971: *nascondere le cose più soggette a cambiamento dietro un confine stabile*. Applicato ai microservizi: tutto è nascosto per default, a meno che non venga esplicitamente esposto.

## Come funziona

- L'interfaccia pubblica di un servizio è il suo contratto con i consumer.
- Tutto ciò che non è nell'interfaccia pubblica può cambiare liberamente senza impattare altri servizi.
- La regola operativa: **non esporre nulla finché non è necessario** ("Don't expose anything until you need it. Be explicit.").

> [!tip] "The more you hide, the easier it is to maintain backwards compatibility." — Sam Newman

## Consumer-first mindset

Il design dell'interfaccia pubblica deve essere guidato da ciò che i consumer hanno bisogno, non da come il servizio è implementato internamente. Questo è il principio "outside-in": pensa al servizio dal punto di vista di chi lo chiama.

Conseguenza pratica: tratta gli endpoint del microservizio come una UI — sono la faccia che mostri al mondo, non un riflesso della tua struttura interna.

## Schema esplicito

Una conseguenza diretta dell'information hiding è avere **schema espliciti** (OpenAPI, Avro, Protobuf). Lo schema implicito esiste sempre, ma non è rilevabile automaticamente. Rendere lo schema esplicito permette di:
- Verificare la backwards compatibility con tool (es. openapi-diff)
- Documentare il contratto formalmente
- Generare client stub automaticamente

## Connessioni

- [[concepts/independent-deployability]] — l'information hiding è il meccanismo che rende possibile l'independent deployability
- [[concepts/bounded-context]] — la boundary del bounded context è il confine dello hiding
- [[concepts/architectural-governance]] — l'architettura dichiarativa (InfoQ) segue lo stesso principio: le decisioni che cambiano spesso sono distillate in dichiarazioni esplicite e stabili
- [[patterns/modular-monolith]] — la distinzione tra API pubblica e implementazione privata del modulo è information hiding applicato a livello intra-applicazione; dipendenze solo sulle API, mai sulle implementazioni
