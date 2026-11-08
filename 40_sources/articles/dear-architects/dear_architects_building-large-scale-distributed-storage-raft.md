---
tags:
  - distributed-systems
  - raft-consensus
  - tikv
  - sharding
  - storage-architecture
feature:
type: article
author: Edward Huang
source: https://tikv.org/blog/building-distributed-storage-system-on-raft/
date: 2026-11-08
---

# Building a Large-scale Distributed Storage System Based on Raft

## Sunto

Edward Huang descrive in questo articolo del blog TiKV come costruire un sistema di storage distribuito su larga scala basato sull'algoritmo di consenso Raft. Il punto di partenza è la strategia di sharding: la scelta tra range-based e hash-based ha implicazioni profonde sulla scalabilità, la gestione degli hotspot e la capacità di eseguire range scan.

Lo **sharding basato su range** mantiene le chiavi in ordine, il che supporta efficacemente le range scan. Il problema è che le scritture sequenziali (es. log con timestamp) creano hotspot sull'ultimo range. TiKV affronta questo problema con split e movement dinamici dei range. Lo **sharding basato su hash** distribuisce uniformemente il carico ma rende difficili le range scan e implica un grande movimento di dati quando si aggiunge un nodo, anche con consistent hashing. TiKV sceglie il range-based sharding, gestendo gli hotspot tramite split automatici.

Il cuore dell'architettura è il sistema di **multi-Raft groups**: i dati vengono divisi in Regions per key range, e ogni Region è un proprio Raft group indipendente. Questo approccio è necessario perché le librerie Raft a singolo gruppo (etcd, Consul, raft-rs) non scalano per lo storage dati. La sfida principale è la gestione di migliaia di gruppi Raft dinamici che si dividono e si riconfigurano.

Gli **split sicuri** sono implementati come log entry Raft, in modo che ogni replica applichi la divisione in modo consistente. Le partizioni di rete e gli snapshot possono creare race condition che richiedono una gestione attenta. I **membership change** seguono il processo di Raft: aggiungere una replica, trasferire la leadership, poi rimuovere quella vecchia. TiKV consente un solo cambiamento di configurazione per Region group alla volta per evitare problemi di maggioranza disgiunta.

Il **Placement Driver (PD)** è il componente centrale di scheduling e routing, ispirato a Google Spanner. Combina la routing table globale con lo scheduler, mantenendo i nodi largamente stateless. PD ricostruisce la routing table dai heartbeat dei nodi (con etcd come cache opzionale), eliminando la necessità di stato persistente.

Il meccanismo di **epoch** usa due clock logici — versione della Region e versione della configurazione — per rilevare informazioni stale durante le partizioni. PD confronta prima la versione della Region e poi quella della configurazione per determinare quale report è più recente.

## Confronto strategie di sharding

| Strategia | Vantaggi | Svantaggi |
|-----------|----------|-----------|
| **Range-based** | Range scan efficienti, split semplice | Hotspot su scritture sequenziali |
| **Hash-based** | Distribuzione uniforme, buono per random write | No range scan, grande data movement su resize |
| **Ibrido** | Combina i vantaggi | Complessità maggiore |

## Insight chiave

- La gestione robusta su larga scala richiede coordinazione automatica e fault-tolerant
- Combinare range sharding con consenso per shard, scheduling stateless e clock logici per il routing
- I client trasparenti con cache della routing table permettono di gestire leadership change senza interruzioni visibili all'applicazione
- Ogni errore "Region not leader" viene gestito lato client aggiornando il leader nella cache

## Codice

L'epoch check in PD per determinare quale heartbeat è più recente:

```go
// Confronto epoch: prima versione Region, poi versione configurazione
func (r *regionInfo) IsEpochStale(epoch *metapb.RegionEpoch) bool {
    return epoch.GetVersion() < r.Meta.GetRegionEpoch().GetVersion() ||
        (epoch.GetVersion() == r.Meta.GetRegionEpoch().GetVersion() &&
            epoch.GetConfVer() < r.Meta.GetRegionEpoch().GetConfVer())
}
```
