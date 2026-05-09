---
tags:
  - java
  - performance
  - concurrency
type: article
author: Ario de Montreal
source: https://www.youtube.com/watch?v=iUT8JRnQZbs
date: 2026-01-28
---

# Java 25 (LTS): Virtually the Best Version Ever

**Speaker:** Ario de Montreal (Managing Director & Head of TS Imagine Canada; EasyMock lead; Java Champion)  
**Evento:** JChampions Conference  
**Durata:** 1:03:54  
**Lingua originale:** inglese

---

## Sintesi

Panoramica completa delle funzionalità di Java 25, la nuova versione LTS dopo Java 21. Il talk copre tutto ciò che è arrivato tra Java 22 e 25: feature diventate finali da preview, miglioramenti JVM, nuovi strumenti, feature di linguaggio, e novità ancora in preview.

Il titolo è un gioco di parole su **Virtual Threads** — la feature più importante introdotta in Java 21 che ora, con Java 24 che ha risolto il problema del *pinning* su `synchronized`, è finalmente affidabile in produzione. Tra le novità più impattanti: **Compact Object Headers** (~25% riduzione heap), **AOT** (Ahead of Time compilation) senza step separato di profiling, e i nuovi **Stream Gatherers** per operazioni intermedie personalizzate.

Il talk comprende live coding esteso su: unnamed patterns (`_`), compact source files, scope values, JFR method tracing, AOT demo, flexible constructor bodies, stream gatherers, multi-source file programs, e Markdown in JavaDoc.

---

## Virtual Threads: il problema risolto in Java 24

**Pinning** = condizione in cui un virtual thread non può essere smontato dalla CPU anche quando bloccato:

| Causa | Stato in Java 25 | Note |
|---|---|---|
| `synchronized` block/method | ✅ **Risolto in Java 24** | Il thread ora viene smontato correttamente |
| Native method call (JNI) | ❌ Non risolvibile | Quando si è in codice nativo non c'è modo di rilevare mutex; rimane pinned |

Il pinning su `synchronized` era il blocco principale all'adozione in produzione. Con Java 24 i virtual threads sono ora **affidabili**.

---

## Feature diventate finali (da preview)

### Unnamed Patterns and Variables (`_`) — Java 22

Il carattere `_` (underscore) è diventato keyword in Java 9; con Java 22 diventa il marcatore ufficiale per "unused":

```java
// Prima: parametri inutilizzati richiedevano nomi inventati
catch (Exception e) { ... }
// Ora: esplicito e compilatore-enforced
catch (Exception _) { ... }

// Destrutturazione di record con campi non usati
if (shape instanceof Circle(_, double radius)) { ... }
```

Specialmente utile con il pattern matching: quando si destruttura un record, molti campi possono essere irrilevanti per un dato ramo.

### Foreign Function and Memory API (FFM) — Java 22

API ufficiale per:
- Accesso a memoria off-heap
- Chiamate a funzioni native (sostituzione di JNI)
- Significativamente più veloce del vecchio JNI

### Compact Source Files e Instance Main Method — Java 25

Rimozione del boilerplate per chi inizia o fa scripting:

```java
// Prima (Java classico)
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello");
    }
}

// Ora (Java 25 compact source file)
void main() {
    IO.println("Hello");  // IO è auto-importato
}
```

- Il nome della classe viene dedotto dal nome del file
- `main()` può essere un metodo di istanza (non più necessariamente `static`)
- Tutto il modulo `java.base` è importato automaticamente
- `IO.println()` sostituisce `System.out.println()` nel contesto compact
- Si può passare direttamente `java MyFile.java` alla JVM (compila + esegue)

### Scope Values — Java 25

Sostituzione di `ThreadLocal` per il contesto condiviso immutabile tra thread:

```java
// ThreadLocal: non funziona nativamente tra thread diversi
static final ThreadLocal<Credentials> CREDS = new ThreadLocal<>();
// Bisogna copiare manualmente tra thread

// ScopeValue: si propaga automaticamente nelle strutture strutturate
static final ScopeValue<Credentials> CREDS = ScopeValue.newInstance();

ScopeValue.where(CREDS, credentials).run(() -> {
    // CREDS.get() disponibile qui e nei subtask strutturati
    fetchData();
});
```

Vantaggi rispetto a `ThreadLocal`:
- Meno memoria
- Non rimane bloccato nel thread (evita memory leak classico dei ThreadLocal)
- Si propaga automaticamente con Structured Concurrency (quando sarà finale)

**Nota:** Structured Task Scope è ancora preview in Java 25; la propagazione automatica di ScopeValues richiederà di aspettare Java 29.

---

## Miglioramenti JVM e performance

### Compact Object Headers ⭐

Riduzione della rappresentazione in memoria degli oggetti nel heap:
- Risparmio stimato: **~25% del heap** (dipende dalla dimensione degli oggetti)
- Non richiede cambi al codice applicativo
- Potenzialmente la ragione da sola per migrare a Java 25

> "Finding a way to remove 25% of the heap after 25 years is amazingly incredible. Given the price of RAM right now, it's a lot of money." — Ario

### Generational ZGC e Shenandoah

- **ZGC**: modalità generazionale ora così superiore che la modalità non-generazionale è stata **rimossa**
- **Shenandoah**: modalità generazionale disponibile
- **G1**: rimane il default con miglioramenti continui (late barrier expansion, region pinning)

### AOT (Ahead of Time) — senza step separato

L'ottimizzazione JIT può essere **cachata e riutilizzata** all'avvio successivo:

```bash
# Primo avvio: crea il profilo JIT
java -XX:AOTMode=record -XX:AOTConfiguration=app.aotconf MyApp

# Avvio successivo: usa il profilo (quasi 3× più veloce)
java -XX:AOTConfiguration=app.aotconf MyApp
```

In Java 25 non è più necessario il step separato di recording — si può fare tutto in un'unica esecuzione. Il JVM usa le ottimizzazioni precedenti come baseline; se l'applicazione cambia, si deottimizza e riottimizza automaticamente.

**Limitazioni attuali:** il profilo AOT è platform-specific (CPU); l'integrazione con Maven/CI/CD non è ancora completamente definita.

---

## Java Flight Recorder (JFR) — miglioramenti

JFR è il profiler a basso overhead già incluso nel JDK. In Java 25 tre nuove capacità:

| Feature | Descrizione | Limite |
|---|---|---|
| **Cooperative sampling** | Campionamento mentre l'app è in esecuzione (no stop al safepoint) | Experimental |
| **CPU profiling asincrono** | Profiling CPU accurato come async-profiler | Solo Linux |
| **Method tracing** | Stack trace di ogni invocazione di un metodo specifico | — |

```bash
# Start JFR profiling
jcmd <pid> JFR.start name=myProfile settings=profile

# Dump
jcmd <pid> JFR.dump name=myProfile filename=app.jfr

# Stop
jcmd <pid> JFR.stop name=myProfile

# Analisi da command line
jfr print --events jdk.MethodInvocation app.jfr
```

`jcmd` sostituisce definitivamente `jmap`, `jstat`, `jhat`, e altri tool legacy.

---

## Nuove feature di linguaggio

### Flexible Constructor Bodies — Java 25

Prima di Java 25, nessun codice poteva precedere `this()` o `super()`:

```java
// Prima: impossibile — si doveva estrarre un metodo statico
class Circle extends Shape {
    Circle(int x, int y, int radius) {
        super(validate(x), validate(y)); // solo chiamate statiche
    }
    private static int validate(int v) { ... }
}

// Ora: codice normale può precedere super()
class Circle extends Shape {
    Circle(int x, int y, int radius) {
        if (radius <= 0) throw new IllegalArgumentException();  // ✅
        super(x, y);
        this.radius = radius;
    }
}
```

**Limitazioni:** non si possono ancora accedere/scrivere attributi della superclasse prima di `super()` — le restrizioni ovvie di correttezza rimangono.

### Module Import Declarations — Java 25

```java
// Prima: importazione package per package
import java.net.http.HttpClient;
import java.net.http.HttpRequest;

// Ora: importazione dell'intero modulo
import module java.net.http;
```

Controverso: il presenter è esplicitamente contro (come per le star import — oscura le dipendenze esplicite). Utile per scripting e prototipazione.

### Stream Gatherers — Java 25

Operazioni intermedie **personalizzate** per la Stream API:

```java
// Creare un Gatherer personalizzato (API di libreria)
Gatherer<User, ?, String> catAge = Gatherer.of(
    () -> new ArrayList<>(),  // initializer
    (state, user, downstream) -> {
        downstream.push(user.name() + ":" + user.age());
        return true;  // continue stream
    },
    (left, right) -> { left.addAll(right); return left; }, // combiner (parallel)
    (state, downstream) -> {}  // finisher
);

// Uso come qualsiasi altra operazione intermedia
users.stream()
     .gather(catAge)
     .toList();
```

I Gatherers sono pensati per **library developers**, non per uso diretto quotidiano. Supportano sia modalità sequenziale che parallela (alcuni non sono parallelizzabili per natura).

### Multi-Source File Programs — Java 25

```bash
# Prima: solo un file alla volta
java Main.java  # Helper.java non veniva incluso

# Ora: tutte le dipendenze nella stessa directory vengono compilate
java Main.java  # compila automaticamente anche Helper.java
```

Utile per scripting con classi helper senza setup di progetto Maven/Gradle.

### Markdown in JavaDoc (`///`) — Java 25

```java
/// # Titolo della classe
///
/// Descrizione in **markdown** normale.
///
/// ```java
/// MyClass obj = new MyClass();
/// ```
///
/// - Lista di punti
/// - Altro punto
public class MyClass { ... }
```

Tre slash invece di `/** */`. Nessun tag HTML, nessun `<p>`, empty lines funzionano naturalmente.

---

## Feature ancora in preview in Java 25

| Feature | Preview n° | Note |
|---|---|---|
| **Structured Concurrency** | 5ª preview | Attesa per Java 29 (LTS) |
| **Stable Values** | 1ª preview | Lazy init con semantica `final`; candidato stabile |
| **Primitive types in patterns/switch** | 3ª preview | Switch su `int`, `float`, ecc. — ancora in discussione |
| **Vector API** | Incubating (permanente?) | Richiede CPU Intel specifica per performance massima |

### Stable Values

Valore lazy-initialized con semantica **final** dopo la prima inizializzazione:

```java
// StableValue: lazy init con JIT optimization come se fosse final
StableValue<Config> config = StableValue.of();

// Prima invocazione: calcola e "freezes"
Config cfg = config.orElseSet(() -> loadConfig());
// Successive: stessa performance di un field final
```

Il JVM può ottimizzare come se fosse un campo `final` dopo che è stato inizializzato.

---

## Library e sicurezza

### Integrity by Default

Il JVM ora emette warning quando le librerie usano:
- JNI (native calls)
- `sun.misc.Unsafe`
- Agent registration
- Reflection su classi interne JDK

Obiettivo: trasparenza su cosa fanno le librerie. Impatto per i maintainer: devono documentare i JVM args necessari.

### Security Manager — disabilitato

Il Security Manager (protezione da `System.exit()`, accesso ai file, ecc.) è stato **disabilitato** in Java 25 — troppo complesso, mai adottato davvero (Google ci ha provato, Tomcat ci ha provato, nessuno l'ha usato). Rimane nel codice ma non fa nulla; verrà rimosso in una versione futura.

### Class File API — Java 24

Sostituzione di ASM per la generazione di bytecode:
- ASM nel JDK era sempre un version behind (Java 25 bundlava il supporto per Java 24)
- La nuova API è fluente (builder pattern) invece del vecchio visitor pattern
- Sempre allineata alla versione corrente del JDK

### Aggiornamenti crittografici

- Algoritmi resistenti al quantum computing (PQC) — quantum computer sono sempre più reali
- Rimozione di algoritmi obsoleti

---

## String Templates — la feature che è scomparsa

Unico caso nella storia di Java di una preview feature **ritirata** (Java 21 preview → Java 22 rimossa):

```java
// Sintassi proposta (mai diventata finale)
String name = "World";
String s = STR."Hello \{name}!";
```

Il team JDK ha deciso di poter fare meglio e ha ritirato la feature per riprogettarla. Non è ancora tornata. **Morale:** le preview possono cambiare radicalmente o sparire — non usarle in produzione.

---

## Citazioni notevoli

> "25% of heap savings after 25 years — just this could actually be a reason to move to 25."  
> — Ario (su Compact Object Headers)

> "AOT is like: run your application once, it saves the profiling. Next time you start, it's almost 3× faster."  
> — Ario (su AOT caching)

> "Virtual threads are now quite reliable — synchronized pinning is fixed in Java 24."  
> — Ario

> "String templates is the first feature ever to disappear. Be aware: previews will be modified and might even disappear. Don't rely too much on previews."  
> — Ario

> "Module import declarations — I'm against star imports, and I'm against module imports as a followup."  
> — Ario (opinione personale)

---

## Riepilogo per decisioni di migrazione

| Ragione per migrare | Categoria |
|---|---|
| **Compact Object Headers**: ~25% heap in meno | Performance |
| **Virtual Thread pinning su `synchronized` risolto** | Affidabilità |
| **AOT** senza step separato: 3× startup | Performance |
| **JFR method tracing** e CPU profiling | Observability |
| **Compact source files**: scripting Java senza boilerplate | Developer Experience |
| **Stream Gatherers**: operazioni intermedie personalizzate | Language |
| **Flexible Constructor Bodies**: validazione prima di super() | Language |
| **Scope Values**: sostituzione ThreadLocal sicura | Concurrency |

---

## Riferimenti e risorse

- **Inside Java** ([inside.java](https://inside.java)) — risorsa ufficiale Oracle per approfondire le novità del JDK
- **JDK Enhancement Proposals** — lista JEP per versione: `https://openjdk.org/projects/jdk/25/`
- **SDKMan** ([sdkman.io](https://sdkman.io)) — tool per gestire versioni JDK, Maven, Gradle su una singola macchina
- **Java Flight Recorder** — `jcmd <pid> JFR.start/dump/stop` + IntelliJ IDEA per analisi flame graph
- **Class File API** — `java.lang.classfile` package (Java 24+)
- **Stream Gatherers** — `java.util.stream.Gatherer` — José Paumard ha presentazioni dedicate
