---
tags:
  - messaging
  - reliability
  - nats
  - pub-sub
  - resilienza
feature:
type: article
author: Dear Architects
source: https://incident.io/blog/we-turned-off-pub-sub-and-nobody-noticed
date: 2026-09-20
---

# We Turned Off Pub/Sub and Nobody Noticed

## Sunto

incident.io gestisce circa 240 milioni di messaggi al giorno su circa 800 topic con oltre 1.000 subscription. La dipendenza esclusiva da Google Cloud Pub/Sub rappresentava un single point of failure inaccettabile: ogni evento della piattaforma transitava attraverso un singolo broker, gestito da un singolo provider, senza alcuna possibilità di routing alternativo. L'obiettivo era raggiungere un SLA del 99.99% eliminando questa vulnerabilità architettonica.

Il team aveva un vantaggio critico: un layer di astrazione esistente chiamato **eventadapter** che esponeva interfacce standard per publish e subscribe. Questo permetteva di sostituire l'implementazione senza modificare migliaia di call site nel codebase. La scelta del secondo broker è ricaduta su **NATS**: adottato da CNCF, nativo su Kubernetes, architettura a singolo binario e scritto in Go (compatibile con lo stack esistente).

La strategia di load balancing adottata è **active-active** (non standby passivo): i messaggi vengono distribuiti 50/50 usando FNV hashing sugli ID dei messaggi. Il failover avviene immediatamente tramite circuit breaker quando un tentativo di publish fallisce. La distribuzione è configurabile senza riavvii, permettendo aggiustamenti operativi dinamici.

Il lato subscription implementa un algoritmo di scheduling **"oldest message first"** ispirato alla teoria delle code MaxWeight: il broker con i messaggi più vecchi in attesa riceve priorità nel consumo. Questo previene la moltiplicazione delle risorse (un unico budget di concorrenza MaxHandlers distribuito tra entrambi i broker) e aggiusta dinamicamente i ratio di consumo in base alla distribuzione reale del carico. I wrapper per-broker implementano metodi Peek/Receive con semafori pesati che gestiscono i pool di goroutine handler.

La validazione è avvenuta attraverso chaos testing in produzione: il team ha deliberatamente eliminato il cluster NATS in produzione e simulato fallimenti di Pub/Sub tramite fault injection, confermando zero impatto sui clienti. Il risultato pratico: i timeout di rete verso Pub/Sub ora fanno graceful failover su NATS senza impatto visibile, la manutenzione di ciascun provider è operativamente sicura, e il pattern stabilisce una primitiva di reliability riutilizzabile per la piattaforma.

## Codice

Pattern di abstraction layer per il dual-broker setup:

```go
// eventadapter espone interfacce standard indipendenti dall'implementazione
type Publisher interface {
    Publish(ctx context.Context, topic string, msg Message) error
}

type Subscriber interface {
    Subscribe(topic string, handler HandlerFunc) error
}

// DualBrokerPublisher distribuisce i messaggi tra i due broker
type DualBrokerPublisher struct {
    primary   Publisher  // Google Cloud Pub/Sub
    secondary Publisher  // NATS
}

func (d *DualBrokerPublisher) Publish(ctx context.Context, topic string, msg Message) error {
    // FNV hashing per determinare il broker target (50/50 split)
    h := fnv.New32a()
    h.Write([]byte(msg.ID))
    if h.Sum32()%2 == 0 {
        if err := d.primary.Publish(ctx, topic, msg); err != nil {
            // Circuit breaker: failover immediato al broker secondario
            return d.secondary.Publish(ctx, topic, msg)
        }
        return nil
    }
    return d.secondary.Publish(ctx, topic, msg)
}
```

Algoritmo di scheduling fair-weighted per il consumer:

```go
// Inbox wrapper per ogni broker con Peek/Receive
type BrokerInbox struct {
    broker    Subscriber
    semaphore *WeightedSemaphore
}

// MaxWeight queuing: priorità al broker con messaggi più vecchi
func (s *FairScheduler) NextMessage() Message {
    pubsubOldest := s.pubsubInbox.Peek()
    natsOldest := s.natsInbox.Peek()

    if pubsubOldest.Timestamp.Before(natsOldest.Timestamp) {
        return s.pubsubInbox.Receive()
    }
    return s.natsInbox.Receive()
}
```
