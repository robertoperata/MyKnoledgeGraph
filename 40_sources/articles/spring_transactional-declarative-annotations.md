---
tags:
  - java
  - spring
  - database
  - transactions
  - aop
  - kotlin
type: article
author: Spring Framework Team
source: https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html
date: 2026-05-09
---

# Using `@Transactional`

## Sunto

La documentazione ufficiale di Spring Framework descrive l'approccio dichiarativo alla gestione delle transazioni tramite l'annotazione **`@Transactional`**, alternativa all'XML-based configuration. L'annotazione permette di esprimere la semantica transazionale direttamente nel codice sorgente Java (o Kotlin), rendendo il comportamento delle transazioni leggibile e mantenibile senza configurazioni esterne.

Spring supporta sia la propria annotazione (`org.springframework.transaction.annotation.Transactional`) sia l'equivalente standard Jakarta (`jakarta.transaction.Transactional`) come drop-in replacement. L'annotazione può essere applicata a livello di classe — propagando il comportamento a tutti i metodi — oppure a livello di singolo metodo per un controllo più granulare. In entrambi i casi, le impostazioni a livello di metodo **sovrascrivono** quelle a livello di classe.

Un aspetto critico documentato è il comportamento dei **proxy AOP**: solo le chiamate provenienti dall'esterno dell'oggetto passano attraverso il proxy e quindi attivano la gestione transazionale. La **self-invocation** (un metodo che chiama un altro metodo dello stesso oggetto) non triggera la transazione in modalità proxy. Per superare questo limite è necessario usare la modalità **AspectJ**, che instrumenta il bytecode direttamente.

Le *transazioni reattive* sono supportate tramite return type reattivi (`Publisher<T>`, `Mono<T>`, `Flow<T>` in Kotlin), permettendo di usare `@Transactional` anche in architetture non-blocking. Il comportamento di default — propagazione `REQUIRED`, isolation `DEFAULT`, rollback solo su `RuntimeException` ed `Error` — può essere personalizzato tramite gli attributi dell'annotazione. A partire da Spring 6.2 è disponibile `@EnableTransactionManagement(rollbackOn=ALL_EXCEPTIONS)` per estendere il rollback automatico anche alle checked exception.

Un pattern avanzato documentato è la creazione di **annotazioni composite personalizzate** (es. `@OrderTx`, `@AccountTx`) che incapsulano la configurazione transazionale ripetuta — transaction manager specifico, label, propagation — riducendo la duplicazione e rendendo l'intent più esplicito nel codice.

---

## Abilitare la gestione transazionale

Per attivare il supporto a `@Transactional` è necessaria una delle seguenti configurazioni:

**Java config:**
```java
@Configuration
@EnableTransactionManagement
public class AppConfig {

    @Bean
    public PlatformTransactionManager txManager(DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
}
```

**XML config:**
```xml
<tx:annotation-driven transaction-manager="txManager"/>
```

> **Nota:** `@EnableTransactionManagement` e `<tx:annotation-driven/>` cercano bean `@Transactional` solo nel medesimo application context in cui sono dichiarati.

---

## Esempi pratici

### Annotazione a livello di classe con override su singolo metodo

Il pattern più comune: read-only di default su tutta la classe, override transazionale sui metodi di scrittura.

```java
@Transactional(readOnly = true)
public class DefaultFooService implements FooService {

    public Foo getFoo(String fooName) {
        // eredita readOnly = true dalla classe
    }

    @Transactional(readOnly = false, propagation = Propagation.REQUIRES_NEW)
    public void updateFoo(Foo foo) {
        // override: read-write, nuova transazione indipendente
    }
}
```

### Transazioni reattive (WebFlux / R2DBC)

```java
@Transactional
public class DefaultFooService implements FooService {

    @Override
    public Mono<Foo> getFoo(String fooName, String barName) {
        // ...
    }

    @Override
    public Mono<Void> insertFoo(Foo foo) {
        // ...
    }
}
```

### Multipli transaction manager

Quando l'applicazione gestisce più datasource, si qualifica il transaction manager per ogni metodo:

```java
public class TransactionalService {

    @Transactional("order")
    public void setSomething(String name) { }

    @Transactional("account")
    public void doSomething() { }

    @Transactional("reactive-account")
    public Mono<Void> doSomethingReactive() { }
}
```

### Annotazioni composite personalizzate

Evitare di ripetere `@Transactional(transactionManager = "order", label = "causal-consistency")` su ogni metodo — incapsularlo in un'annotazione dedicata:

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Transactional(transactionManager = "order", label = "causal-consistency")
public @interface OrderTx { }

@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Transactional(transactionManager = "account", label = "retryable")
public @interface AccountTx { }

// utilizzo:
public class TransactionalService {
    @OrderTx
    public void setSomething(String name) { }

    @AccountTx
    public void doSomething() { }
}
```

---

## Attributi di `@Transactional`

| Attributo | Tipo | Default | Descrizione |
|---|---|---|---|
| `value` / `transactionManager` | `String` | `"transactionManager"` | Qualifier del transaction manager da usare |
| `label` | `String[]` | — | Label descrittive per la transazione |
| `propagation` | `Propagation` | `REQUIRED` | Comportamento di propagazione |
| `isolation` | `Isolation` | `DEFAULT` | Livello di isolamento (si applica su `REQUIRED` o `REQUIRES_NEW`) |
| `timeout` | `int` | -1 (sistema) | Timeout in secondi |
| `timeoutString` | `String` | — | Timeout come stringa (supporta placeholder) |
| `readOnly` | `boolean` | `false` | Transazione read-only |
| `rollbackFor` | `Class[]` | — | Classi di eccezione che causano rollback |
| `rollbackForClassName` | `String[]` | — | Pattern di nome eccezione per rollback |
| `noRollbackFor` | `Class[]` | — | Classi di eccezione che NON causano rollback |
| `noRollbackForClassName` | `String[]` | — | Pattern di nome eccezione esclusi da rollback |

### Comportamento di rollback di default

| Condizione | Comportamento |
|---|---|
| `RuntimeException` o `Error` | Rollback automatico |
| Checked `Exception` | **Nessun rollback** (commit) |
| Spring 6.2+: `rollbackOn=ALL_EXCEPTIONS` | Rollback su tutte le eccezioni |

---

## Visibilità dei metodi e proxy mode

| Tipo proxy | Visibilità supportata | Note |
|---|---|---|
| **JDK (interface-based)** | `public` only | Default quando `proxyTargetClass=false` |
| **CGLIB (class-based)** | `public`, `protected`, package-visible | Default da Spring 6.0 |

**Problema self-invocation:** in proxy mode, un metodo che chiama un altro metodo dello stesso bean non passa attraverso il proxy → la transazione del metodo chiamato **non viene avviata**.

```java
// PROBLEMA: updateFoo() chiama internamente validateFoo()
// validateFoo() è annotata @Transactional ma NON viene avvolta in una transazione
@Service
public class FooService {

    @Transactional
    public void updateFoo(Foo foo) {
        validateFoo(foo); // self-invocation: bypass del proxy!
        // ...
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void validateFoo(Foo foo) {
        // questa annotazione viene IGNORATA quando chiamata da updateFoo()
    }
}
```

**Soluzione:** usare la modalità AspectJ (instrumentazione bytecode) oppure iniettare il bean stesso come dipendenza.

---

## Configurazione annotation-driven

| Attributo XML | Attributo Java | Default | Descrizione |
|---|---|---|---|
| `transaction-manager` | N/A | `transactionManager` | Nome del bean transaction manager |
| `mode` | `mode` | `proxy` | `proxy` o `aspectj` |
| `proxy-target-class` | `proxyTargetClass` | `false` | Usa proxy class-based se `true` |
| `order` | `order` | `LOWEST_PRECEDENCE` | Ordine dell'advice transazionale nella catena AOP |

---

## Punti chiave e trade-off

- **Preferire annotare le classi concrete** (non le interfacce), specialmente con AspectJ mode, per garantire che le annotazioni siano sempre rilevate.
- **Naming delle transazioni:** il nome transazionale segue il pattern `fully.qualified.ClassName.methodName` — utile per il monitoring.
- **readOnly = true:** non garantisce che il database blocchi le scritture — è un hint per l'ottimizzazione del driver/connection pool. Alcuni driver usano questo hint per disabilitare il flush automatico o abilitare ottimizzazioni di lettura.
- **isolation:** si applica solo quando la propagazione crea una nuova transazione fisica (`REQUIRED` con nessuna transazione attiva, oppure `REQUIRES_NEW`). In una transazione nested (Savepoint), l'isolation dell'outer transaction si propaga.
- **Custom composed annotations** sono il pattern raccomandato per codebases con più datasource o strategie transazionali ricorrenti.

---

## Link esterni

- [Transaction Propagation](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/tx-propagation.html) — dettaglio dei comportamenti REQUIRED, REQUIRES_NEW, NESTED, etc.
- [Rollback Rules](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/rolling-back.html) — configurazione avanzata del rollback
- [Using @Transactional with AspectJ](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/aspectj.html) — soluzione al problema self-invocation
- [jakarta.transaction.Transactional](https://jakarta.ee/specifications/transactions/) — specifica Jakarta standard

---

## Immagini

Nessuna immagine presente
