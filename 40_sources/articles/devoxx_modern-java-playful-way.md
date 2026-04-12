# Learning Modern Java the Playful Way

**Fonte:** [Devoxx Poland 2025 — Marit van Dijk, Piotr Przybył](https://read.readwise.io/new/read/01knxvtq7s1w83bk1h7rzx0xxz)  
**Data lettura:** 2026-04-11  
**Tipo:** Conference Talk Transcript (Video)  
**Tag:** #java #java24 #java25 #virtual-threads #structured-concurrency #stream-gatherers #vector-api #pattern-matching #intellij #elasticsearch #modern-java

---

## Contesto

Talk tenuto a **Devoxx Poland 2025** da:

- **Marit van Dijk** — Developer Advocate @ JetBrains, Java Champion
- **Piotr Przybył** — Developer Advocate @ Elastic, Java Champion

Il talk esplora in modo pratico e interattivo le funzionalità delle versioni recenti di Java (21–25), mostrando come aggiornare codice "legacy Java 8-style" con costrutti moderni. Il tutto attraverso una demo concreta: un'applicazione di ricerca immagini con Elasticsearch che utilizza hybrid search (vettoriale + testuale).

**Messaggio centrale:** ogni nuova versione di Java è più semplice, più veloce e migliore — e rimane gratuita. Non esiste una buona ragione per restare su Java 8.

---

## Feature Java Moderne Dimostrate

### 1. Compact Source Files & Instance Main Methods *(Java 25 / Preview in Java 24)*

Prima (Java 8+):
```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello World");
    }
}
```

Dopo (Java 25):
```java
void main() {
    println("Hello World");
}
```

- Rimuove il boilerplate di `public static void main`
- `println` e `readLine` disponibili come metodi statici (Simple IO)
- Java diventa meno verboso di linguaggi "più semplici" come Python per il classico Hello World
- Per abilitare in Java 24: impostare Language Level su **Preview** nelle Project Settings di IntelliJ

---

### 2. Module Imports *(Java 25)*

```java
// Prima: lunga lista di import individuali
import java.util.List;
import java.util.Map;
import java.io.IOException;
// ...

// Dopo:
import module java.base;
```

Importa automaticamente tutti i package esportati dal modulo `java.base`, riducendo drasticamente il numero di import espliciti.

---

### 3. Records *(Java 16 — stabile)*

```java
// Prima: Java Bean verboso con getter/setter/toString/equals
public class SearchResult { ... }

// Dopo:
record SearchResult(String fileName, float score) {}
```

- Immutable data carriers trasparenti
- Eliminano boilerplate; si può ancora fare override di `toString`, `equals`, ecc. se necessario
- Rendono il codice più conciso, leggibile e manutenibile

---

### 4. Stream Gatherers con `mapConcurrent` *(Java 24 — stabile)*

```java
// Prima: parallel stream (rischio di esaurire il common ForkJoinPool)
results = queries.parallelStream()
    .map(this::performSearch)
    .collect(toList());

// Dopo: usando Gatherers.mapConcurrent con virtual threads
results = queries.stream()
    .gather(Gatherers.mapConcurrent(5, this::performSearch))
    .collect(toList());
```

- `Gatherers` è la nuova API per **operazioni intermedie personalizzate** sugli stream
- `mapConcurrent` usa **virtual threads** internamente — sicuro e performante
- Evita i problemi di `parallelStream` (ForkJoinPool fisso, non virtual threads)
- Si può osservare nel debugger di IntelliJ IDEA (EAP) che i worker del ForkJoinPool portano virtual threads

> **Nota:** `parallelStream()` va evitato nel critical path in produzione — rischio di esaurire il common ForkJoinPool.

---

### 5. Structured Concurrency *(Preview da Java 21, revisionata in Java 25)*

```java
// Prima: CompletableFuture con gestione esplicita della cancellazione
CompletableFuture<List<Result>> vectorFuture = 
    CompletableFuture.supplyAsync(this::performVectorSearch);
CompletableFuture<List<Result>> classicFuture = 
    CompletableFuture.supplyAsync(this::performClassicSearch);

vectorFuture.exceptionally(e -> { classicFuture.cancel(true); return null; });
classicFuture.exceptionally(e -> { vectorFuture.cancel(true); return null; });
// ...con N future il problema scala esponenzialmente

// Dopo (Java 24):
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    var vectorSubtask = scope.fork(this::performVectorSearch);
    var classicSubtask = scope.fork(this::performClassicSearch);
    scope.join();
    return combineWithRRF(List.of(vectorSubtask.get(), classicSubtask.get()));
}

// Java 25 (variante configurabile):
try (var scope = StructuredTaskScope.open()) {
    // cancellazione automatica in caso di fallimento
    // supporto per timeout, thread factory custom, naming per debugger/JFR
}
```

**Vantaggi rispetto a `CompletableFuture`:**
- Cancellazione automatica reciproca tra subtask in caso di errore
- Pensiero sincrono per codice concorrente (simile ad async/await in JavaScript)
- Elimina thread leaks e cancellation delays
- Scala senza problemi con N task (nessun grafo di dipendenze esplicite)
- Leggibile: tutta la logica concorrente in poche righe

> *"Completable Future has no future."*  
> — **Dr. Vena** *(citato dai speaker)*

---

### 6. Markdown Javadoc *(Java 23+)*

```java
/// # Ricerca immagini
/// Esegue una **hybrid search** con RRF.
/// 
/// @param query termine di ricerca testuale
/// @return lista di risultati ordinati per rilevanza
public List<SearchResult> search(String query) { ... }
```

- La documentazione può essere scritta in **Markdown** invece che in HTML
- IntelliJ IDEA supporta il rendering del Markdown nei Javadoc
- AI tools (es. JetBrains AI Assistant) possono generare la documentazione automaticamente

---

### 7. Pattern Matching con Tipi Primitivi *(Preview in Java 23+)*

```java
// Prima: catena di if-else difficile da leggere
public double calculateDiscount(int quantity) {
    if (quantity >= 100) return 0.20;
    else if (quantity >= 50) return 0.15;
    else if (quantity >= 10) return 0.10;
    else return 0.0;
}

// Dopo: switch expression con guard e primitive types
public double calculateDiscount(int quantity) {
    return switch (quantity) {
        case int q when q >= 100 -> 0.20;
        case int q when q >= 50  -> 0.15;
        case int q when q >= 10  -> 0.10;
        default -> 0.0;
    };
}
```

- Più leggibile, più manutenibile
- Supporto per tipi primitivi in tutti i contesti di pattern matching
- Lo switch deve essere **esaustivo** (il compilatore lo verifica)

---

### 8. Vector API — Project Panama *(Incubator)*

Permette di sfruttare le istruzioni **SIMD** (Single Instruction Multiple Data) della CPU per operazioni vettoriali, rilevanti per embedding search e algebra lineare.

```java
// Approccio scalare (sequenziale)
for (int i = 0; i < a.length; i++) {
    result[i] = a[i] * b[i];
}

// Approccio vettoriale: processa 4 int (o più) per istruzione CPU
VectorSpecies<Integer> SPECIES = IntVector.SPECIES_PREFERRED;
for (int i = 0; i < a.length; i += SPECIES.length()) {
    var va = IntVector.fromArray(SPECIES, a, i);
    var vb = IntVector.fromArray(SPECIES, b, i);
    va.mul(vb).intoArray(result, i);
}
```

- Il codice vettoriale per il demo è stato **generato da JetBrains AI Assistant (Junior)**
- Con registri CPU a 128 bit e int a 32 bit: 4 operazioni per istruzione → ~2x speedup
- Non tutti i tipi di operazione traggono vantaggio (es. vector addition risulta più lenta in certi casi)
- Elasticsearch stesso usa la Vector API internamente per ESQL

> **Nota:** non era un benchmark rigoroso — solo una dimostrazione del potenziale. Eseguire benchmark propri prima di adottare in produzione.

---

## Demo: Image Search con Elasticsearch

L'applicazione dimostrativa implementa una **hybrid search** su immagini:

```
[Immagini] → [Multi-modal Encoder] → [Vector Embeddings] → [Elasticsearch]
                                                                    ↑
[Query testuale] → [Encoder] → [Vector] → [Vector Search] ─┐
                                          [Text Search]   ─┴→ RRF → Risultati
```

**Componenti:**
- **Multi-modal Encoder**: processa immagini e testo producendo vector embeddings
- **Elasticsearch**: indicizza file name + vector embeddings delle immagini
- **Hybrid Search**: combina ricerca testuale classica + ricerca vettoriale
- **RRF (Reciprocal Rank Fusion)**: algoritmo di fusione dei risultati, descritto in un paper di sole 2 pagine

**Evoluzione del codice durante il demo:**
1. Codice iniziale "Java 8 style" con classi verbose, parallel stream, CompletableFuture
2. Refactoring progressivo: record, module imports, stream gatherers, structured concurrency
3. Aggiornamento del client Elasticsearch alla nuova API builder-style

---

## Ecosistema e Dipendenze che Evolvono

- **Spring Boot** ha già adottato Java 17 come baseline
- **Maven 4** richiederà Java 17 come runtime (compilazione di progetti legacy ancora supportata)
- **Elasticsearch Java Client** ha rinnovato l'API → stile builder configurabile

> *"È come l'Agile: se qualcosa è doloroso, fallo più spesso e in piccoli pezzi. Aggiorna Java ad ogni versione, non aspettare anni."*

**Nota sulle LTS:**
> Se non si sta pagando per il supporto commerciale, non c'è motivo di restare solo sulle versioni LTS. La "S" sta per **Support** — se non si paga per quel supporto, si può adottare qualsiasi versione.

---

## Come Restare Aggiornati su Java

- **Conferenze**: Devoxx, JVM-related events
- **Java User Groups (JUG)**: locali o virtual (Virtual JUG disponibile online)
- **Blog Oracle "Inside Java"**
- **Blog IntelliJ IDEA**
- **JVM Bloggers** *(aggregatore settimanale per la community polacca, con contenuti anche in inglese)*

---

## Tool e Risorse Citate

| Tool/Risorsa | Descrizione |
|---|---|
| **IntelliJ IDEA EAP** | Early Access Program con feature preview (es. debug di virtual threads) |
| **JetBrains AI Assistant (Junior)** | Ha generato il codice Vector API durante il demo |
| **start.local** (Elastic) | Script per avviare Elasticsearch in locale con Docker in un singolo comando |
| **SDK Manager / SDKMAN** | Gestione versioni JDK; IntelliJ può scaricare anche versioni Early Access |

---

## Riferimenti Esterni

> **[Talk esterno]** **Adam** — *Virtual Threads* (Devoxx Poland 2025, stessa sala)  
> Consigliato dai speaker per approfondire virtual threads e il loro funzionamento interno.

> **[Talk esterno]** **Bala** — *Virtual Threads / Structured Concurrency* (Devoxx Poland 2025, ore 14:00, stessa sala)  
> Menzionato più volte durante il talk come complementare agli argomenti trattati.

> **[Talk esterno]** **Johan** — *Records in Java* (Devoxx Poland 2025, mercoledì)  
> Citato per un approfondimento sull'uso dei record.

> **[Paper]** *Reciprocal Rank Fusion (RRF)* — paper di 2 pagine che descrive l'algoritmo di fusione dei risultati di search da meccanismi diversi. Usato da Elasticsearch per il hybrid search.

> **[Community]** [JVM Bloggers](https://jvm-bloggers.com) — aggregatore settimanale di contenuti Java/JVM, prevalentemente polacco ma con articoli in inglese. Gestito da Software Mill.

> **[Citazione]** *"Completable Future has no future."* — **Dr. Vena** (fonte non specificata nel talk)

---

## Punti Chiave e Takeaway

- **Non restare su Java 8**: ogni versione porta feature reali, performance migliori e codice più leggibile
- **Records** riducono il boilerplate per i data holder (stabile da Java 16)
- **Stream Gatherers** con `mapConcurrent` sono la soluzione moderna a `parallelStream()` — usano virtual threads
- **Structured Concurrency** è superiore a `CompletableFuture` per task concorrenti: codice più leggibile, cancellazione automatica, nessun leak
- **Compact source files** abbassano la barriera all'ingresso di Java (utile per l'insegnamento)
- La **Vector API** (incubator) può dare speedup significativi per operazioni numeriche intensive
- L'AI (JetBrains Junior) è utile per scrivere codice boilerplate complesso come la Vector API
- Aggiornare le librerie (es. Elasticsearch client) porta spesso API più pulite oltre ai benefici tecnici

---

## Analisi e Riflessioni

### Punti di forza del talk:
- Approccio pratico con una demo realistica (image search con Elasticsearch), non esempi artificiali
- Progressione chiara: mostra il "prima" e il "dopo" per ogni feature
- Copertura equilibrata tra novità sintattiche (compact files) e novità concettuali (structured concurrency)

### Connessioni con altri concetti della knowledge base:
- **Virtual threads** e **Structured Concurrency** si collegano direttamente ai corsi *Java Threads Basics*, *Java Threads Demystified* e *Concurrent Programming Core Concepts*
- **Stream Gatherers** estende i concetti del corso *Java Next Steps: Streams & Collectors*
- La **hybrid search con RRF** e l'uso di Elasticsearch in Java è rilevante per progetti che combinano Java ecosystem e search/AI
- Il pattern di **configurazione tramite lambda builder** (es. `StructuredTaskScope.open()`) è coerente con il trend Java di API fluent/configurabili

### Domande aperte:
- Quando `Structured Concurrency` uscirà dallo stato preview? (Previsto stabilizzarsi in Java 25?)
- La **Vector API** uscirà dall'incubator in Java 25?
- Quali sono i limiti pratici di `mapConcurrent` con molti task in parallelo?
- Come si comporta `Structured Concurrency` con timeout + cancellazione parziale?
