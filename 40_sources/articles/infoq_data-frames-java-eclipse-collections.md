---
tags:
  - java
  - performance
  - database
  - developer-experience
type: article
author: Vladimir Zakharov
source: https://www.infoq.com/presentations/data-frames-java/
date: 2026-02-09
---

# Are You Missing a Data Frame? The Power of Data Frames in Java

## Sunto

Vladimir Zakharov — autore e committer di DataFrame-EC, già membro dell'Executive Committee del Java Community Process e contributor a JSR 335 (Lambda e Stream API di Java 8) — presenta i *DataFrame* come strumento fondamentale per la programmazione orientata ai dati nell'ecosistema Java. Il punto di partenza è una domanda diretta: gli sviluppatori Java stanno perdendo qualcosa rispetto a Python/pandas per la manipolazione di dataset tabulari?

La presentazione contrappone due paradigmi. La programmazione **orientata agli oggetti** incapsula stato e comportamento in classi, usando il polimorfismo per nascondere i dettagli implementativi. La programmazione **orientata ai dati** modella il dominio come collezioni immutabili di proprietà — simili a tabelle relazionali o Java records — con funzioni separate che operano su quei dati esposti. Un DataFrame è proprio questo: una struttura tabellare composta da colonne tipizzate, con operazioni di alto livello (filtraggio, aggregazione, join, pivot) già integrate.

Il banco di prova è il *One Billion Row Challenge* (Gunnar Morling, gennaio 2024): elaborare un miliardo di righe di misurazioni temperatura per stazione. I risultati mostrano che **DataFrame-EC** (basato su Eclipse Collections) è competitivo con Python/pandas in termini di performance e lo supera nel consumo di memoria. Kotlin DataFrame, al contrario, non completa nemmeno la sfida al miliardo di righe. Tablesaw carica i dati velocemente ma è di un ordine di grandezza più lento nell'aggregazione.

| Framework | 100M righe | 1B righe | Memoria |
|---|---|---|---|
| Python/pandas | Più veloce | Completa | Alta |
| DataFrame-EC (16GB heap) | Buono | Completa | Efficiente |
| DataFrame-EC (32GB heap) | Buono | Migliore (meno GC) | Efficiente |
| Tablesaw | Più lento (aggregazione) | Aggregazione molto lenta | Media |
| Kotlin DataFrame | Lento | Non completa | — |

Internamente, le performance di DataFrame-EC derivano dall'uso di strutture dati specializzate: *primitive collections* (`IntList`, `DoubleList`) evitano il boxing, l'*object pooling* riduce l'impronta di stringhe e date in memoria, le *bitmap mutabili* gestiscono i null per i primitivi, e le *multimap* forniscono indici per lookup veloci. Tutto questo è costruito sopra Eclipse Collections, che aggiunge tipi primitivi, collezioni immutabili e API funzionali ricche al JDK standard.

Il valore pratico non è solo la performance: i DataFrame permettono di scrivere pipeline di trasformazione dati enterprise senza dover uscire dall'ecosistema Java, senza la complessità di un cluster Spark, e con più flessibilità di un database relazionale. Casi d'uso tipici: esplorazione ad hoc di dati, pipeline ETL, elaborazione di file come punti di recovery tra sistemi di record.

---

## Esempi pratici

### Stessa operazione in 4 framework a confronto (One Billion Row Challenge)

```python
# Python/pandas
df = pd.read_csv(FILE)
result = df.groupby('station').agg({
    'temperature': ['min', 'mean', 'max']
}).sort_values('station')
```

```java
// DataFrame-EC
DataFrameSchema schema = DataFrameSchema.builder()
    .addColumn("station", String.class)
    .addColumn("temperature", Double.class)
    .separator(';')
    .hasHeader(false)
    .build();

DataFrame df = DataFrameLoaderFactory.load(FILE, schema);

DataFrame result = df
    .aggregateBy(Lists.immutable.of(
        new AggregationFunction("temperature", "min", Min.class),
        new AggregationFunction("temperature", "mean", Avg.class),
        new AggregationFunction("temperature", "max", Max.class)
    ))
    .groupBy("station")
    .sortBy("station");
```

```java
// Tablesaw
Table df = Table.read().csv(FILE);
Table result = df
    .summarize("temperature", min, mean, max)
    .by("station")
    .sortAscendingOn("station");
```

```kotlin
// Kotlin DataFrame
val df = readCSV(FILE)
val result = df
    .groupBy { it["station"] }
    .aggregate {
        "min" to it["temperature"].min(),
        "mean" to it["temperature"].mean(),
        "max" to it["temperature"].max()
    }
    .sortBy { it["station"] }
```

### Filtro con DSL esterno (DataFrame-EC) vs DSL interno (Tablesaw)

```java
// DataFrame-EC: DSL esterno (stringa)
DataFrame result = orders.selectBy(
    "orderDate = $tomorrow AND (quantity >= 12 OR customer = 'Bob')"
);

// Tablesaw: DSL interno (espressioni Java)
Table result = orders.where(
    orders.dateColumn("orderDate").isEqualTo(tomorrow)
    .and(
        orders.intColumn("quantity").isGreaterThanOrEqualTo(12)
        .or(orders.stringColumn("customer").isEqualTo("Bob"))
    )
);
```

### Join con colonna calcolata (DataFrame-EC)

```java
// Aggiunge colonna derivata con espressione, poi fa il join
DataFrame prices = donuts.addColumn(
    "order_price",
    "quantity < 12 ? price : discount_price"
);

DataFrame withPrices = orders.joinBy("donut")
    .with(prices)
    .keepColumns("customer", "order_price");

DataFrame result = withPrices
    .aggregateBy(Lists.immutable.of(
        new AggregationFunction("order_price", "total_spend", Sum.class)
    ))
    .groupBy("customer")
    .sortBy("customer");
```

### Pivot (conteggio ciambelle per cliente per giorno)

```java
// DataFrame-EC
DataFrame result = orders.pivot(
    Lists.immutable.of("customer", "orderDate"),
    Lists.immutable.of(new AggregationFunction("donut", "count", Count.class))
);

// Tablesaw
Table result = orders.pivot("customer", "orderDate", "donut", count);
```

---

## Link esterni

- [The One Billion Row Challenge](https://github.com/gunnarmorling/1brc) — challenge di Gunnar Morling (gennaio 2024) usato come benchmark nella presentazione
- [Eclipse Collections](https://www.eclipse.org/collections/) — framework su cui è costruito DataFrame-EC; aggiunge tipi primitivi, collezioni immutabili e API ricche al JDK
- [DataFrame-EC su GitHub](https://github.com/vmzakharov/dataframe-ec) — il framework presentato, con esempi Python/Kotlin/Java

---

## Immagini

Nessuna immagine presente
