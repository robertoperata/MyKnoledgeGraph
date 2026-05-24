---
tags:
  - queue-theory
  - capacity-planning
  - distributed-systems
  - kafka
  - resilience
feature:
type: article
author: Rajesh Kumar Pandey
source: https://www.infoq.com/articles/capacity-planning-queue-recovery/
date: 2026-05-24
---

# Capacity Planning with Queues: Modeling Recovery Time Under Load

## Sunto

L'articolo trasforma la gestione dei backlog delle code da guesswork a disciplina ingegneristica, fornendo formule operative per il capacity planning nei sistemi basati su code (Kafka, SQS, RabbitMQ, Redis). Il punto di partenza sono tre numeri fondamentali: il tasso di arrivo (λ — messaggi al secondo che entrano nella coda), il tasso di elaborazione (μ — messaggi che un singolo consumer gestisce al secondo), e il numero di consumer (c). La capacità totale è `c × μ`, e quando supera il tasso di arrivo il sistema è stabile.

Il ciclo di vita di un backlog si articola in tre fasi ben definite. Nella **fase di accumulo** il tasso di crescita è `arrival_rate - effective_processing_capacity`: se 15 consumer crashano riducendo la capacità da 10.000 a 4.000 msg/sec, si accumulano 6.000 messaggi al secondo. Nella **fase di stabilizzazione** la causa radice è risolta ma la coda non si svuota. Nella **fase di drain** il surplus è `total_capacity - arrival_rate` e il tempo di recupero è `backlog_size / surplus`. Un'intuizione critica: i sistemi dimensionati esattamente per il traffico nominale hanno surplus zero e quindi capacità di recovery nulla.

La Legge di Little (`queue_depth = arrival_rate × time_in_queue`) è l'invariante fondamentale che collega le tre variabili del sistema. Conoscendone due si ricava la terza, rendendo possibile comunicare l'impatto del backlog in termini comprensibili: con 600.000 messaggi e un tasso di arrivo di 5.000 msg/sec, ogni nuovo messaggio attende ~120 secondi. Le complicazioni reali includono il degrado durante il drain (i messaggi in backlog sono stale — cache miss, refresh di token — che rallentano il throughput), i pattern di traffico non uniformi (il surplus esiste solo nelle ore di bassa intensità), e l'amplificazione dei retry, il feedback loop più pericoloso: il crescere della coda aumenta i timeout → più retry → tasso di arrivo effettivo ancora più alto → la coda cresce anche dopo aver risolto la causa radice.

La formula di capacity planning integrata permette di calcolare il numero di consumer necessari in funzione degli obiettivi di recovery: `consumers_needed = (arrival_rate / processing_rate) + (max_backlog / (processing_rate × rto))`. Con 10.000 msg/sec di arrivo, 400 msg/sec per consumer, 5 milioni di messaggi come worst-case e un RTO di 30 minuti si ottengono 32 consumer (28% di overhead rispetto allo steady-state). Per l'auto-scaling, l'articolo raccomanda di reagire al **tasso di variazione** della coda piuttosto che alla profondità assoluta: quando la crescita persiste da 2+ minuti si calcola il backlog stimato al momento in cui le nuove istanze saranno operative.

Il dead letter queue (DLQ) è parte integrante del modello: i messaggi "veleno" (payload non validi, schema mismatch) consumano capacità senza progressi. Dopo un numero limitato di retry, isolare questi messaggi nel DLQ protegge il throughput effettivo durante il recovery. La raccolta sistematica di metriche post-incidente (peak backlog, degrado effettivo, amplificazione dei retry) permette, dopo 3-4 incidenti, di calibrare i parametri del modello sulla realtà specifica del sistema.

## Codice

Formula fondamentale per il tasso di crescita del backlog durante un'interruzione:

```
growth_rate = arrival_rate - effective_processing_capacity
```

Legge di Little applicata al calcolo dell'attesa media per i messaggi:

```
queue_depth = arrival_rate × time_in_queue
```

Formula per il tempo di drain dopo la stabilizzazione:

```
surplus = total_processing_capacity - arrival_rate
drain_time = backlog_size / surplus
```

Formula per l'amplificazione dei retry (metastable failure):

```
effective_arrival_rate = base_arrival_rate × (1 + retries_per_timeout × timeout_probability)
```

Formula completa per il capacity planning con obiettivo RTO:

```
consumers_needed = (arrival_rate / processing_rate) +
                   (max_backlog / (processing_rate × rto))
```

Strategia di auto-scaling basata sul tasso di variazione invece che sulla profondità assoluta:

```python
if queue_growth_rate > 0 for 2+ minutes:
    estimated_backlog = current_depth + (growth_rate × scale_up_time)
    target_consumers = calculate_based_on_rto(estimated_backlog)
```

Formula per la durata massima tollerabile di un incidente prima di violare l'RTO:

```
max_incident_duration = rto × surplus / accumulation_rate
```
