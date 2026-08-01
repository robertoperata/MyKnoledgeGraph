---
tags:
  - workflow-orchestration
  - postgresql
  - ai-workflow
  - durable-execution
  - database-architecture
  - distributed-systems
feature:
type: article
author: Jeremy Edberg, Qian Li
source: https://www.infoq.com/presentations/dbos/
date: 2026-08-01
---

# Compiling Workflows into Databases: The Architecture That Shouldn't Work (But Does)

## Sunto

DBOS Transact affronta uno dei problemi più insidiosi nei sistemi distribuiti moderni: gli orchestratori di workflow esterni (come AWS Step Functions o Temporal) introducono nuovi punti di fallimento invece di eliminarli, aggiungono latenza misurabile per ogni step, e richiedono infrastruttura operativa separata. La tesi centrale della presentazione di Jeremy Edberg e Qian Li è che "i workflow sono dati" e che i database relazionali — strumenti affinati in 50 anni di sviluppo per gestire esattamente i dati — sono la fondazione naturale per workflow durabili e fault-tolerant.

L'approccio architetturale è una libreria in-process, non un servizio esterno. DBOS Transact si integra nell'applicazione esistente (FastAPI, Spring Boot, LangChain) senza richiedere rearchitettura. Il meccanismo di checkpoint usa due tabelle PostgreSQL: `workflow_status` (stato complessivo dell'esecuzione con metadata) e `step_outputs` (cache dei risultati dei singoli step via foreign key). Il flusso di esecuzione è deterministico: genera un ID workflow come chiave di idempotenza, scrive gli input nel database, esegue la funzione del workflow normalmente in-process, fa checkpoint degli output a ogni step, e marca lo stato finale. In caso di fallimento, il recovery carica i workflow pending e riprende dall'ultimo step completato.

Una delle innovazioni chiave è l'implementazione della queue tramite `SKIP LOCKED` di PostgreSQL: quando worker multipli interrogano la stessa tabella queue, invece di aspettare che le righe già bloccate si liberino (causando contesa), saltano direttamente alle righe disponibili. Questo permette la distribuzione del lavoro tra worker senza coordinazione esplicita. Il cron scheduling decentralizzato usa il timestamp schedulato come ID univoco del workflow: il constraint di primary key del database garantisce che solo il primo worker che scrive riesca a creare il task, mentre gli altri lo trovano già esistente e lo saltano; il jitter random nei tempi di wake-up distribuisce il carico nel tempo.

I benchmark contro AWS Step Functions mostrano un divario significativo: mentre il numero di step aumenta, la latenza di Step Functions cresce di centinaia di millisecondi per step, mentre l'approccio DBOS mantiene un overhead costante di pochi millisecondi (una scrittura database per step). L'intera architettura si riduce a 2 scritture database per workflow e 1 scrittura per step, numeri che i moderni deployment PostgreSQL (che gestiscono decine di migliaia di transazioni al secondo) possono sostenere facilmente.

I vincoli fondamentali dell'approccio sono due: gli step devono essere idempotenti (ri-eseguire con gli stessi input produce risultati identici — i sistemi AI soddisfano naturalmente questo requisito perché i risultati vengono memorizzati dopo la prima esecuzione), e i workflow devono essere deterministici per poter essere riprodotti correttamente dai checkpoint. DBOS Transact è open-source con licenza MIT, disponibile per Python, TypeScript, Go e Java, tutto costruito su PostgreSQL.

## Codice

Schema delle due tabelle di checkpoint che costituiscono il cuore dell'architettura:

```sql
CREATE TABLE workflow_status (
    workflow_id    TEXT PRIMARY KEY,
    status         TEXT NOT NULL,  -- PENDING | RUNNING | COMPLETED | FAILED
    input          JSONB,
    output         JSONB,
    created_at     TIMESTAMPTZ DEFAULT NOW(),
    updated_at     TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE step_outputs (
    workflow_id    TEXT REFERENCES workflow_status(workflow_id),
    step_name      TEXT NOT NULL,
    output         JSONB,
    executed_at    TIMESTAMPTZ DEFAULT NOW(),
    PRIMARY KEY (workflow_id, step_name)
);
```

Pattern SKIP LOCKED per la distribuzione del lavoro tra worker senza contesa:

```sql
SELECT workflow_id, input
FROM workflow_status
WHERE status = 'ENQUEUED'
ORDER BY created_at
FOR UPDATE SKIP LOCKED
LIMIT 10;
```

Pattern per il cron scheduling decentralizzato che usa il primary key come garanzia di esecuzione singola:

```sql
-- Ogni worker esegue questo: solo il primo INSERT ha successo
-- gli altri ottengono un errore di duplicate key e saltano
INSERT INTO workflow_status (workflow_id, status, input)
VALUES (
    'cron_' || to_char(:scheduled_time, 'YYYY-MM-DD-HH24:MI'),
    'PENDING',
    :workflow_input
)
ON CONFLICT (workflow_id) DO NOTHING;
```
