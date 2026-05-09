---
tags:
  - java
  - performance
  - concurrency
  - distributed-systems
type: article
author: Daniel Shaya
source: https://www.youtube.com/watch?v=BD9cRbxWQx8
date: 2026-05-05
---

# How Low Can You Go? Ultra Low Latency Java in the Real World

**Speaker:** Daniel Shaya (low latency Java consultant, ex-Chronicle Software)  
**Evento:** London Java Community (LJC)  
**Durata:** 55:31  
**Lingua originale:** inglese

---

## Sintesi

Daniel Shaya risponde a tre domande fondamentali per chi lavora in sistemi ultra-low latency (< 100 microsecondi): Java è una scelta appropriata? Come si sviluppa correttamente in questo contesto? I microservizi hanno senso?

Il punto di partenza è il contesto della finanza ad alta frequenza — dove 3 millisecondi di vantaggio costano $300 milioni di infrastruttura (caso Spread Networks, Chicago-New York 2010), dove un GC pause può annullare quell'investimento, e dove ogni spike di latenza significa perdite in arbitraggio. Il talk è costruito sull'idea che il low-latency programming richiede un **approccio scientifico**: ipotesi → esperimento controllato → conclusione. Non è possibile ragionare a priori su cosa sarà più veloce — la distanza tra il codice scritto e ciò che esegue l'hardware è troppo grande.

La risposta sulla scelta del linguaggio è sfumata: **Java è ragionevole per target > 10 microsecondi**, ma sotto quella soglia non bisogna nemmeno iniziare. C/C++ è nella stessa fascia di Java con buoni sviluppatori. FPGA scende di un ordine di grandezza (< 1 microsecondo). ASIC arriva a 400 nanosecondi. Python è fuori gara.

La sezione più densa riguarda le tecniche di sviluppo: **zero garbage creation** come regola assoluta per target sub-millisecondo, design scientifico dei test harness, misura edge-to-edge, correzione per la *coordinated omission*, e documentazione precisa dei requisiti di latenza (percentili specifici, hardware specifico, throughput specifico).

Il finale sorprende: **i microservizi low-latency sono possibili** — e non solo possibili, ma spesso superiori — se si usano le giuste primitive: shared memory (non TCP), thread singolo per servizio, CPU pinning, NUMA awareness, e registrazione completa di input/output per replay in produzione.

---

## I tre benchmark di linguaggio (DMA application)

Sistema di riferimento: client → FIX parser → OMS (risk checks + market data) → FIX engine → exchange. Misura: tempo dal packet in entrata al packet in uscita.

| Linguaggio/Hardware | Latenza minima (p99) | Note |
|---|---|---|
| **Java** (ottimizzato) | ~10 μs | Con tecniche specifiche; GC è il nemico |
| **C/C++** | ~10 μs | Simile a Java con buoni sviluppatori; più produttivo = Java |
| **FPGA** | < 1 μs | Compile time 8h, costoso, ma ubiquo nelle banche |
| **ASIC** | ~400 ns | Custom chip, costo enorme (giustificato > 400k unità) |
| **Python** | Non competitivo | Fuori dalla gara |

> Esperimento Aeron (Martin Thompson, Pete Lawrey, et al.): stesso algoritmo portato in Java, C++, C#, Go. Dopo ottimizzazioni: **C# batte Java** perché permette di controllare il layout della memoria (struct). Java batte tutti all'inizio grazie alla produttività. Conclusione: il tempo speso a ottimizzare il linguaggio toglie tempo agli algoritmi, affidabilità, e test.

---

## Punti chiave

- **Java è ragionevole sopra i 10 μs**: sotto quella soglia, le limitazioni strutturali (GC, memory layout non controllabile, startup JIT, istruzioni CPU non accessibili) rendono il target quasi irraggiungibile.

- **Garbage collection = nemico assoluto per sub-millisecondo**: regola: **nessun GC**. Non si possono avere pause GC in un sistema dove ogni millisecondo costa. La warm-up del JVM (prime 10.000 iterazioni) va pianificata senza impatto sulla produzione.

- **Zero allocation API design**: non forzare chi usa l'API ad allocare. Esempi:
  - `CharSequence` invece di `String` (Chronicle Map, kolobok)
  - `getUsing(reusableObject)` invece di `get()` che crea un nuovo oggetto a ogni chiamata
  - Object pooling single-threaded (senza sincronizzazione se il servizio è single-threaded)

- **Approccio scientifico obbligatorio**: non si può ottimizzare a occhio. Le CPU moderne (4 GHz, pipeline, branch prediction) rendono impossibile ragionare a priori. Esempio reale di Shaya: cache di valori calcolati → risultato 3× **più lento** perché il cache miss in L3/RAM costa ordini di grandezza più del ricalcolo.

- **Coordinated Omission — misurare correttamente**: misurare da `System.nanoTime()` al momento dell'invio (non all'arrivo della risposta) porta a sottostimare la latenza. La metafora giusta è il treno: la latenza si misura dall'orario **previsto** di partenza, non dall'orario effettivo. Uno spike causa un effetto valanga sulle misurazioni successive. Tool: HDR Histogram di Gil Tene (Azul Systems).

- **Documentazione precisa dei requisiti**: "i messaggi devono prendere 100 microsecondi" non è un requisito. Un contratto di latenza reale specifica: attività + sistema di riferimento + throughput esatto + durata del test + burst + hardware specifico (CPU clock, NIC, OS config) + percentile target + correzione coordinated omission sì/no + metodo di timing (Corvil device, packet capture, HDR histogram).

- **Categorie di real-time system**: Hard RT (pacemaker, weapons: il worst case non può mai superare soglia), First-to-bell (cycling team: conta solo il più veloce, gli altri possono rallentare), Web RT (l'utente non deve accorgersi), Soft RT (la maggioranza dei sistemi finanziari: tutti i percentili contano, ma uno spike non è catastrofico).

- **Microservizi low-latency: sì, se si usa shared memory**: Chronicle Queue (round-trip echo test: 500 ns) e Aeron (250 ns round-trip, 100 ns singolo trip) rendono il costo di messaging tra microservizi trascurabile anche in sistemi sub-10 μs. TCP è escluso.

- **Single-threaded microservices**: eliminano la sincronizzazione. `synchronized` non compare nel codice. Object pooling è triviale (no contesa). Scaling = partizione dei dati + più microservizi. Vantaggio: CPU pinning a un core dedicato, spinning attivo (busy-wait) senza interrupt.

- **NUMA awareness e CPU pinning**: i microservizi non devono attraversare boundary NUMA. Mappare esplicitamente quale microservizio va su quale CPU. Bloccare gli interrupt su quel core.

- **Record tutto (input + output)**: ogni messaggio in ingresso e in uscita dal microservizio va registrato in shared memory (non su disco con I/O lenta). Permette replay in produzione e riproduzione del raro 1-in-10.000 spike in ambiente di test.

---

## Citazioni notevoli

> "A GC pause can easily last over three milliseconds. Imagine someone from your organization has just shelled out $150,000 a month [for the Spread Networks cable] and you've got latency spikes of three milliseconds completely negating that."  
> — Daniel Shaya

> "Operations on the CPU are almost free compared with memory access. Even a relatively expensive computation can be faster to run than fetching data from L3 cache — let alone main memory."  
> — Daniel Shaya (sul suo errore di caching che rendeva il sistema 3× più lento)

> "Low latency requires a scientific approach: hypothesis, experiment, conclusion. You can't just reason about it — there's an awfully long way between when you write a line of code and when it gets executed by the hardware."  
> — Daniel Shaya

> "Go back to your small JVM. Use single-threaded microservices. The word 'synchronized' doesn't appear anywhere in my code."  
> — Daniel Shaya (contro il trend delle JVM da 256GB con milioni di thread)

> "The cost of sending messages via shared memory between microservices is negligible — 100 nanoseconds with Aeron — even for sub-10 microsecond systems."  
> — Daniel Shaya

---

## Tipologie di latenza target e approccio (da Jack Shirazi)

| Target latenza | Approccio chiave |
|---|---|
| Secondi | Standard Java, GC accettabile |
| Centinaia di ms | GC tuning, evitare allocazioni critiche |
| Decine di ms | GC ridotto, test di throughput |
| **< 1 ms (sub-millisecondo)** | **Zero GC, zero allocation, scientifico** |

---

## Riferimenti e risorse

- **Chronicle Software** ([chronicle.software](https://chronicle.software)) — Chronicle Map, Chronicle Queue, Chronicle Fix; framework low-latency open source
- **Aeron** ([github.com/real-logic/aeron](https://github.com/real-logic/aeron)) — messaging ultra-low latency; 250ns round-trip, 100ns singolo trip
- **kolobok** (Roman Leventov) — libreria Java per mappe con `withKeyEquivalence` su `CharSequence`, evita allocazione di String nei lookup
- **HDR Histogram** (Gil Tene / Azul Systems) — tool per misurare percentili con correzione coordinated omission
- **Corvil devices** — strumenti hardware di packet capture e timing per misura edge-to-edge
- **Jack Shirazi** — [Java Performance Tuning Newsletter](https://www.javaperformancetuning.com) — newsletter di riferimento sulla performance JVM
- **Martin Thompson** — co-autore di Aeron, esperto di mechanical sympathy e low-latency Java
- **Spread Networks** — caso studio: cavo Chicago-NY, 827 miglia, $300M, 3ms di vantaggio, $150k/mese
- **"The Seven Habits of Highly Effective People"** — citato per il principio "lavora nel tuo cerchio di influenza": benchmarka prima il tuo framework null, poi il SO, poi la tua app
