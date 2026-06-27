---
tags:
  - context-engineering
  - ai-agents
  - apache-kafka
  - memory-management
  - distributed-systems
feature:
type: article
author: Adi Polak
source: https://www.infoq.com/presentations/context-engineering-data/
date: 2026-06-27
---

# Beyond Prompting: Context Engineering and Memory Management for AI Systems at Scale

## Sunto

Adi Polak (Director at Confluent), forte di 15 anni di esperienza in sistemi distribuiti, illustra la transizione dai prompt stateless agli agenti AI stateful e context-aware. La tesi fondamentale è che il prompt engineering — pur essendo necessario — rappresenta solo una componente di un framework più ampio: il "context engineering", che include gestione della memoria a breve e lungo termine, state management e orchestrazione degli strumenti.

Il framework di context engineering proposto da Polak si articola in cinque dimensioni. La prima è il prompt engineering classico (role assignment, few-shot examples, chain-of-thought, constraint setting). La seconda è la memoria a breve termine, basata su log di conversazione che richiedono summarizzazione gerarchica per rimanere nei limiti del contesto. La terza è la memoria a lungo termine, implementata tramite database vettoriali e RAG. La quarta è lo state management, che traccia il progresso su task multi-step. La quinta è l'accesso agli strumenti tramite MCP, che permette al modello di invocare API esterne in modo standardizzato.

Per la gestione della memoria distribuita, Polak propone uno stack basato su Apache Kafka e Apache Flink. Kafka funge da event streaming pub/sub con garanzie "exactly once" di consistenza. Flink elabora i dati in real-time con latenza a livello di millisecondi. La strategia di tiered storage prevede SSD per la memoria a breve termine (session-based) e object storage (S3/Blob) per la memoria a lungo termine, con aging automatico dei dati verso lo storage economico. RocksDB viene utilizzato per il checkpointing dello stato persistente.

Un caso d'uso concreto è presentato con E*TRADE: l'anomaly detection nei volumi di trading combina dati di mercato in streaming (Kafka) con processing Flink per l'ottimizzazione real-time delle soglie. Un agente AI addestrato sui dati di trading raccomanda aggiustamenti delle soglie per gli algoritmi di anomaly detection, progredendo gradualmente tramite A/B testing da semplici alert fino a decisioni autonome.

L'architettura Kora di Confluent, ispirata al design di AWS S3, introduce un approccio a "celle" (cell-based architecture) che separa compute da storage, usando volumi SSD VM-attached come strato di memoria. Questo permette una resilienza elevata durante outage cloud e supporta multi-tenancy efficiente. La conclusione di Polak sintetizza il cambio di paradigma: "Stiamo passando da applicazioni stateless a sistemi state-aware che richiedono memoria ingegnerizzata a più livelli."

## Codice

Invocazione di strumenti SQL-based tramite MCP con la keyword `AI_TOOL_INVOKE`:

```sql
-- Pattern MCP per invocazione strumenti con gestione sicura delle API key
SELECT AI_TOOL_INVOKE(
  tool_name => 'deepwiki_search',
  params => '{"query": "context engineering patterns", "limit": 5}',
  secret_ref => 'my_api_key_secret'
) FROM dual;
```
