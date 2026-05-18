---
tags:
  - aws
  - serverless
  - streaming
  - microservices
  - cost-optimization
feature:
type: article
author: Daniele Frasca
source: https://www.infoq.com/presentations/streaming-application-aws-infrastructure/
date: 2026-05-18
---

# Evolution of a Backend for a Streaming Application

## Sunto

Daniele Frasca, AWS Serverless Community Builder presso ProSiebenSat.1 Media, descrive il percorso di trasformazione architetturale di Joyn, una piattaforma streaming tedesca con milioni di utenti in Austria, Svizzera e Germania. In diciotto mesi, un team di due persone ha trasformato un backend fragile e mono-nodo in un'architettura serverless multi-regione, resiliente e cost-effective.

Il problema originale era critico: nodo database singolo senza ridondanza, dati inconsistenti tra sei servizi, cicli di deploy di 90 minuti, crash dell'infrastruttura durante i picchi di traffico e servizi che esponevano lo stato interno al bus eventi aziendale. La soluzione ha richiesto l'introduzione del pattern Hub-and-Spoke con AWS EventBridge come layer di astrazione tra Kafka (bus aziendale) e i singoli servizi, risolvendo i problemi di consistenza dei dati e di boundaries tra servizi.

Per la scalabilità, il team ha implementato un'architettura multi-layer con Route 53 + CloudFront/Global Accelerator per il routing edge, API Gateway o Application Load Balancer per la distribuzione delle richieste, Lambda (da 0 a 1000 istanze in millisecondi) e Fargate per la computazione, Momento Cache in sostituzione di Redis, e DynamoDB + Aurora DSQL come database completamente serverless. L'isolamento cell-based ha suddiviso i servizi per regione geografica, tipo di utente (paid/free) e piattaforma.

Una strategia di caching a tre livelli ha ridotto gli accessi al database a meno del 10% durante i picchi: CloudFront per le richieste ripetitive, caching in-memory in Lambda/Fargate, e Momento Cache prima del database. La configurazione multi-regione active-active con DynamoDB global tables garantisce la replica in tempo quasi reale tra regioni, eliminando i single point of failure.

Le ottimizzazioni di costo sono state significative: il passaggio da API Gateway ad Application Load Balancer ha portato a un risparmio del 90%. Il traffico viene spostato dinamicamente tra Fargate e Lambda in base al carico, con Fargate scalato a zero durante la notte. I risultati finali: zero outage legati all'infrastruttura, deploy ridotti da 90 minuti a pochi minuti, 99.99% di disponibilità con ALB + Lambda.

## Immagini

Nessuna immagine tecnica disponibile nella presentazione.
