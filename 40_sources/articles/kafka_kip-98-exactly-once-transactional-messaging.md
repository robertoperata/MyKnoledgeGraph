---
tags:
  - streaming
  - event-driven
  - distributed-systems
  - java
type: article
author: Jason Gustafson, Flavio Paiva Junqueira, Apurva Mehta, Sriram, Guozhang Wang
source: https://cwiki.apache.org/confluence/display/KAFKA/KIP-98+-+Exactly+Once+Delivery+and+Transactional+Messaging
date: 2026-05-05
---

# KIP-98 — Exactly Once Delivery and Transactional Messaging

**Stato:** Adopted  
**JIRA:** KAFKA-4815

---

## Sunto

KIP-98 è la proposta che ha introdotto la **semantica exactly-once** in Apache Kafka, attraverso due meccanismi complementari: il *producer idempotente* e il *producer transazionale*. L'obiettivo è superare la semantica "at least once" di default, eliminando duplicati e garantendo scritture atomiche su più topic-partition.

Il **producer idempotente** assegna ad ogni producer un *Producer ID (PID)* univoco. I messaggi portano numeri di sequenza monotonicamente crescenti per partizione: il broker rifiuta o scarta i messaggi out-of-order, garantendo che ogni messaggio venga persisted esattamente una volta all'interno di una sessione producer. Questa garanzia è per singola sessione — un riavvio del producer genera un nuovo PID.

Il **producer transazionale** introduce il concetto di `TransactionalId` — un identificatore stabile fornito dall'utente che sopravvive ai riavvii. Permette di raggruppare in un'unica **transazione atomica** scritture su più topic-partition, inclusi i commit di offset dei consumer. Questo è il pattern fondamentale per *consume-transform-produce* (Kafka Streams): leggere da un topic, trasformare, scrivere su un altro topic, e committare l'offset di input — tutto atomicamente.

Il meccanismo chiave è il **Transaction Coordinator** (analogo al Group Coordinator per i consumer group): gestisce lo stato delle transazioni in un *transaction log* topic interno. Il **Producer Epoch** permette di fare "fencing" dei *zombie producer* — istanze precedenti con lo stesso `TransactionalId` che potrebbero ancora tentare di scrivere. L'`initTransactions()` garantisce che qualsiasi transazione incompleta della sessione precedente venga recuperata o abortita prima di iniziarne una nuova.

Sul lato consumer, il parametro `isolation.level=read_committed` fa sì che il consumer veda solo messaggi appartenenti a transazioni committate. Il *Last Stable Offset (LSO)* è l'offset massimo che può essere restituito in modalità `read_committed` — i messaggi delle transazioni aperte sono bufferizzati fino al commit o abort.

---

## Producer API (5 nuovi metodi)

```java
// 1. Inizializza il producer transazionale — recupera/abortisce transazioni precedenti
void initTransactions() throws IllegalStateException;

// 2. Inizia una nuova transazione
void beginTransaction() throws ProducerFencedException;

// 3. Committare gli offset del consumer atomicamente nella transazione corrente
void sendOffsetsToTransaction(
    Map<TopicPartition, OffsetAndMetadata> offsets,
    String consumerGroupId
) throws ProducerFencedException;

// 4. Committa la transazione — rende visibili tutti i messaggi inviati
void commitTransaction() throws ProducerFencedException;

// 5. Abortisce la transazione — scarta tutti i messaggi inviati
void abortTransaction() throws ProducerFencedException;
```

> `ProducerFencedException` = un'altra istanza con lo stesso `TransactionalId` ha preso il controllo (epoch più alto). Il producer corrente è uno zombie e deve terminare.

---

## Pattern consume-transform-produce (Kafka Streams)

```java
// Configurazione producer transazionale
Properties producerConfig = new Properties();
producerConfig.put(ProducerConfig.TRANSACTIONAL_ID_CONFIG, "my-transactional-id");
producerConfig.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, true);

KafkaProducer<String, String> producer = new KafkaProducer<>(producerConfig);
KafkaConsumer<String, String> consumer = new KafkaConsumer<>(consumerConfig);

producer.initTransactions(); // Recupera/abortisce transazioni incomplete precedenti

while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));

    producer.beginTransaction();
    try {
        for (ConsumerRecord<String, String> record : records) {
            // Trasformazione e produzione
            producer.send(new ProducerRecord<>("output-topic", transform(record)));
        }
        // Offset committati atomicamente nella stessa transazione
        producer.sendOffsetsToTransaction(
            getOffsets(records),
            consumer.groupMetadata().groupId()
        );
        producer.commitTransaction();
    } catch (Exception e) {
        producer.abortTransaction();
    }
}
```

---

## Configurazioni chiave

### Producer

| Parametro | Default | Note |
|---|---|---|
| `enable.idempotence` | false | Richiede `acks=all`, `retries>1`, `max.inflight.requests.per.connection=1` |
| `transactional.id` | "" | Obbligatorio per transazioni; abilita `enable.idempotence` automaticamente |
| `transaction.timeout.ms` | 60000 (1 min) | Timeout oltre il quale il coordinator abortisce la transazione |

### Consumer

| Parametro | Default | Note |
|---|---|---|
| `isolation.level` | `read_uncommitted` | `read_committed` = vedi solo messaggi di transazioni committed |

### Broker

| Parametro | Default | Note |
|---|---|---|
| `transactional.id.timeout.ms` | 604800000 (7 giorni) | Dopo questo tempo senza heartbeat, il TransactionalId scade |
| `max.transaction.timeout.ms` | 900000 (15 min) | Limite massimo per `transaction.timeout.ms` del producer |
| `transaction.state.log.replication.factor` | 3 | Replication factor del topic interno `__transaction_state` |
| `transaction.state.log.num.partitions` | 50 | Partizioni del transaction log |
| `transaction.state.log.min.isr` | 2 | Min ISR per il transaction log |

---

## Architettura del Transaction Coordinator

### Componenti principali

| Componente | Ruolo |
|---|---|
| **Transaction Coordinator** | Entità broker-side che gestisce l'intero ciclo di vita delle transazioni; assegna PID; mantiene lo stato in un transaction log |
| **Transaction Log** (`__transaction_state`) | Topic interno, replicato, persistente; è lo stato store del coordinator |
| **Producer ID (PID)** | Identificatore univoco assegnato per sessione producer |
| **Producer Epoch** | Contatore incrementato ad ogni `initTransactions()`; permette fencing degli zombie |
| **TransactionalId** | Stringa stabile fornita dall'utente; collega sessioni diverse dello stesso producer logico |
| **Last Stable Offset (LSO)** | Massimo offset leggibile in `read_committed`; avanza solo dopo commit/abort di tutte le transazioni aperte fino a quel punto |
| **Control Messages** | Messaggi speciali scritti dai broker (COMMIT marker, ABORT marker) che indicano ai consumer la fine di una transazione |

### Flusso RPC per una transazione

```
Producer          Transaction Coordinator         Broker (Leader)       Consumer
   |                        |                          |                    |
   |-- InitPidRequest ----->|                          |                    |
   |<-- PID + Epoch --------|                          |                    |
   |                        |                          |                    |
   |-- beginTransaction() (client-side only)          |                    |
   |                        |                          |                    |
   |-- AddPartitionsToTxnReq ->|                      |                    |
   |<-- OK -----------------|                          |                    |
   |                        |                          |                    |
   |-- ProduceRequest (PID, Epoch, SeqNum) ----------->|                    |
   |<-- ACK --------------------------------------------|                    |
   |                        |                          |                    |
   |-- sendOffsetsToTransaction()                      |                    |
   |-- AddOffsetsToTxnReq ->|                          |                    |
   |-- TxnOffsetCommitReq ->|                          |                    |
   |                        |                          |                    |
   |-- EndTxnRequest (COMMIT) ->|                      |                    |
   |                        |-- PREPARE_COMMIT (log) ->|                    |
   |                        |-- WriteTxnMarkersReq --->|                    |
   |                        |                          |-- COMMIT marker -->|
   |                        |-- COMMITTED (log) ------>|                    |
   |<-- OK -----------------|                          |                    |
   |                        |                          | (LSO advances)     |
```

---

## Formato messaggi v2 (introdotto con KIP-98)

### MessageSet header (record batch level)

```
FirstOffset            => int64
Length                 => int32
PartitionLeaderEpoch   => int32
Magic                  => int8   (= 2, bumped)
CRC                    => int32  (CRC32C - più veloce via hardware)
Attributes             => int16
LastOffsetDelta        => int32  {NUOVO}
FirstTimestamp         => int64  {NUOVO}
MaxTimestamp           => int64  {NUOVO}
PID                    => int64  {NUOVO}
ProducerEpoch          => int16  {NUOVO}
FirstSequence          => int32  {NUOVO}
Messages               => [Message]
```

### Record (messaggio individuale)

```
Length         => varint  (compressione per offset piccoli)
Attributes     => int8
TimestampDelta => varint  (delta rispetto a FirstTimestamp)
OffsetDelta    => varint  (delta rispetto a FirstOffset)
KeyLen         => varint
Key            => data
ValueLen       => varint
Value          => data
Headers        => [Header]  {NUOVO}
```

### Attributi MessageSet (int16)

| Bits | Campo | Note |
|---|---|---|
| 0-2 | Compression | None/GZIP/Snappy/LZ4 |
| 3 | Timestamp Type | CreateTime / LogAppendTime |
| 4 | **Transactional flag** | Indica che seguiranno transaction markers |
| 5 | **Control flag** | Messaggio non consumabile dagli utenti (marker) |
| 6-15 | Unused | — |

### Control Messages (COMMIT / ABORT marker)

```
ControlMessageKey:
  Version             => int16
  ControlMessageType  => int16  (0=COMMIT, 1=ABORT)
```

### Efficienza del nuovo formato (confronto)

| N. messaggi | Formato vecchio | Formato v2 | Risparmio |
|---|---|---|---|
| 1 messaggio | 34 byte | 60 byte | −76% (overhead fisso batch) |
| 3 messaggi | 102 byte | 74 byte | +38% |
| 100 messaggi | 3.400 byte | 753 byte | +78% |

> Il break-even è ~2 messaggi per batch. Con batch normali (10-100+ messaggi) il formato v2 è nettamente più efficiente.

**Scelte di design:**
- CRC32C (non CRC32): supporto hardware nativo → più veloce
- CRC rimosso a livello di singolo record: non c'è garanzia end-to-end (il broker modifica timestamp/format), il CRC batch è sufficiente
- Header per record: campo aggiuntivo per propagare metadati (es. trace ID) senza toccare il value
- Offset delta come varint: batch con 100 messaggi → offset 0-99 → massimo 7 bit → 1 byte invece di 8

---

## Garanzie transazionali — limitazioni note

Il KIP riconosce esplicitamente i seguenti casi in cui le garanzie potrebbero non valere:

1. **Compaction**: un topic compacted può sovrascrivere messaggi transazionali, rimuovendo messaggi di un abort
2. **Log segment deletion**: se i segmenti iniziali vengono eliminati per retention, i messaggi early nella transazione vengono persi
3. **Consumer seek arbitrario**: un consumer che salta a un offset intermedio può leggere messaggi di una transazione senza il marker iniziale
4. **Partizioni non lette**: il consumer potrebbe non leggere tutte le partizioni partecipanti alla transazione

---

## Modello di autorizzazione

Nuovo resource type: `ProducerTransactionalId`

| Operazione | Permesso richiesto |
|---|---|
| InitPid, AddPartitionsToTxn, AddOffsetsToTxn, EndTxn | Write su `ProducerTransactionalId` |
| AddPartitionsToTxn | Write sul topic target |
| AddOffsetsToTxn | Read sul consumer group |
| Produce con PID transazionale | Write su `ProducerTransactionalId` |
| WriteTxnMarkers (inter-broker) | ClusterAction (solo broker-to-broker) |

---

## Link esterni

- [KIP-98 su Apache Kafka Wiki](https://cwiki.apache.org/confluence/display/KAFKA/KIP-98+-+Exactly+Once+Delivery+and+Transactional+Messaging) — documento originale
- [KAFKA-4815 (JIRA)](https://issues.apache.org/jira/browse/KAFKA-4815) — tracking issue
- [KIP-32 (message format v1)](https://cwiki.apache.org/confluence/display/KAFKA/KIP-32+-+Add+timestamps+to+Kafka+message) — precedente formato messaggi su cui si basa v2
- [Discussione mailing list](http://search-hadoop.com/m/Kafka/uyzND1jwZrr7HRHf?subj=+DISCUSS+KIP+98+Exactly+Once+Delivery+and+Transactional+Messaging) — thread originale

---

## Immagini

Nessuna immagine tecnica presente nel documento Confluence (KIP è spec testuale con schemi RPC).
