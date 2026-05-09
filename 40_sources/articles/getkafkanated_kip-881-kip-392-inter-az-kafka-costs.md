---
tags:
  - streaming
  - distributed-systems
  - cloud
  - aws
  - performance
type: article
author: Stanislav Kozlovski
source: https://getkafkanated.substack.com/p/how-kip-881-and-kip-392-reduce-inter
date: 2026-02-20
---

# How KIP-881 and KIP-392 Reduce Inter-AZ Networking Costs in Classic Kafka

**Pubblicazione:** Get Kafka-Nated (Substack)

## Sunto

Il trasferimento dati tra availability zone (*inter-AZ*) è la componente di costo dominante di un cluster Kafka su cloud. L'articolo mostra come due Kafka Improvement Proposals — **KIP-392** (Fetch From Follower) e **KIP-881** (Rack-Aware Partition Assignment) — permettano di ridurre i costi del cluster di circa il **50%** allineando il traffico di consume alla zona del consumer.

In un cluster Kafka distribuito su 3 AZ, il comportamento di default produce una quantità enorme di traffico cross-zona: due terzi delle fetch dei consumer finiscono su broker in zone diverse, tutto il traffico di replicazione attraversa zone, e il traffico di replicazione ammonta a 2× il traffico del producer. L'articolo quantifica questi costi con un esempio concreto: 10 MB/s di scritture con fanout 5× generano **$36.9k/anno** in data transfer charges su AWS — di cui $20.5k sono solo i consumer read (la componente più grande).

KIP-392 (disponibile da Kafka 2.4) permette ai consumer di leggere da follower nella propria AZ invece che sempre dal leader. KIP-881 (Kafka 3.5+) risolve due problemi che KIP-392 da solo non affronta: lo sbilanciamento del carico sui broker quando i follower non sono distribuiti uniformemente, e il caso in cui il replication factor è inferiore al numero di AZ del cluster (alcune AZ non hanno repliche locali di certi topic).

Un avvertimento critico riguarda AWS: l'uso di **IP pubblici** all'interno della stessa AZ viene comunque fatturato come traffico cross-zona. Per beneficiare effettivamente del risparmio, è obbligatorio usare IP privati (same VPC, VPC peering, o Private Link).

---

## Il costo del traffico cross-AZ in un cluster Kafka standard
In un cluster standard su 3 AZ, il traffico si distribuisce così:

| Tipo di traffico | Comportamento | Quota cross-AZ |
|---|---|---|
| **Producer → Leader** | Il producer scrive sul leader della partizione | Dipende dalla distribuzione dei leader |
| **Replicazione** | Il leader invia a tutti i follower ISR | ~100% (i follower sono in altre AZ) |
| **Consumer → Leader** | Il consumer legge sempre dal leader (default) | ~2/3 (2 AZ su 3 hanno il leader altrove) |

**Il traffico di replicazione = 2× il traffico producer** (con RF=3, ogni messaggio va copiato in 2 altre AZ).

### Esempio di costo concreto (10 MB/s write, 5× fanout)

![Crescita dei costi consumer in funzione del fanout ratio](https://substack-post-media.s3.amazonaws.com/public/images/09c891d2-a5ca-4e91-a947-b36ac61745ca_700x600.png)

Con 10 MB/s di scritture e 50 MB/s di letture (fanout 5×), a $0.02/GiB (AWS):

| Componente | Volume mensile | Costo annuo |
|---|---|---|
| Producer cross-zone | 16.72 TiB | $4.1k |
| Replicazione cross-zone | 50.15 TiB | $12.3k |
| Consumer read cross-zone | 83.58 TiB | **$20.5k** |
| **Totale** | | **$36.9k** |

> Il traffico di consume da solo supera la somma di producer + replicazione. Con fanout elevati, è la componente dominante del bill.

---

## KIP-392 — Fetch From Follower

L'idea è semplice: i dati sotto l'*high watermark* sono già replicati su tutti i follower ISR. Un log append-only non può produrre read "stale" per dati già committati. Quindi è sicuro permettere ai consumer di leggere da un follower nella propria AZ.

**Configurazione:**

| Configurazione | Valore | Dove |
|---|---|---|
| `client.rack` | ID dell'AZ (es. `eu-west-1a`) | Consumer, Connect Worker, MirrorMaker2 |
| `broker.rack` | ID dell'AZ del broker | Broker (già spesso configurato) |
| `replica.selector.class` | `org.apache.kafka.common.replica.RackAwareReplicaSelector` | Broker |

Il consumer invia il proprio `client.rack` in ogni fetch request; il `RackAwareReplicaSelector` nel broker instrada la richiesta al follower nella stessa AZ.

![KIP-392: consumer nella stessa AZ del follower, nessun traffico cross-zona](https://substack-post-media.s3.amazonaws.com/public/images/23e2999c-ae7f-4d2f-8cee-a00524efa6bd_1220x976.png)

**Requisito minimo:** Kafka 2.4.

---

## KIP-881 — Rack-Aware Partition Assignment

KIP-392 da solo non è sufficiente in due scenari:

### Problema 1: sbilanciamento del carico sui broker

L'assegnazione di default bilancia le risorse dei *client* (un consumer per partizione), non quelle dei *broker*. Se i follower non sono distribuiti uniformemente tra i broker di una AZ, reindirizzare i consumer verso i follower locali può concentrare il carico su pochi broker.

![Sbilanciamento del carico: KIP-392 senza KIP-881 può concentrare il traffico su un subset di broker](https://substack-post-media.s3.amazonaws.com/public/images/7801c5f4-bb38-4b0c-bab4-bff567a63538_1220x786.png)

### Problema 2: mismatch tra replication factor e numero di AZ

Se il cluster ha più AZ del replication factor, alcune AZ non hanno repliche locali di certi topic. Con RF=3 e 5 AZ, le AZ 4 e 5 non hanno repliche: i consumer in quelle AZ devono comunque fare fetch cross-zona.

![Cluster con 5 AZ e RF=3: le AZ 4 e 5 non hanno repliche locali, traffico cross-zona inevitabile senza KIP-881](https://substack-post-media.s3.amazonaws.com/public/images/94175ef9-7cb3-4625-8f8d-a2d8f7810b6e_1600x1100.png)

### Soluzione KIP-881

KIP-881 propaga l'informazione di rack del consumer fino all'*assignor* (il componente del consumer group che assegna le partizioni). L'assignor assegna a ogni consumer solo partizioni che hanno una replica nella sua AZ.

![KIP-881: assegnazione rack-aware — ogni consumer ottiene partizioni con repliche locali, nessun traffico cross-zona](https://substack-post-media.s3.amazonaws.com/public/images/9cf012b3-dc33-44d4-8d17-2c94e4bc265d_1600x1100.png)

**Stato di supporto:**

| Protocollo consumer group | Stato rack-aware | Note |
|---|---|---|
| **v1** (classic) | ✅ Supportato (priorità secondaria) | Gli assignor di default (range, cooperative sticky) gestiscono il rack come criterio secondario |
| **v2** (KIP-848) | ❌ Non ancora supportato | Tracciato in KAFKA-19387 — richiede assignor custom per ora |

**Requisito minimo:** Kafka 3.5.

---

## ⚠️ Attenzione: il problema degli IP pubblici su AWS

![AWS Data Transfer cost calculator — verifica i costi reali prima di andare in produzione](https://substack-post-media.s3.amazonaws.com/public/images/394317af-74dc-49a4-9bf1-3d074cf955db_735x479.gif)

Su AWS, anche il traffico **nella stessa AZ** viene fatturato come cross-zona se:
- Si usano **IP pubblici IPv4**
- Si usano **IP pubblici IPv6** tra VPC diversi non connessi tramite VPC peering

**Per beneficiare effettivamente del risparmio, è obbligatorio usare IP privati.**

Opzioni per garantire IP privati tra client e cluster Kafka:

| Opzione | Scenario tipico | Costo extra |
|---|---|---|
| Same VPC | Client e broker nello stesso VPC | Nessuno |
| VPC Peering | Client in VPC diverso | Basso |
| Private Link / Transit Gateway | Setup multi-account, organizzazioni grandi | Aggiuntivo — valutare |

---

## Riepilogo configurazione

```properties
# Consumer (ogni consumer, Connect Worker, MirrorMaker2)
client.rack=eu-west-1a  # ID della propria AZ

# Broker
broker.rack=eu-west-1a  # già configurato nella maggior parte dei cluster
replica.selector.class=org.apache.kafka.common.replica.RackAwareReplicaSelector  # KIP-392
```

Per KIP-881 non è necessaria configurazione aggiuntiva con il protocollo v1 — è sufficiente che `client.rack` sia configurato sui consumer e che Kafka sia ≥ 3.5.

---

## Link esterni

- [KIP-392: Fetch From Follower](https://cwiki.apache.org/confluence/display/KAFKA/KIP-392%3A+Allow+consumers+to+fetch+from+closest+replica) — specifica ufficiale
- [KIP-881: Rack-aware Partition Assignment](https://cwiki.apache.org/confluence/display/KAFKA/KIP-881%3A+Rack-aware+Partition+Assignment+for+Kafka+Consumers) — specifica ufficiale
- [KAFKA-19387](https://issues.apache.org/jira/browse/KAFKA-19387) — tracking issue per rack-aware support nel protocollo v2 (KIP-848)
- [AWS Data Transfer Pricing](https://aws.amazon.com/ec2/pricing/on-demand/#Data_Transfer) — pricing ufficiale per verificare i costi inter-AZ
