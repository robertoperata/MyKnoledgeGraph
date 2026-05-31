---
tags:
  - tail-latency
  - hedged-requests
  - distributed-systems
  - performance
  - p99
feature:
type: article
author: Prathamesh Bhope
source: https://www.infoq.com/articles/adaptive-hedged-requests-p99-latency/
date: 2026-05-31
---

# Stragglers, Not Failures: How Adaptive Hedged Requests Reduce p99 Latency by 74 Percent

## Sunto

L'articolo affronta una distinzione fondamentale nel contesto dei sistemi distribuiti ad alte prestazioni: la differenza tra *straggler* (richieste lente ma completate) e *failure* (richieste che non completano). Retryare uno straggler è controproducente perché aggiunge ulteriore carico su un backend già sotto pressione; la strategia corretta è l'*hedging* — inviare una richiesta di backup in parallelo e usare la risposta più veloce, annullando l'altra. L'articolo descrive come Prathamesh Bhope di Walmart ha implementato un meccanismo di hedging adattivo che riduce la p99 del 73% mantenendo un overhead accettabile.

Il problema delle **fan-out amplification** è il punto di partenza: con 100 servizi downstream ciascuno con un tasso di straggler dell'1%, circa il 63% delle richieste di livello superiore subisce ritardi. Questo effetto è invisibile nei singoli dashboard per-servizio, che mostrano metriche sane. Il hedging adattivo risolve il problema calibrandosi in tempo reale sulle distribuzioni di latenza osservate, anziché usare soglie statiche.

Il cuore del meccanismo è **DDSketch**, un algoritmo per la stima dei quantili con garanzie di errore relativo (±1%) e complessità O(1) in memoria, con un overhead di circa 35 nanosecondi per richiesta. Le finestre di osservazione vengono ruotate ogni 30 secondi per adattarsi ai cambiamenti di latenza. Ogni host mantiene la propria stima del p90 locale, e una richiesta viene hedgiata se il primario supera quella soglia.

Per proteggere il backend da cascate di carico durante rallentamenti generalizzati, il sistema usa un **token bucket** che cap il tasso di hedge a una percentuale configurabile (default 10%). Quando il bucket si esaurisce (in circa un secondo con rallentamento sostenuto), l'hedging si ferma automaticamente, evitando di amplificare i problemi invece di risolverli. Il response del primario annulla automaticamente la richiesta di backup se arriva per primo, con connection pooling safety garantita dal draining in background del body.

I benchmark su 50.000 richieste con latenza lognormale (media 5ms, stddev 2ms) e 5% di probabilità di straggler mostrano: senza hedging p99 = 65.0 ms, con hedging adattivo p99 = 17.3 ms (riduzione del 73%) a fronte di un overhead dell'8.9%. Per gli endpoint di streaming LLM la metrica corretta è il TTFT (time-to-first-token), non il TTFB, ottenendo un miglioramento comparabile con overhead del 17-19.8%. L'hedging non è applicabile a operazioni non-idempotenti (scritture, pagamenti) né a servizi con rate limit condivisi come le API LLM.

## Immagini

![SDLC con hedging adattivo](https://www.infoq.com/articles/adaptive-hedged-requests-p99-latency/)

## Codice

Configurazione del client Go con hedge adattivo drop-in come RoundTripper HTTP:

```go
// Drop-in HTTP RoundTripper con hedging adattivo — zero configurazione
client := &http.Client{
    Transport: hedge.NewAdaptiveTransport(http.DefaultTransport),
}

// Oppure con opzioni esplicite
client := &http.Client{
    Transport: hedge.NewAdaptiveTransport(
        http.DefaultTransport,
        hedge.WithBudget(0.10),          // max 10% di hedge rate
        hedge.WithWindow(30*time.Second), // finestra di osservazione DDSketch
        hedge.WithPercentile(0.90),       // soglia p90 per il trigger
    ),
}
```

Token bucket per protezione del budget di hedging:

```go
// Logica del token bucket — si esaurisce in ~1s con rallentamento sostenuto al 10%
// bucket capacity = 100 token, rate = budget * requests/s
if !hedgeBudget.Allow() {
    // budget esaurito: esegui solo la richiesta primaria
    return primary.Do(req)
}
// invia richiesta di hedge in parallelo
go hedge.Do(req.Clone(ctx))
```
