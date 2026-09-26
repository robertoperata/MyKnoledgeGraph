---
tags:
  - postgresql
  - workflow-orchestration
  - durable-execution
  - database-patterns
  - distributed-systems
feature:
type: article
author: Raman Varma
source: https://www.infoq.com/articles/durable-workflows-postgres/
date: 2026-09-26
---

# Implementing Durable Workflows on Postgres Without an External Orchestrator

## Sunto

L'articolo di Raman Varma propone un'architettura alternativa all'uso di orchestratori dedicati come Temporal o AWS Step Functions: implementare workflow durevoli direttamente su PostgreSQL, sfruttando i primitivi relazionali già disponibili. Il principio fondamentale è la *durable execution*: ogni workflow salva il proprio stato in database come checkpoint, consentendo a un nuovo processo di riprendere da dove il precedente si era interrotto, come un sistema di autosave in un videogioco.

Il meccanismo chiave per l'acquisizione esclusiva dei task è `SELECT ... FOR UPDATE SKIP LOCKED`, che garantisce semantica *exactly-once* anche con più worker concorrenti che interrogano la stessa tabella. Due server possono fare polling contemporaneamente senza che lo stesso workflow venga assegnato a due esecutori diversi. I checkpoint idempotenti sono implementati con chiave primaria `(execution_id, step_id)` e `ON CONFLICT DO NOTHING`, rendendo sicura la ri-esecuzione di step già completati.

Per la gestione dei fault dei worker, viene adottato un pattern *lease-and-sweeper*: ogni worker aggiorna periodicamente un timestamp `lease_expires` (heartbeat). Un processo sweeper separato reimposta allo stato `enqueued` tutte le esecuzioni il cui lease è scaduto, permettendo il recovery automatico da crash senza dipendenze esterne. Anche le attese di lunga durata (approvazioni umane, timer) sono rappresentate come righe in una tabella `workflow_waits`, persistendo il workflow attraverso restart dell'applicazione o reschedule Kubernetes.

Rispetto a un orchestratore dedicato, l'approccio postgres-backed elimina la complessità di un sistema stateful aggiuntivo, riduce la superficie di attacco, e semplifica l'osservabilità: monitorare i workflow diventa semplicemente SQL standard. L'architettura scala orizzontalmente aggiungendo worker stateless, e beneficia della HA già in uso su PostgreSQL (repliche, multi-AZ).

I limiti dell'approccio sono esplicitati con chiarezza: il table churn genera bloat (mitigabile con autovacuum e partitioning), la pressione sulle connessioni richiede PgBouncer in transaction pooling, e la latenza di dispatch rende l'approccio inadatto a workload con migliaia di worker o requisiti di latenza inferiori al secondo. Per automazioni I/O-bound con concorrenza moderata, tuttavia, riutilizzare un'installazione PostgreSQL esistente riduce significativamente la complessità infrastrutturale.

## Codice

Schema delle tabelle principali per la gestione dei workflow:

```sql
CREATE TABLE workflow_executions (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    workflow_name TEXT NOT NULL,
    status TEXT NOT NULL DEFAULT 'enqueued',
    input JSONB NOT NULL,
    owner_id TEXT,
    lease_expires TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE operation_outputs (
    execution_id BIGINT NOT NULL REFERENCES workflow_executions(id),
    step_id TEXT NOT NULL,
    output JSONB NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (execution_id, step_id)
);

CREATE TABLE workflow_waits (
    execution_id BIGINT NOT NULL REFERENCES workflow_executions(id),
    step_id TEXT NOT NULL,
    wake_at TIMESTAMPTZ NOT NULL,
    PRIMARY KEY (execution_id, step_id)
);
```

Acquisizione esclusiva di un workflow con `FOR UPDATE SKIP LOCKED`:

```sql
BEGIN;
WITH claimed AS (
    SELECT id FROM workflow_executions
    WHERE status = 'enqueued'
    ORDER BY created_at LIMIT 1
    FOR UPDATE SKIP LOCKED
)
UPDATE workflow_executions e
SET status = 'running', owner_id = $1,
    lease_expires = now() + INTERVAL '30 seconds',
    updated_at = now()
FROM claimed WHERE e.id = claimed.id
RETURNING e.id, e.input;
COMMIT;
```

Salvataggio idempotente dell'output di uno step:

```sql
INSERT INTO operation_outputs (execution_id, step_id, output)
VALUES ($1, $2, $3)
ON CONFLICT (execution_id, step_id) DO NOTHING
RETURNING output;
```

Recovery automatico da crash tramite sweeper periodico:

```sql
UPDATE workflow_executions
SET status = 'enqueued', owner_id = NULL
WHERE status = 'running' AND lease_expires < now();
```
