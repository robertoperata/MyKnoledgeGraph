---
tags:
  - load-balancing
  - kubernetes
  - scalability
  - performance
  - distributed-systems
feature:
type: article
author: Conor Gallagher
source: https://engineering.zalando.com/posts/2026/06/client-side-load-balancing.html
date: 2026-07-05
---

# Client-Side Load Balancing at a Million Requests Per Second

## Sunto

Il team di Zalando ha migrato la Product Read API (PRAPI) da un load balancer condiviso lato edge (Skipper) a un load balancer client-side in-process (CSLB), gestendo oltre un milione di richieste al secondo. La decisione è nata dalla natura fan-out dell'API: ogni batch request viene scomposta in fino a 100 chiamate parallele downstream, ciascuna delle quali transitava per Skipper. Questo significava che la latenza di un batch seguiva la singola chiamata più lenta tra le cento, amplificando in modo sistematico qualsiasi problema di infrastruttura condivisa.

L'implementazione tecnica del CSLB ha privilegiato la compatibilità algoritmica con Skipper per preservare la cache locality. Il sistema usa l'algoritmo **xxHash64** su un ring di 64 bit con 100 virtual node per endpoint, con binary search per il routing. L'equivalenza del ring è stata validata tramite unit test su tutte le configurazioni di pod possibili. Per la discovery dei pod su Kubernetes, anziché il polling — che avrebbe rischiato di sovraccaricare il control plane — è stato adottato un pattern **watch-based informer** con debounce di 2 secondi per coalescare i picchi di scale-up.

Il sistema incorpora tre meccanismi di ottimizzazione. Il primo è il **N-Ring Fade-In**: ogni evento di scale-up crea ring indipendenti che fondono il traffico con una curva `^2.5`, permettendo ai nuovi pod di scaldarsi con gli stessi prodotti che serviranno a regime, evitando cache eviction churn e picchi su DynamoDB. Il secondo è il **Occupancy-Based Bounded Load**: applicando la Legge di Little (`occupancy = total_occupied_time / window_duration`), il sistema misura il carico reale dei pod invece del semplice numero di richieste in volo, con un fattore di latency weighting (`effectiveLoad = max(inflight, occupancy) × min(podLatency / globalLatency, 5)`). Il terzo meccanismo, **AZ-Aware Routing**, è implementato ma temporaneamente disabilitato in attesa di stabilizzazione.

I risultati in produzione sono stati significativi: la fleet di Skipper è passata da oltre 50 pod a 8, con un risparmio infrastrutturale di circa $1.000 al giorno. L'ottimizzazione basata sull'occupancy ha permesso di aumentare la soglia HPA dal 50% al 65% di CPU e ridurre ulteriormente del 25% il numero di pod. La migrazione ha anche rivelato — tramite il logging di pod IP e node IP negli errori — ricorrenti freeze di rete di 2-3 secondi a livello di nodo, difetti di infrastruttura precedentemente invisibili sotto l'astrazione del load balancer condiviso.

Il deployment è avvenuto in modo incrementale con tre toggle di sicurezza (`CSLB_ENABLED`, `CSLB_PERCENTAGE`, fallback implicito a Skipper) e una cadenza 1% → 10% → 50% → 100%. Un prerequisito fondamentale è stato l'hardening della pipeline CI/CD, che ha ridotto il tempo di build da 21 a 12 minuti e il tempo medio di deployment da 289 a 128 minuti. L'autore conclude che per la maggior parte delle organizzazioni la risposta è "non costruire il proprio CSLB", ma per traffici genuinamente edge-case come quello di PRAPI il controllo diretto sul routing si è rivelato essenziale.

## Codice

Il calcolo del carico effettivo per il bounded load, che combina richieste in volo, occupancy e latency weighting:

```go
effectiveLoad = max(inflight, occupancy) * min(podLatency/globalLatency, 5)
```

La formula di occupancy basata sulla Legge di Little:

```
occupancy = total_occupied_time / window_duration
```

Distribuzione del traffico durante il N-Ring Fade-In con curva ^2.5:

| Tempo trascorso | Progresso | Quota traffico nuovo pod |
|---|---|---|
| 3s | 10% | 0.3% |
| 15s | 50% | 17.7% |
| 30s | 100% | 100% |
