---
tags:
  - user-tracking
  - scalable-architecture
  - grpc
  - pubsub
  - data-quality
  - software-architecture
feature:
type: article
author: Alina Krasavina
source: https://www.infoq.com/presentations/mobile-user-tracking-service/
date: 2026-08-01
---

# Challenging Google Analytics: Building a Scalable, Cost-Effective User Tracking Service

## Sunto

Delivery Hero ha deprecato Google Analytics (Universal Analytics) e costruito internamente una piattaforma di tracking utenti per rispondere a molteplici esigenze convergenti: la dismissione forzata di Universal Analytics, la necessità di dati in tempo reale (GA forniva dati solo una o due volte al giorno), la conformità GDPR che richiedeva il controllo totale sull'infrastruttura di raccolta dati, e l'obiettivo di non pagare più di quanto si pagava per GA. Il risultato finale è un sistema attualmente 3x più economico di GA, capace di gestire 10x il carico originale con una completezza dei dati del 97%.

L'architettura MVP iniziale era volutamente semplice: SDK mobile e TypeScript frontend per la raccolta dati, una semplice API con due processori che leggono da Pub/Sub, un processore di fallback per la gestione degli errori, e streaming verso BigQuery tramite Google Cloud. Questa semplicità intenzionale ha permesso iterazioni rapide e ha gestito il carico iniziale senza incidenti di perdita dati. La metrica di partenza era l'85% di match rate degli ordini con GA come baseline.

L'architettura si è evoluta aggiungendo gRPC con percorsi di scrittura duali (processori + BigQuery), backoff esponenziale per i retry, e prioritizzazione degli eventi in base al valore aziendale (i dati fatturabili hanno la priorità più alta). La sfida principale della perdita di dati durante i riavvii dei pod è stata risolta rendendo l'elaborazione delle richieste sincrona: "Se il pod muore, risponde semplicemente con 500. Il client rimanda i dati." Questo ha aumentato la latenza di 7x ma ha eliminato la perdita di dati.

Un contributo architetturale particolarmente significativo è la governance tramite code generation: i modelli di eventi vengono generati dal codice a partire da schemi centralizzati, abilitando la validazione a compile-time per gli sviluppatori e prevenendo inconsistenze nei valori null (spazi, stringhe vuote, stringa "null", zeri). Questo ha trasformato la cultura della qualità dei dati da reattiva a proattiva: gli errori vengono catturati durante lo sviluppo, non dopo la produzione.

La strategia di rollout progressivo — deployment degli SDK e dei cambiamenti backend gradualmente su brand più piccoli prima — ha ridotto il blast radius in caso di incidenti. Le lezioni più importanti riguardano l'osservabilità: i gap identificati post-lancio includevano logging insufficiente per il debug delle perdite di dati e alerting inadeguato nei punti critici del sistema. Il testing del caos (dopo un'interruzione di GCP) è diventato parte standard del processo di validazione.

## Codice

Schema della strategia di gestione degli errori nel processing sincrono — il pod risponde 500 invece di perdere dati silenziosamente:

```
Request → API → Pub/Sub Publisher
                    ↓ (sync: wait for ack)
              Pub/Sub ACK → Response 200
                    ↓ (on error)
              Response 500 → Client retry (exponential backoff)
```

Struttura della prioritizzazione degli eventi per SDK mobile:

```
Priority Queue:
  1. Billable events (orders, payments)
  2. Business-critical events
  3. Standard behavioral events
  4. Low-priority telemetry

Backoff: exponential (base 2s, max 60s)
Batch size: configurable per event type
Queue overflow: drop lowest-priority events
```
