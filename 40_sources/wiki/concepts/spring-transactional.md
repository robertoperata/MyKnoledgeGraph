---
title: Spring @Transactional — Gestione Dichiarativa delle Transazioni
type: concept
tags:
  - thread2-java
sources:
  - "[[spring_transactional-declarative-annotations]]"
updated: 2026-05-09
related:
  - "[[concepts/jpa-hibernate-performance]]"
  - "[[patterns/transactional-outbox]]"
  - "[[concepts/java-concurrency]]"
---

# Spring @Transactional — Gestione Dichiarativa delle Transazioni

## Definizione

`@Transactional` è l'annotazione Spring (e Jakarta EE) che esprime la semantica transazionale direttamente nel codice sorgente. Sostituisce la configurazione XML e delega a Spring AOP la responsabilità di aprire, committare o rollbackare la transazione attorno ai metodi annotati.

Due versioni coesistono:
- `org.springframework.transaction.annotation.Transactional` — Spring-native, più attributi
- `jakarta.transaction.Transactional` — standard Jakarta, supportata come drop-in replacement

## Comportamento di default

| Attributo | Default |
|---|---|
| Propagazione | `REQUIRED` (usa transazione esistente o ne crea una) |
| Isolation | `DEFAULT` (delega al database) |
| readOnly | `false` (read-write) |
| Timeout | Nessuno (usa il default del sistema) |
| Rollback su | `RuntimeException` e `Error` |
| **NON rollback su** | **Checked `Exception`** ← fonte frequente di bug |

> **Insidia critica:** le checked exception non causano rollback di default. Un metodo annotato `@Transactional` che lancia una `IOException` o `SQLException` causa il **commit** della transazione, non il rollback. Per estendere il rollback: `@Transactional(rollbackFor = Exception.class)` oppure, da Spring 6.2+: `@EnableTransactionManagement(rollbackOn = ALL_EXCEPTIONS)`.

## Il problema della self-invocation (proxy mode)

Spring implementa `@Transactional` tramite **proxy AOP**. Il proxy intercetta solo le chiamate che arrivano dall'esterno del bean. Se un metodo chiama un altro metodo dello stesso oggetto (self-invocation), la chiamata bypassa il proxy e la transazione del metodo interno **non viene avviata**.

```java
@Service
public class FooService {

    @Transactional
    public void outer() {
        inner(); // ← self-invocation: bypassa il proxy!
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void inner() {
        // Questa annotazione viene IGNORATA quando chiamata da outer()
        // Non si apre una nuova transazione separata
    }
}
```

**Soluzioni:**
1. **AspectJ mode** (`@EnableTransactionManagement(mode = AdviceMode.ASPECTJ)`) — instrumentazione bytecode, nessun proxy, nessun limite di visibilità
2. **Self-injection** — iniettare il bean stesso come dipendenza e chiamare il metodo sul proxy
3. **Estrarre il metodo in un bean separato** — approccio più pulito architetturalmente

## Visibilità dei metodi

| Tipo proxy | Visibilità supportata |
|---|---|
| JDK (interface-based, `proxyTargetClass=false`) | `public` only |
| CGLIB (class-based, default da Spring 6.0) | `public`, `protected`, package-visible |

Il Spring team raccomanda di annotare metodi delle **classi concrete**, non delle interfacce, per garantire compatibilità con entrambe le modalità proxy e con AspectJ.

## readOnly = true

`readOnly = true` **non** impedisce le scritture a livello di database — è un **hint per il driver e il framework**. Effetti pratici:

- Hibernate disabilita il dirty checking → nessun flush automatico → risparmio CPU
- Alcuni driver JDBC ottimizzano la connessione per le letture
- HikariCP può usare read replica se configurato

**Pattern comune:** classe annotata `@Transactional(readOnly = true)` per tutti i metodi, con override `@Transactional(readOnly = false)` sui soli metodi di scrittura.

```java
@Transactional(readOnly = true)
public class DefaultFooService implements FooService {

    public Foo getFoo(String id) { ... } // eredita readOnly = true

    @Transactional(readOnly = false, propagation = Propagation.REQUIRES_NEW)
    public void updateFoo(Foo foo) { ... } // override per scrittura
}
```

## Propagazione — i casi d'uso principali

| Propagation | Comportamento | Quando usarlo |
|---|---|---|
| `REQUIRED` (default) | Usa transazione esistente; se non esiste ne crea una | Caso generale |
| `REQUIRES_NEW` | Sospende la transazione corrente, ne apre una nuova | Operazioni che devono committare indipendentemente (es. audit log) |
| `NESTED` | Crea un Savepoint dentro la transazione corrente | Operazioni parzialmente annullabili |
| `MANDATORY` | Richiede transazione esistente, altrimenti eccezione | Metodi che non possono essere chiamati senza contesto transazionale |
| `NOT_SUPPORTED` | Sospende la transazione corrente | Operazioni read-only veloci che non beneficiano della transazione |

> **Anti-pattern documentato (Walkowiak):** `REQUIRES_NEW` richiede **due connessioni simultanee** dal pool — una per la transazione outer in attesa, una per quella inner. Con pool piccoli (HikariCP default: 10) questo può causare deadlock del pool.

## Multipli transaction manager e annotazioni composite

Quando l'applicazione gestisce più datasource, si qualifica il transaction manager per ogni metodo. Per evitare ripetizioni, si creano **annotazioni composite**:

```java
// Definizione dell'annotazione composita
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Transactional(transactionManager = "order", label = "causal-consistency")
public @interface OrderTx { }

// Utilizzo — intento dichiarativo esplicito
@OrderTx
public void processOrder(Order order) { ... }
```

## Transazioni reattive

Per architetture non-blocking (WebFlux, R2DBC), `@Transactional` funziona con return type reattivi. Il transaction manager deve essere reattivo (`R2dbcTransactionManager`, `ReactiveMongoTransactionManager`):

```java
@Transactional
public Mono<Void> insertFoo(Foo foo) { ... }

@Transactional
public Flow<Foo> getAllFoos() { ... }  // Kotlin coroutines
```

Il contesto transazionale si propaga tramite il `ReactiveTransactionManager` e il `TransactionContextManager` reattivo — non tramite `ThreadLocal` come nel caso sincrono.

## Tensioni documentate

> **Tensione: `@Transactional` è sia la soluzione sia il problema in JPA.**
> - **Come soluzione** (anti-pattern 6 in Walkowiak): senza `@Transactional`, ogni chiamata al repository apre una sessione Hibernate separata → le entity diventano unmanaged → SELECT extra inattesi.
> - **Come problema** (anti-pattern 3 in Walkowiak): `@Transactional` a grana grossa tiene la connessione aperta durante chiamate esterne (REST, sleep, ecc.) → esaurimento del connection pool.
> - **Risoluzione:** usare `TransactionTemplate` per il controllo esplicito dei confini quando il metodo fa operazioni miste (DB + chiamate esterne).

## Connessioni

- [[concepts/jpa-hibernate-performance]] — anti-pattern @Transactional a grana grossa vs sessioni separate; REQUIRES_NEW e il connection pool; readOnly come ottimizzazione Hibernate
- [[patterns/transactional-outbox]] — l'outbox pattern richiede una transazione @Transactional che includa sia la scrittura dell'entity sia l'INSERT nella tabella outbox; il rollback su checked exception è critico qui
- [[concepts/java-concurrency]] — il connection pool è thread-bounded; REQUIRES_NEW tiene una connessione bloccata per thread
