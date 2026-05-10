---
tags:
  - caching
  - redis
  - distributed-systems
  - infrastructure-scalability
  - proxy-architecture
feature:
type: article
author: Kevin Lin
source: https://www.figma.com/blog/figmas-next-generation-data-caching-platform/
date: 2026-05-10
---

# Figma's Next-Generation Data Caching Platform

## Sunto

Con la crescita di Figma, Redis è passato da infrastruttura non critica a componente mission-critical. L'articolo di Kevin Lin descrive come il team ha progettato **FigCache**, un'infrastruttura di caching di nuova generazione che raggiunge sei nines di uptime (99,9999%) attraverso un'architettura a proxy stateless combinata con librerie client proprietarie.

Il problema principale era multidimensionale: i cluster Redis si avvicinavano ai limiti massimi di connessioni; il rapido scaling dei servizi client causava "thundering herd" durante l'apertura di connessioni massiccia; la frammentazione delle metriche su librerie client disparate complicava la diagnosi degli incidenti; le applicazioni potevano corrompere dati cross-cluster senza un traffic management centralizzato. Le soluzioni esistenti sul mercato sono state scartate perché incapaci di estrazione semantica dei comandi Redis, di supporto per comandi custom (distributed locking multi-cluster, graceful connection draining) e di flessibilità ecosistemica.

L'architettura di FigCache separa il proxy in due layer distinti. Il **frontend layer** gestisce l'I/O di rete, il parsing del protocollo RESP (Redis Serialization Protocol) tramite il framework ResPC (portmanteau di RESP e RPC), e la gestione delle connessioni client. Il **backend layer** si occupa dell'elaborazione dei comandi, del connection multiplexing verso i backend Redis (ElastiCache), e dell'esecuzione fisica. Questa separazione consente di scalare i componenti indipendentemente e di inserire logica custom nel data plane (cifratura, guardrail, backpressure).

Un'innovazione chiave è il **fanout engine**: Redis Cluster restituisce errori `CROSSSLOT` per operazioni che attraversano hash slot diversi, impedendo garanzie atomiche su operazioni multi-chiave. Il fanout engine intercetta le pipeline read-only ammissibili e le decompone internamente in operazioni scatter-gather parallelizzate per shard, riassemblando i risultati in modo trasparente per il client. La configurazione dei behavior runtime è espressa tramite programmi Starlark valutati all'avvio, eliminando la necessità di ricompilare il binario server per modificare il routing del traffico.

La migrazione è avvenuta in tre fasi: deploy delle librerie client proprietarie (Go, Ruby, TypeScript) mantenendo compatibilità di interfaccia con i client open-source esistenti; productionizzazione del proxy Redis interno; migrazione graduale e reversibile delle applicazioni con traffic shifting incrementale per carico elevato e feature flag per il revert d'emergenza. Il risultato: le connessioni ai cluster Redis sono scese di un ordine di grandezza e sono diventate significativamente meno volatili, eliminando la classe di incidenti da thundering herd e rendendo le operazioni di manutenzione (rotazione hardware, upgrade OS, patch di sicurezza, failover di shard) eseguibili come operazioni di background routine.

## Codice

Esempio di configurazione Starlark per il routing dei comandi tra engine Redis:

```python
# Configurazione del router comandi tra cluster Redis diversi
cmd_router = enginepb.Router(
    rules = [
        enginepb.Rule(
            command = respcpb.Schema(name = "GET"),
            engine = redis_foo,
        ),
        enginepb.Rule(
            command = respcpb.Schema(name = "SET"),
            engine = bar_router,
        ),
    ],
)
# La configurazione Starlark viene valutata all'avvio del proxy,
# eliminando la necessità di ricompilare il binario per modificare il routing.
```
