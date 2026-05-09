---
tags:
  - java
  - developer-experience
  - observability
  - testing
type: article
author: Marco Billa, Andrei Belyaev (JetBrains)
source: https://www.youtube.com/watch?v=XWoivKnqsyo
source2: https://blog.jetbrains.com/idea/2025/06/demystifying-spring-boot-with-spring-debugger/
date: 2026-05-05
---

# Spring Debugger for IntelliJ IDEA — Live Coding Session + Article

**Fonti combinate:**
- **Video:** *Troubleshooting Spring Boot Applications with the Spring Debugger* (YouTube, 1h04m) — Marco Billa (developer advocate) + Andrei Belyaev (product manager), JetBrains
- **Articolo:** *Demystifying Spring Boot With Spring Debugger* — Andrey Belyaev, JetBrains Blog, giugno 2025

---

## Sintesi

Spring Debugger è un plugin per IntelliJ IDEA che risponde a una domanda semplice e ricorrente per chi lavora su applicazioni Spring Boot: **"dove devo cliccare?"** — cioè, come trovo rapidamente la fonte di un problema senza dover navigare manualmente tra decine di bean, file di configurazione, livelli transazionali annidati o porte Docker randomiche?

La demo è strutturata come un onboarding ticket su un progetto Spring Boot fittizio (Maven, Java 21, JPA, Thymeleaf, MySQL, Docker Compose) e attraversa 9 task pratici. Per ciascuno, Spring Debugger azzera il tempo di investigazione: l'unica azione richiesta è premere il **tasto debug** — niente configurazione aggiuntiva, niente Spring Boot Actuator, niente agenti custom.

Il plugin funziona inserendo **non-suspending breakpoints** nelle librerie Spring stesse, raccogliendo dati da heap e stack durante startup e runtime, e aggiornando un modello interno all'IDE usato per navigation, inlay e visualizzazioni. L'impatto sulle performance è minimo (hit del breakpoint + stack trace collection). Non richiede nessuna dipendenza aggiuntiva nel progetto.

Le quattro aree coperte oggi: **Property Resolution**, **Bean Visualization**, **Transaction Tracking**, **Database Connection Detection**. In roadmap: SQL query debugging, remote debugging, @Cacheable, event tracing, HTTP tracing, Spring Security visualization.

---

## Feature in dettaglio

### 1. Property Resolution

**Problema:** Spring Boot carica properties da 12-13 sorgenti diverse (file, env var, system properties, custom post-processor…). Capire quale sorgente ha vinto richiede debugging manuale o lettura della documentazione.

**Soluzione Spring Debugger:**

- Nel **Project view**: i file di properties caricati nel contesto attivo hanno una bolla verde; quelli non caricati sono in grigio (subito visibile quali profili sono attivi).
- **Inlay hints** nel file di properties: accanto ad ogni valore configurato compare il valore effettivo a runtime. Se c'è discrepanza (override da altra sorgente), si vede immediatamente.
- **Link navigabile**: cliccando sull'inlay si viene portati alla sorgente che ha definito il valore effettivo — anche se è un `EnvironmentPostProcessor` custom, una env var o una system property (per env var/system property non c'è navigazione ma viene mostrata la sorgente).
- **Expression evaluator** durante il debug: dropdown "Spring Properties" nell'evaluate window — autocomplete su tutte le properties del contesto, incluse quelle Spring predefinite (es. `spring.jpa.hibernate.*`, `spring.kafka.*`). Con "Detect" si vede da quale file/sorgente proviene quella property.

```
Scenario: developer.account = "undefined" nonostante il file marco.properties
→ Spring Debugger mostra che un EnvironmentPostProcessor custom ha la priorità più alta
→ click sul link → naviga direttamente alla classe responsabile
```

---

### 2. Bean Visualization

**Problema:** in un progetto con 13 implementazioni di una stessa interfaccia (condizionate da profili, `@Primary`, `@ConditionalOnProfile`…), capire quale bean è stato effettivamente istanziato richiede breakpoint manuali o ispezione del contesto.

**Soluzione Spring Debugger:**

- **Project view**: i bean istanziati nel contesto corrente hanno icona verde; quelli scansionati ma non istanziati sono in grigio. I **mock bean** (es. `@MockBean` nei test) sono mostrati in arancione.
- **Breakpoint con metadata del bean**: quando si è fermi su un breakpoint in un metodo di un bean, il debugger panel mostra: scope (singleton/prototype), profili attivi, lista dei bean che dipendono da questo, lista dei bean da cui questo dipende.
- **Navigazione alla definizione**: "Navigate" porta alla classe/metodo/XML da cui il bean è stato definito, indipendentemente dal meccanismo (annotation scan, `@Bean` method, XML, legacy context file).
- **Evaluate su qualsiasi bean**: senza che il bean sia iniettato nella classe corrente, si può scrivere `entityManager.createQuery("from Customer").getResultList()` o accedere a `HomeController` con i suoi campi. Funziona anche per le conditional breakpoints.
- **Inlay nel codice**: nei punti di iniezione, Spring Debugger mostra metadati del bean iniettato inline.

> **Lazy beans**: non ancora supportati (osservare il bean lo renderebbe non lazy). In roadmap per fase 2.

---

### 3. Transaction Tracking

**Problema:** con livelli transazionali annidati (propagation REQUIRED, REQUIRES_NEW, ecc.) distribuiti su N service class, capire chi ha aperto la transazione corrente e perché c'è un deadlock richiede navigazione manuale lunga.

**Soluzione Spring Debugger:**

- Quando si entra in un metodo con una transazione attiva, viene aggiunto **automaticamente un watch** nella finestra debugger con i metadati della transazione:
  - `readOnly`: true/false
  - `isolationLevel`: es. DEFAULT
  - `propagationBehavior`: es. REQUIRED, REQUIRES_NEW
  - **L1 entity cache**: lista delle entità gestite dalla persistence context corrente

- **Navigate to source**: porta direttamente al metodo che ha aperto la transazione (non a quello corrente), anche se si è profondi nello stack.

- **Parent transaction**: quando una transazione è `REQUIRES_NEW`, viene mostrata anche la transazione logica padre — utile per tracciare deadlock da transazioni concorrenti sulle stesse righe.

- **Entity lifecycle**: mentre si steppa su `repository.save(entity)`, l'entità passa da `TRANSIENT` a `MANAGED` e compare nella L1 cache — visualizzato in tempo reale.

```
Scenario: click "Add Random Customer" → spinning wheel → timeout
→ Breakpoint in createRandomCustomer → Transaction watch mostra REQUIRED (livello 1)
→ Secondo hit: REQUIRES_NEW da TransactionLevel5Service
→ Navigate to source → identifica il metodo buggy → commenta → fix
```

---

### 4. Database Connection Detection

**Problema:** Spring Boot Docker Compose e Testcontainers assegnano porte random ai container. Connettersi al DB durante il debug richiede di cercare la porta nei log, nel service tool window, o di configurare porte statiche (con tutti i problemi di conflitti).

**Soluzione Spring Debugger:**

- In debug mode, Spring Debugger **popola automaticamente il Database tool window** di IntelliJ con tutti i DataSource rilevati nel contesto applicativo — con username, password e porta già configurati.
- Funziona con: Docker Compose, Testcontainers (`@ServiceConnection`), DataSource standard.
- Supporta tutti i DB relazionali + MongoDB.
- La connessione scompare quando l'applicazione si ferma (o può essere clonata/persistita).
- Non richiede nessuna configurazione: basta il tasto debug.

---

## Implementazione tecnica

| Aspetto | Dettaglio |
|---|---|
| Meccanismo | Non-suspending breakpoints nelle librerie Spring |
| Sorgente dati | Heap, stack trace, frame durante startup e runtime |
| Agenti custom | Non necessari |
| Spring Boot Actuator | Non necessario |
| Impatto performance | Basso (hit breakpoint + stack trace collection + memoria IDE) |
| Versioning | Plugin separato (rilasci indipendenti da IntelliJ IDEA) — diventerà default |
| Compatibilità | Versioni Spring Boot multiple (il plugin detecta la versione e usa i breakpoint giusti) |

---

## Roadmap (in ordine di priorità dichiarata)

| Feature | Stato | Note |
|---|---|---|
| **SQL query debugging** | In sviluppo (priorità 1) | Mostrare query + parametri nel debugger panel; link per eseguirla nella Database console |
| **Remote debugging** | Pianificato | Attach a applicazioni remote (microservizi, app server legacy) |
| `@Cacheable` visualization | Roadmap | Vedere oggetti in cache, evict manuale dall'annotation |
| Event tracing | Roadmap | Navigate to source per eventi Spring pubblicati nel contesto |
| HTTP tracing | Roadmap | HTTP request corrente visibile nel debugger panel |
| Spring Security visualization | Roadmap | Regole/filtri attivi per la request corrente |
| Database connection profiling | Roadmap | — |
| Lazy bean support | Fase 2 | Tecnicamente complesso: osservare il bean lo rende non-lazy |
| Inlay senza breakpoint | Pianificato | Property e bean inlay visibili anche senza debug session attiva |

---

## Citazioni notevoli

> "Where do I have to click? The debug button is literally the only thing you need to do with Spring Debugger."  
> — Marco Billa

> "We don't want you to have to leave IntelliJ IDEA again. Just stay in IDEA because we give you all the information you would ever need right where you are."  
> — Marco Billa

> "As soon as you start observing the photon, it behaves differently. So as soon as you start trying to get something for a lazy bean, it becomes not lazy and you can just disrupt the program execution."  
> — Andrei Belyaev (su perché i lazy bean non sono ancora supportati)

> "The most requested features are SQL debugging and remote debugging."  
> — Andrei Belyaev

---

## Riferimenti e risorse

- **Plugin:** [Spring Debugger su JetBrains Plugin Marketplace](https://plugins.jetbrains.com/plugin/25302-spring-debugger)
- **Documentazione ufficiale:** [JetBrains Help — Spring Debugger](https://www.jetbrains.com/help/idea/spring-debugger.html)
- **Repository demo:** Spring Debugger demo project su GitHub (link nel video)
- **Contatti:** marco@jetbrains.com, andrei@jetbrains.com (per feedback)
- **Thorben Janssen** — esperto Hibernate/JPA, citato nel contesto dell'L1 entity cache
- **Spring Boot Docker Compose support** — legge `compose.yaml` e avvia i container a startup
- **Testcontainers + `@ServiceConnection`** — alternativa a Docker Compose per i test
