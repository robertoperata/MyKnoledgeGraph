---
tags:
  - java
  - performance
  - concurrency
  - distributed-systems
type: article
author: Ivan Zvieriev (Epam, 7+ anni in HFT)
source: https://www.youtube.com/watch?v=TaKU2Xh52GA
date: 2023-11-01
---

# Java Compilers: Why Java is One of the First Options for Low Latency Apps

**Speaker:** Ivan Zvieriev (senior low-latency developer, Epam)  
**Evento:** Java ON 2023  
**Durata:** 1h 00m 43s  
**Lingua originale:** inglese

> Nota: l'autore fa esplicito riferimento al talk di Daniel Shaya "How Low Can You Go?" come approfondimento sui dettagli di latenza. I due talk sono complementari.

---

## Sintesi

Ivan Zvieriev dimostra perché Java è oggi **una delle prime scelte** per sistemi low-latency, invertendo il pregiudizio storico che lo vedeva come linguaggio "enterprise ma lento". Il talk ha tre sezioni: architettura di un sistema HFT (High Frequency Trading), strumenti e tecniche Java per ogni aspetto critico (transport, memoria, concorrenza), e infine il cuore del talk — il **pipeline di compilazione JVM** e perché può battere C++ compilato staticamente.

Il punto controintuitivo centrale: un'applicazione Java completamente riscaldata può essere **più veloce del codice C++ compilato ahead-of-time**, perché la JVM raccoglie informazioni di profiling a runtime (frequenza di chiamata dei metodi, rami if/else più frequenti, escape analysis dinamico) e può ricompilare il codice con ottimizzazioni specifiche per il workload reale — cosa impossibile per un compilatore statico.

La sezione pratica copre: come rendere UDP affidabile in HFT (sequencer + rewinder pattern), perché zero GC è un requisito non negoziabile (full GC = production incident), come i compilatori JVM (javac → C1 → C2) ottimizzano il codice in modo progressivo, e perché i framework reactive "di mercato" (Akka, etc.) non si usano in HFT — si scrivono soluzioni custom per mantenere pieno controllo del garbage.

---

## Domini che richiedono low-latency

| Dominio | Esempio |
|---|---|
| **Financial services / HFT** | Ordine aperto e chiuso entro 1 secondo, tutto algoritmico |
| **Gaming** | Latenza = perdere la partita (lag) |
| **Telecom** | Qualità audio/video in tempo reale |
| **Automotive / Military / Aviation** | Autopilota Tesla che reagisce a frenata improvvisa |
| **High Performance Computing** | Esplorazione spaziale, computazione scientifica massiccia |

---

## Architettura di un sistema HFT

```
Exchange (dati di mercato)
    │
    ▼ Market data feed
Market Data Receiver  ←── Collocated (fisica vicina ai server exchange)
    │
    ▼
Strategie algoritmiche (N strategie parallele)
    │ trigger ordine
    ▼
Order Management System (OMS)
    │ ordine validato
    ▼
Exchange (esecuzione) ←── stesso path fisico
    │
Trade Monitoring Backend (logging, monitoring, stabilità)
```

**Super-fast path alternativo**: per le strategie più veloci, un canale dedicato che bypassa gli strati più lenti e invia l'ordine all'exchange quasi direttamente.

---

## Requisiti tecnici per sistemi HFT

| Requisito | Chi lo gestisce | Dettaglio |
|---|---|---|
| **Fast hardware** | Azienda (non developer) | Server overcloccati, NIC specializzate |
| **Proximity / Colocation** | Azienda | Server fisicamente vicini all'exchange; per legge USA stessa lunghezza di fibra per tutti |
| **Security perimeter** | Architettura | Nessuna crittografia/token check su ogni request — security all'esterno, mai nel path critico |
| **Code efficiency** | Developer | Algoritmi ottimali, strutture dati cache-friendly |
| **Fast transport** | Developer + Framework | UDP + sequencer/rewinder; LMAX Disruptor, Aeron |
| **Effective memory** | Developer | Zero GC, heap sizing fisso, GC specializzati |
| **Effective concurrency** | Developer | Single-thread by design, thread affinity |
| **Logging & persistence** | Architettura | Async in thread separati, o delega a microservizio separato via UDP bus |

---

## Fast Transport: UDP vs TCP in HFT

### Perché UDP è usato in HFT nonostante non sia "reliable"

UDP non è reliable perché non aspetta conferma di ricezione — se il consumer non legge abbastanza velocemente, i pacchetti vengono persi. La soluzione in HFT: **il consumer legge almeno alla stessa velocità del producer** (spesso molto più veloce). In pratica, sistemi in produzione hanno operato settimane senza un singolo packet loss UDP.

### Pattern: Sequencer + Rewinder

Ogni datagram UDP ha un **sequence number**. Il consumer verifica la sequenza attesa:
- Se riceve il numero atteso → OK
- Se il numero arriva dal "secondo bus" (ridondanza) → OK
- Se manca la sequenza (es. attende 13, arriva 17) → **Rewinder**: richiede i datagram da 13 a 17, continua a bufferizzare il flusso normale nel frattempo

### Soluzioni framework per fast transport in Java

| Framework | Tipo | Note |
|---|---|---|
| **LMAX Disruptor** | Open source | Progettato per HFT; include strutture dati e framework concorrenza |
| **Project Aeron** | Open source | Ultra-low latency messaging |
| **TIBCO FTL** | Commerciale | Soluzioni fast queue TIBCO |
| **kdb+** | Commercial | In-memory storage + transport capabilities |
| **Noto** | Nuovo | Dichiarano alta velocità (non verificato dall'autore) |

---

## Effective Memory: Zero Garbage Collection

### Perché GC = production incident in HFT

Un full GC con stop-the-world anche di mezzo secondo = incident. In uno dei progetti dell'autore, **il GC log era monitorato in produzione** e ogni trigger era un incident con alert.

### Strategie per evitare GC

1. **Zero allocation code**: non creare oggetti dove non necessario (riutilizzo, object pooling)
2. **Heap sizing fisso**: `Xms = Xmx` — la JVM non ridimensiona mai l'heap. Motivo: il ridimensionamento invalida le CPU cache (L1/L2/L3) e causa spike di latenza
3. **Garbage collector specializzati**:
   - **ZGC** (Java 15+, generazionale da Java 21): dichiara pause "quasi zero" (tecnicamente microscopiche ma trascurabili)
   - **Shenandoah**: simile a ZGC, in alcuni JDK sperimentale
   - **Epsilon GC**: non raccoglie garbage — alloca finché può, poi crasha. Perfetto se la generazione di garbage è zero by design
   - **Azul C4** (JDK commerciale Azul): garbage collector dedicato a basse pause

### CPU Cache e data locality

| Cache | Latenza accesso tipica |
|---|---|
| L1 cache | 1–3 ns |
| L3 cache | 10–30 ns |
| DDR4 RAM (migliore) | 50–100 ns |

Differenza di 2 ordini di grandezza tra L1 e RAM. Il caching miss è costoso.

**Strutture dati cache-friendly** (accesso sequenziale in memoria):
- ✅ Array
- ✅ Ring buffer (usato da LMAX Disruptor)
- ❌ LinkedList (accesso non sequenziale → cache miss)
- ❌ HashMap standard

**Data locality**: iterare su un array da 0 a N è cache-friendly. Saltare elementi casuali → cache miss.

**Thread Affinity**: un thread iniziato su un core dovrebbe continuare su quel core per non perdere la cache locale del processore. Difficile da implementare, ma critico.

### Warm-up: non solo compilazione

Il warm-up non serve solo a compilare i metodi (JIT) — serve anche a **popolare le CPU cache**. Un'applicazione completamente riscaldata legge dai cache invece che dalla RAM.

---

## Concorrenza: Single-Thread by Design

Aneddoto diretto dell'autore: in un colloquio di 60 minuti, 40 erano dedicati a concorrenza avanzata. Il primo giorno di lavoro il senior developer mostrò il sistema: **completamente single-thread by design**.

> "On interview you should understand these concepts, but in fact concurrency must be very smart. Most probably you will not be able to avoid it — but only for logging and persistence."

**Approccio**: logica di business su thread singolo → logging/persistence su thread separato asincrono → o meglio ancora, un microservizio dedicato riceve i dati via UDP bus e li persiste.

---

## Pipeline di compilazione Java (il cuore del talk)

### Livelli di compilazione (tiered compilation)

```
Codice Java sorgente
    │
    ▼ javac
Bytecode (.class)  ← già ottimizzato (constant folding, method inlining, loop unrolling...)
    │
    ▼ Tier 0: Interpretazione
JVM interpreta il bytecode (esecuzione lenta, raccolta profiling iniziale)
    │
    ▼ Tier 1-3: C1 Compiler (dopo ~1500 invocazioni del metodo)
Compilazione veloce senza profiling completo
    │  (metodi triviali si fermano qui)
    ▼ Tier 4: C2 Compiler (fully profiled)
Compilazione ottimizzata con tutte le informazioni di profiling
```

### Ottimizzazioni già a livello javac

| Ottimizzazione | Descrizione |
|---|---|
| Constant folding | Sostituisce `2 * 3` con `6` a compile time |
| Dead code elimination | Rimuove codice mai raggiungibile |
| Local variable type inference | `var` risolto staticamente |
| Method inlining | Inlina il corpo di metodi piccoli (evita invocation overhead) |
| Constant pool optimizations | String pool, int pool e altri pool interni |
| Control flow optimization | Loop unrolling, semplificazione branch, riduzione istruzioni branch |
| String concatenation optimization | Riduce allocazioni per operazioni `+` su stringhe |
| Tail call optimization | (solo su alcune JVM, non standard HotSpot) |

### Informazioni di profiling raccolte dalla JVM a runtime

| Tipo di profiling | Uso |
|---|---|
| Method profiling | Frequenza di invocazione → decide se promuovere a C2 |
| Branch profiling | Quale ramo if/else è più frequente → riordina il codice macchina |
| Loop profiling | Ottimizzazione dei loop più frequenti |
| Tiered compilation state | Cosa è già compilato, cosa ricompilare |
| **Escape analysis** | Se un oggetto "esce" dal metodo → se non esce, può essere allocato sullo stack invece che sull'heap |
| Type profiling | Tipo runtime degli oggetti → ottimizzazione dei virtual dispatch |

### Ottimizzazioni C1/C2 (basate sul profiling)

- **Loop splitting/fusion**: spezza loop grandi o unisce loop simili contigui
- **Branch prediction**: riordina il codice macchina mettendo il ramo più frequente prima → meno cache miss di istruzioni
- **Dead code elimination avanzata**: basata su statistiche runtime
- **Escape analysis → stack allocation**: oggetti che non escono dal metodo vengono allocati sullo stack, eliminando completamente il GC overhead per quegli oggetti
- **Null check elimination**: se l'analisi statuta dimostra che un oggetto non può essere null, elimina il check
- **Lock elimination**: se l'escape analysis dimostra che un oggetto sincronizzato non è condiviso, elimina il lock

### Perché Java può battere C++

C e C++ compilano **una volta sola** prima dell'esecuzione: il compilatore non ha informazioni su come il programma verrà effettivamente usato. Java compila **dinamicamente** con informazioni reali sul workload:

- Sa che il ramo `if (x > 0)` è vero il 99.9% delle volte → ottimizza per quel caso
- Sa che un oggetto non esce mai dal metodo → lo alloca sullo stack
- Sa che un lock non è mai conteso → lo elimina

Un'applicazione Java completamente riscaldata può superare C++ in scenari ad alta riuso di pattern.

---

## Concorrenza e Logging

**Concorrenza**: deve essere "very smart". Più thread → più contesa su risorse condivise → latenza non prevedibile. La regola: **single-thread per la logica critica**, thread separati solo per I/O e logging.

**Framework reactive**: non si usano in HFT framework come Akka, WebFlux, etc. Motivo: non si sa quanto garbage creano internamente. Si usano solo:
- LMAX Disruptor e Project Aeron → progettati per zero garbage
- O soluzioni custom scritte da zero (con JUnit come unica dipendenza)

---

## Citazioni notevoli

> "Full garbage collection was a production incident. We were monitoring GC logs and we definitely knew that GC shouldn't trigger. If it did, it was an incident."  
> — Ivan Zvieriev (sull'approccio al GC in HFT reale)

> "On the first day he showed me the system and said 'all our system is single-thread by design.' I was like — why? What was the sense of 40 minutes of concurrency questions?"  
> — Ivan Zvieriev (aneddoto sul colloquio vs realtà del sistema)

> "JVM is watching. It's collecting data during execution and it can use this data for profiling. This is something that you cannot do with ahead-of-time compilation."  
> — Ivan Zvieriev (sul vantaggio del JIT rispetto a C++)

> "Reactive frameworks are good and useful, but most probably they will be self-made. I doubt someone will take Akka because what's going on inside — we don't know and we cannot predict."  
> — Ivan Zvieriev (nella Q&A su framework reactive in HFT)

---

## Riferimenti e risorse

- **Daniel Shaya** — talk "How Low Can You Go? Ultra Low Latency Java in Real World" — esplicitamente citato come approfondimento sui dettagli di latenza (analogo a questo talk)
- **LMAX Disruptor** ([github.com/LMAX-Exchange/disruptor](https://github.com/LMAX-Exchange/disruptor)) — ring buffer lock-free per Java, progettato per HFT, zero garbage
- **Project Aeron** ([github.com/real-logic/aeron](https://github.com/real-logic/aeron)) — transport ultra-low latency, zero garbage
- **kdb+** — database in-memory + transport per HFT
- **Alex Shipilёv (shipilev.net)** — autore di Shenandoah GC, contributore principale OpenJDK, autore di JMH; blog estremamente tecnico su JVM internals
- **Jack Shirazi** — [Java Performance Tuning Newsletter](https://www.javaperformancetuning.com) — newsletter dal 2001 su performance JVM
- **Epsilon GC** — garbage collector "no-op" di Java: non raccoglie nulla, crasha quando l'heap è pieno — perfetto se il codice è zero-garbage by design
- **ZGC** (Java 21 generazionale) — garbage collector con pause "quasi zero" (<1ms tipicamente)
- **Azul C4** — garbage collector commerciale di Azul JDK per basse pause
- **Ivan Zvieriev su Twitch** — occasionalmente fa streaming su topic low-level Java
