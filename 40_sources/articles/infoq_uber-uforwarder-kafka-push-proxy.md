---
tags:
  - streaming
  - event-driven
  - microservices
  - distributed-systems
  - performance
type: article
author: Leela Kumili
source: https://www.infoq.com/news/2026/02/uber-uforwarder-kafka-push-proxy
date: 2026-02-23
---

# Uforwarder: Uber's Scalable Kafka Consumer Proxy for Efficient Event-Driven Microservices

## Sunto

Uber Engineering ha rilasciato in open source **uForwarder**, un *consumer proxy push-based* per Apache Kafka progettato per migliorare scalabilità, efficienza e controllo operativo nei sistemi di event streaming ad alta throughput. Il sistema si interpone come strato intermediario tra Kafka e i servizi consumer, sostituendo le implementazioni dirette del client Kafka con un'interfaccia push basata su **gRPC**.

Il contesto di partenza è la scala di Uber: oltre 1.000 servizi consumer downstream, trilioni di messaggi e diversi petabyte di dati processati ogni giorno. I consumer group tradizionali di Kafka presentavano limitazioni di scalabilità significative: gestione complessa delle partizioni, supporto linguistico inconsistente, overhead operativo. Ogni servizio doveva implementare autonomamente logica di offset handling, retry e delay, moltiplicando il rischio di inefficienze.

Le quattro funzionalità chiave di uForwarder sono:

| Feature | Descrizione |
|---|---|
| **Context-Aware Routing** | Gli header dei messaggi Kafka propagano metadati di routing nelle chiamate gRPC, consentendo decisioni di delivery a livello infrastrutturale (regione, tenant, ambiente) senza filtering applicativo |
| **Out-of-Order Commit Tracker** | Monitora il progresso dei commit indipendentemente; i messaggi problematici vengono reindirizzati verso *dead letter queue* avanzando il commit pointer, prevenendo lo stallo della partizione |
| **Consumer Auto Rebalancer** | Valuta continuamente CPU, memoria e throughput tra i worker, ridistribuendo le partizioni per bilanciare il carico e scalare dinamicamente |
| **DelayProcessManager** | Abilita pausa e ripresa a livello di partizione per la gestione granulare del *backpressure*: solo le partizioni bloccate vengono messe in buffer, le altre continuano normalmente |

L'architettura risultante migliora l'isolamento dei workload, riduce il consumer lag e ottimizza l'utilizzo hardware, semplificando al contempo la logica dei servizi consumer. uForwarder è diventato internamente la modalità dominante di consumo Kafka in Uber. Gli sviluppi pianificati includono expanded queue capacity, meccanismi di offset rewinding con side consumer processing per dati ritardati, e supporto nativo a Protobuf per consentire ai servizi di ricevere messaggi strutturati direttamente.

---

## Link esterni

- [uForwarder su GitHub](https://github.com/uber/uForwarder) — repository open source del progetto

---

## Immagini

- ![Architettura high-level del consumer proxy](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/news/2026/02/uber-uforwarder-kafka-push-proxy/en/resources/1uber-uforwarder-highlevel-1771098473968.jpeg) — overview dell'architettura con uForwarder come intermediario tra Kafka e i servizi consumer
- ![Context-aware routing](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/news/2026/02/uber-uforwarder-kafka-push-proxy/en/resources/1uforwarder-contextaware-1771098473968.jpeg) — propagazione dei metadati di routing via header Kafka nelle chiamate gRPC
- ![Delay processing architecture](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/news/2026/02/uber-uforwarder-kafka-push-proxy/en/resources/1uforwaderdelayprocessing-1771098473968.jpeg) — gestione del backpressure a livello di partizione con DelayProcessManager
