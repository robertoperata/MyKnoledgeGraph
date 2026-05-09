---
tags:
  - java
  - aws
  - event-driven
  - microservices
type: article
author: Tomaz Fernandes
source: https://tomazfernandes.dev/posts/from-sqslistener-to-your-method/
date: 2026-05-05
---

# What Happens Between @SqsListener and Your Method in Spring Cloud AWS SQS

## Sunto

L'articolo spiega i meccanismi interni di Spring Cloud AWS SQS: cosa succede tra l'annotazione `@SqsListener` su un metodo e l'effettiva invocazione di quel metodo con un messaggio. L'autore mostra che dietro l'apparente semplicità dell'annotazione si nasconde una **pipeline asincrona composabile** che orchestra il flusso dei messaggi dalla coda SQS al codice applicativo.

Il ciclo di vita è diviso in due fasi distinte. Nella **fase di assembly**, il framework rileva le annotazioni `@SqsListener` tramite `SqsListenerAnnotationBeanPostProcessor` e le converte in oggetti `Endpoint` che descrivono la configurazione del listener. Questi endpoint vengono poi processati da una `MessageListenerContainerFactory` che crea le istanze di `MessageListenerContainer` responsabili dell'esecuzione reale.

Nella **fase di runtime**, il container gestisce quattro stadi in sequenza:

| Stadio | Componenti principali | Responsabilità |
|---|---|---|
| **Ingress / Polling** | `MessageSource`, `BackPressureHandler` | Controlla l'ingresso dei messaggi e bilancia throughput con capacità disponibile |
| **Dispatch** | `MessageSink` | Instrada i messaggi in base alla semantica della coda (standard, FIFO, batch) |
| **Processing** | `MessageInterceptor`, `MessageListener`, `ErrorHandler`, `AcknowledgementHandler` | Pipeline interna che coordina intercettazione, elaborazione ed error handling |
| **Acknowledgement** | `AcknowledgementHandler`, `AcknowledgementProcessor`, `AcknowledgementExecutor`, `AcknowledgementResultCallback` | Decide la strategia di cancellazione e gestisce i fallimenti parziali |

Il *dispatch* differenzia il comportamento in base al tipo di coda: le code standard ricevono delivery parallela, le code FIFO mantengono l'ordine per gruppo, e i batch listener ricevono insiemi di messaggi. Questa distinzione è fondamentale perché FIFO richiede garanzie di ordinamento che vincola il parallelismo.

L'intera architettura si basa sulla composizione di `CompletableFuture` per un'orchestrazione **non bloccante**: il framework gestisce la complessità asincrona sottostante, permettendo agli sviluppatori di scrivere codice bloccante nei propri listener senza preoccuparsi dei thread. Ogni componente della pipeline esiste nelle varianti sincrona e asincrona. Il *back pressure* è il meccanismo chiave che impedisce il sovraccarico bilanciando il rate di polling con la capacità di elaborazione disponibile.

---

## Link esterni

- [Companion example project](https://github.com/tomazfernandes/tomazfernandes-dev/tree/main/examples/from-sqslistener-to-your-method) — progetto GitHub con scenari eseguibili che dimostrano assembly, interception, error handling e acknowledgement

---

## Immagini

Nessuna immagine presente
