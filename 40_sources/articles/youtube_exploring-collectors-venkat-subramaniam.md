---
tags:
  - java
  - functional-programming
  - streams
  - collectors
  - concurrency
type: article
author: Venkat Subramaniam
source: https://www.youtube.com/watch?v=pGroX3gmeP8
date: 2026-05-08
---

# Exploring Collectors

**Speaker:** Venkat Subramaniam (agile developer, educator, autore)  
**Evento:** Devoxx  
**Durata:** 2:24:37  
**Lingua originale:** inglese

---

## Sintesi

Venkat Subramaniam tiene un workshop pratico e approfondito sull'API `Collectors` di Java, partendo dai fondamenti di programmazione funzionale con stream fino alle composizioni ricorsive di collector. Il filo conduttore è che **`collect` è un'operazione di reduce** e che i `Collectors` sono una libreria di utility che ci solleva dall'onere di scrivere manualmente codice di riduzione concorrente-safe.

Il talk si struttura in tre blocchi: (1) fondamenti di programmazione funzionale — laziness, purezza delle funzioni, immutabilità; (2) i collector di base — `toList`, `toSet`, `toMap`, `joining`, `partitioningBy`, `groupingBy`; (3) composizione ricorsiva di collector — `mapping`, `filtering`, `collectingAndThen`, `counting`, `maxBy`, `minBy`, `flatMapping`, `teeing`.

Il punto teorico più importante è che **lazy evaluation richiede purezza delle funzioni**: se una funzione ha side-effect, non può essere valutata lazily senza compromettere la correttezza. Questo giustifica l'enfasi sull'immutabilità nel functional programming — non come moda, ma come precondizione per la laziness e quindi per l'efficienza.

Il punto pratico più importante è che **la mutabilità locale è accettabile**, purché non sia condivisa. I `Collectors` sfruttano internamente la mutabilità locale per performance, evitando la shared mutability che causerebbe race condition in stream paralleli.

---

## I fondamentali: filter, map, reduce

### Dalla sintassi alla semantica

Venkat distingue tra sintassi e semantica: aggiungere `.stream()` prima di `.filter()` non è solo zucchero sintattico — è un segnale semantico che sposta l'esecuzione da **eager evaluation** a **lazy evaluation**.

```java
// forEach è terminale ed eager
people.forEach(System.out::println);

// stream() introduce la laziness: filter non esegue finché non arriva il terminale
people.stream()
      .filter(p -> p.getAge() > 30)
      .forEach(System.out::println);
```

### Lazy evaluation e purezza delle funzioni

**Lazy evaluation richiede purezza.** Se una funzione ha side-effect (cambia lo stato esterno), non si può decidere arbitrariamente quando eseguirla — prima o dopo altri eventi cambia il risultato. Quindi:

- **Regola 1 (necessaria):** le funzioni pure non cambiano nulla.
- **Regola 2 (sufficiente):** le funzioni pure non dipendono da nulla che possa cambiare.

Venkat usa l'analogia con Haskell (cattedrale: tutto puro per default, devi "implorare" per rendere qualcosa impuro) vs Java (bazar: tutto impuro per default, devi sforzarti per rendere le cose pure). In Java la responsabilità di scrivere funzioni pure è del programmatore, non del compilatore.

### Reduce: ridurre uno stream a "qualcosa di concreto"

`reduce` non produce necessariamente un singolo valore — produce *qualcosa di più concreto* a partire da uno stream. In Java ha due forme principali:
- **`reduce()`**: riduzione esplicita
- **`collect()`**: riduzione delegata a un `Collector`

---

## Il problema della shared mutability

### Pattern errato (da NON fare)

```java
// SBAGLIATO: forEach con side-effect su lista esterna
List<String> result = new ArrayList<>();
people.stream()
      .filter(p -> p.getAge() > 30)
      .map(Person::getName)
      .map(String::toUpperCase)
      .forEach(result::add); // ← IMPURE: muta una variabile condivisa
```

Il problema è silenzioso in esecuzione sequenziale, ma **distruttivo in parallelo**: race condition che causano perdita di elementi, difficilissime da debuggare.

### Forma corretta con `reduce` a tre argomenti

```java
people.stream()
      .filter(p -> p.getAge() > 30)
      .map(Person::getName)
      .map(String::toUpperCase)
      .reduce(
          new ArrayList<>(),            // valore iniziale (accumulator)
          (names, name) -> { names.add(name); return names; },  // accumulator fn
          (names1, names2) -> { names1.addAll(names2); return names1; } // combiner (per parallelo)
      );
```

La mutabilità qui è **locale** all'accumulator, non condivisa. Ma il codice è verboso, error-prone e nessuno vuole scriverlo ogni giorno.

### Soluzione: `collect` + `Collectors`

```java
people.stream()
      .filter(p -> p.getAge() > 30)
      .map(Person::getName)
      .map(String::toUpperCase)
      .collect(Collectors.toList());
```

`Collectors.toList()` fa esattamente il lavoro del reduce a tre argomenti, gestendo correttamente la concorrenza. **I Collectors sono una libreria di delegation**: deleghiamo il lavoro di reduce al Collector giusto.

---

## I Collector di base

### `toList`, `toSet`, `toUnmodifiableList`

```java
// Lista modificabile
List<Integer> ages = people.stream().map(Person::getAge).collect(toList());

// Lista immutabile (Java 10+) — preferita quando si restituisce a chi chiama
List<Integer> ages = people.stream().map(Person::getAge).collect(toUnmodifiableList());

// Set (rimuove duplicati automaticamente)
Set<String> names = people.stream().map(Person::getName).collect(toSet());
```

**Raccomandazione di Venkat:** preferire `toUnmodifiableList()`, `toUnmodifiableSet()`, `toUnmodifiableMap()` rispetto alle versioni modificabili, specialmente quando il risultato viene restituito al chiamante.

### `joining`

```java
// Stringa comma-separated di nomi in uppercase di over-30
String result = people.stream()
    .filter(p -> p.getAge() > 30)
    .map(Person::getName)
    .map(String::toUpperCase)
    .collect(joining(", "));
```

Risolve elegantemente il problema della "virgola finale" che affligge l'approccio imperativo.

### `toMap`

```java
// Map<nome, età>
Map<String, Integer> nameToAge = people.stream()
    .collect(toMap(Person::getName, Person::getAge));
```

Lancia `IllegalStateException` in caso di chiavi duplicate — gestibile con il terzo argomento (merge function).

---

## Composizione ricorsiva di Collector

### La struttura di `Collector<T, A, R>`

Un `Collector` è parametrico su tre tipi: `T` (tipo dell'input), `A` (tipo dell'accumulatore interno), `R` (tipo del risultato). I Collector si compongono ricorsivamente: un Collector può prendere un altro Collector come argomento.

### `partitioningBy`: split in due gruppi

```java
// Divide in {true: età pari, false: età dispari} in un solo passaggio
Map<Boolean, List<Person>> byParity = people.stream()
    .collect(partitioningBy(p -> p.getAge() % 2 == 0));
```

Alternativa a due filter separati che iterano il stream due volte.

### `groupingBy`: split in N gruppi (il "bucket model")

```java
// Map<nome, lista di persone con quel nome>
Map<String, List<Person>> byName = people.stream()
    .collect(groupingBy(Person::getName));
```

**Mental model dei "bucket":** per ogni persona, `groupingBy` guarda il classificatore (il nome), trova il bucket etichettato con quel nome (o ne crea uno nuovo), e ci butta dentro la persona.

### `groupingBy` + `mapping`: mettere un valore trasformato nel bucket

```java
// Map<nome, lista di età> invece di lista di Person
Map<String, List<Integer>> agesByName = people.stream()
    .collect(groupingBy(Person::getName,
                        mapping(Person::getAge, toList())));
```

`mapping(fn, downstream)` prende la persona, applica `fn` per ottenere il valore da mettere nel bucket, e usa `downstream` (altro Collector) per raccoglierlo. Questo è il motivo per cui si usa `mapping` invece di fare `map()` a livello di stream: la `map()` perderebbe il campo di classificazione usato da `groupingBy`.

### `groupingBy` + `counting`: frequenza

```java
// Quante persone per nome
Map<String, Long> countByName = people.stream()
    .collect(groupingBy(Person::getName, counting()));
```

`counting()` restituisce `Long`. Per ottenere `Integer`:

```java
Map<String, Integer> countByName = people.stream()
    .collect(groupingBy(Person::getName,
                        collectingAndThen(counting(), Long::intValue)));
```

### `collectingAndThen`: collector + trasformazione finale

```java
// Prima applica il collector, poi applica una funzione al risultato
collectingAndThen(counting(), Long::intValue)     // Long → Integer
collectingAndThen(toList(), Collections::unmodifiableList) // lista → immutabile
```

`collectingAndThen(downstream, finisher)` è il duale di `mapping`: mentre `mapping` applica una funzione **prima** di passare al downstream collector, `collectingAndThen` applica la funzione **dopo** che il downstream ha prodotto il suo risultato.

### `maxBy` e `minBy`: estremi comparabili

```java
// Persona più anziana (restituisce Optional<Person>)
Optional<Person> oldest = people.stream()
    .collect(maxBy(comparing(Person::getAge)));

// Nome della persona più giovane
String youngest = people.stream()
    .collect(collectingAndThen(
        minBy(comparing(Person::getAge)),
        p -> p.map(Person::getName).orElse("")));
```

`maxBy`/`minBy` operano sulle **persone**, non sui valori primitivi. Restituiscono `Optional` perché il collection potrebbe essere vuota.

### `filtering`: filtrare durante il reduce (Java 11+)

```java
// Raggruppa per età, ma nell'elenco per ogni età tieni solo nomi > 4 caratteri
Map<Integer, List<String>> result = people.stream()
    .collect(groupingBy(Person::getAge,
                        mapping(Person::getName,
                                filtering(name -> name.length() > 4, toList()))));
```

**`filter` vs `filtering`:** `filter()` applica il predicato sul stream *prima* del reduce. `filtering(pred, downstream)` applica il predicato *durante* il reduce, all'interno della pipeline di collector.

### `teeing`: due collector in parallelo, poi merge (Java 12+)

```java
// Esegue due collector separati e poi combina i risultati
teeing(
    summingInt(Person::getAge),    // primo collector → somma età
    counting(),                    // secondo collector → conteggio
    (sum, count) -> sum / (double) count  // merger → media
)
```

`teeing(downstream1, downstream2, merger)` valuta entrambi i collector sullo stesso stream e poi combina i risultati con `merger`. Utile per calcoli che richiedono due aggregazioni simultanee.

---

## `flatMap` e `flatMapping`

### `map` vs `flatMap`: funzioni 1-to-1 vs 1-to-N

| Funzione nell'input | Metodo da usare | Output |
|---|---|---|
| 1-to-1: un input → un output | `map(fn)` | `Stream<R>` |
| 1-to-N: un input → molti output | `flatMap(fn)` | `Stream<R>` (appiattito) |

**`flatMap` = map + flatten** — nome che preserva l'ergonomia articolatoria rispetto a "map flatten".

```java
// map: ogni numero → numero × 2 (1-to-1)
numbers.stream().map(n -> n * 2)  // [2, 4, 6]

// flatMap: ogni numero → [n-1, n+1] (1-to-N) → lista appiattita
numbers.stream().flatMap(n -> Stream.of(n - 1, n + 1))  // [0, 2, 1, 3, 2, 4]
```

`flatMap` richiede che la funzione restituisca un `Stream` (non una lista o set), perché `flatMap` ha bisogno di un iteratore, non di una collection arbitraria.

### `mapping` vs `flatMapping` nei Collector

Stesso rapporto di `map` vs `flatMap`, ma durante il reduce:

```java
// flatMapping: durante groupingBy, per ogni persona ottieni i caratteri del nome (1-to-N) appiattiti
Map<Integer, Set<String>> charsByAge = people.stream()
    .collect(groupingBy(Person::getAge,
                        mapping(Person::getName,
                                flatMapping(name -> Stream.of(name.split("")),
                                            toSet()))));
```

---

## Eccezzioni e debugging nel functional style

### Eccezioni checked: pattern sbagliato e giusto

**Da evitare:** convertire checked exception in runtime exception per farla "passare" nel lambda. È un abuso che maschera errori reali.

**Approccio corretto:** trattare l'errore come dato. Ispirazione dalla reactive library (RxJava, Reactor): tre canali — data, error, complete. Nel functional style si progetta la pipeline in modo che l'errore sia un valore alternativo nel tipo di ritorno, non un'eccezione che fa esplodere lo stack.

### Debugging

Venkat suggerisce tre discipline pratiche:
1. **Funzioni piccole e coese**: non scrivere lambda con corpo multi-riga. "Lambda is glue code — two lines maybe too many." Estrarre la logica in metodi separati, testabili individualmente.
2. **Test-first**: scrivere prima il test, poi il codice minimo per farlo passare. Il test offre il contesto mentale necessario per debuggare efficacemente.
3. **Breakpoint nel test**: piuttosto che debuggare nell'applicazione, mettere il breakpoint nel test relativo e fare stepping lì — il contesto ristretto rende evidente il problema quasi immediatamente.

### Istanze di tipo e polimorfismo

Quando si è tentati di usare `instanceof` all'interno di un lambda (es. filtro per tipo derivato), Venkat suggerisce di riformulare il problema in termini polimorfici: spostare il comportamento specifico alla classe derivata, anche come no-op nella classe base, eliminando il type-check.

### Sorting

```java
// Ordinamento per età, poi per nome in caso di parità
people.stream()
      .sorted(comparing(Person::getAge).thenComparing(Person::getName))
      .forEach(System.out::println);
```

---

## Mappa concettuale: famiglie di Collector

| Collector | Forma | Funzione |
|---|---|---|
| `toList`, `toSet`, `toMap` | terminali semplici | raccogliere in struttura dati |
| `toUnmodifiableList/Set/Map` | terminali immutabili | come sopra, ma immutabile (Java 10+) |
| `joining` | terminale stringa | concatenare con delimitatore |
| `partitioningBy` | biforcazione | split in due gruppi (boolean) |
| `groupingBy` | multi-bucket | split in N gruppi con downstream opzionale |
| `mapping(fn, downstream)` | trasformazione pre-bucket | map + collector; input → T' → downstream |
| `filtering(pred, downstream)` | filtro pre-downstream | filter + collector (Java 11+) |
| `collectingAndThen(downstream, fn)` | trasformazione post-collector | downstream → R → fn(R) |
| `counting` | aggregazione | conta elementi (→ Long) |
| `summingInt/Long/Double` | aggregazione | somma numerica |
| `maxBy`, `minBy` | aggregazione | estremo secondo comparator (→ Optional) |
| `flatMapping(fn, downstream)` | trasformazione 1-to-N pre-downstream | flatMap + collector (Java 9+) |
| `teeing(d1, d2, merger)` | parallelo | due collector su stesso stream, poi merge (Java 12+) |

---

## Citazioni notevoli

> "Syntax without the right semantics is just noise. What you want to focus on is not what the code says but what the code means."  
> — Venkat Subramaniam

> "Laziness is not just about being efficient. Without laziness you have beautiful syntax but not the semantics of functional programming."  
> — Venkat Subramaniam

> "Immutability is not fashionable. You can't have performance, you can't have lazy evaluation if you don't honor immutability."  
> — Venkat Subramaniam

> "Mutation is okay. Shared mutation is purely evil."  
> — Venkat Subramaniam

> "Lambda expressions are glue code — two lines maybe too many."  
> — Venkat Subramaniam

> "Exception handling is an imperative style of programming concept. In functional programming, treat error as data."  
> — Venkat Subramaniam

> "flatMap = map and then flatten. It's written to not hurt your jaw."  
> — Venkat Subramaniam (sul nome `flatMap` vs "map flatten")

---

## Riferimenti e risorse

- **Venkat Subramaniam** — [agiledeveloper.com](https://agiledeveloper.com) — corsi, libri ("Programming Kotlin", "Functional Programming in Java"), talk
- **Java Streams API** — `java.util.stream.Collectors` (Javadoc)
- **`Comparator`** — `java.util.Comparator.comparing()`, `thenComparing()` per sorting composito
- **Reactive libraries** (RxJava, Project Reactor) — ispirazione per gestire errori come dati nei canali separati
- **Haskell** — linguaggio citato come riferimento per purezza assoluta (opposto di Java)
- **Kotlin** — citato per `flatten()` esplicita (che Java non ha nativa a livello di API, ma il comportamento è replicato da `flatMap`)
