---
tags:
  - architecture
  - microservices
  - ddd
  - java
  - database
type: article
author: Axel Fontaine
source: https://www.youtube.com/watch?v=BOvxJaklcr0
date: 2018-11-14
---

# Majestic Modular Monoliths

**Speaker:** Axel Fontaine (fondatore di Flyway e Boxfuse)  
**Evento:** Devoxx  
**Durata:** 41:29  
**Lingua originale:** inglese

---

## Sintesi

Axel Fontaine presenta il **Modular Monolith** come alternativa pragmatica alla scelta binaria tra monolite e microservizi. Il punto di partenza è una critica al cargo-culting: le aziende che adottano un'infrastruttura da 10.000× il loro traffico non si divertono. Amazon e Netflix hanno quel modello perché ne hanno bisogno — AWS ha oggi 3.800 posizioni aperte solo per sviluppatori e architetti. La tua azienda probabilmente no.

Il talk ribalta il frame: invece di parlare di "monolite" (con connotazione negativa) e "microservizi" (con alone eroico), Fontaine propone i termini neutri di **integrated system** (deployment unico) e **distributed system** (deployment separati). E cita Fowler: "The first law of distributed object design: don't distribute your objects." La distribuzione introduce reti inaffidabili, service discovery, circuit breaker, problemi di transazioni distribuite — complessità reale che richiede ingegneria reale.

La soluzione è prendere la **semplicità fisica del monolite** (un unico artefatto deployabile) e combinarla con la **chiarezza logica dei microservizi** (moduli ben separati con API stabili e basso coupling). Il risultato è il Majestic Modular Monolith: organizzazione del codice guidata da DDD (aggregate roots come confini di modulo), dipendenze cicliche assenti, isolamento applicato a livelli crescenti di rigore tramite uno spettro di strumenti.

La sezione sul database ribalta un altro assunto: il database relazionale non è nemico della modularità. Ogni modulo accede solo alle proprie tabelle. **No join cross-modulo** — si usa invece l'API del modulo. La foreign key è accettabile (e si rimuove solo quando si estrae il modulo in servizio separato). Per la comunicazione asincrona tra moduli, Fontaine mostra come il database relazionale stesso possa funzionare da coda con `DELETE ... RETURNING ... FOR UPDATE SKIP LOCKED` su Postgres — atomicità garantita, nessuna infrastruttura aggiuntiva.

Il messaggio finale: per la maggior parte delle organizzazioni, il modular monolith è una scelta più produttiva e pragmatica dei microservizi. Non è una soluzione per i "IT giants" — è per chi vuole produrre software di qualità senza overhead operativo sproporzionato.

---

## Confronto: Monolite vs Microservizi

| Dimensione | Integrated System (Monolite) | Distributed System (Microservizi) |
|---|---|---|
| **Artefatto di deployment** | Uno solo — semplice | Molti — complesso |
| **Accoppiamento** | Rischio di entanglement | Unità separate — bassa coupling |
| **Comunicazione** | Method call — affidabile | Rete — inaffidabile, richiede circuit breaker, service discovery |
| **Disponibilità** | Tutto up o tutto down | Ogni servizio deployato indipendentemente |
| **Refactoring** | IDE trova tutti i riferimenti | Difficile trovare i client di una REST API |
| **Scaling** | Scale all-or-nothing | Knob per ogni servizio |
| **Persistenza** | Un DB — transazioni native | Scelta per servizio, ma transazioni distribuite a carico tuo |
| **Piattaforma** | Unica (JVM, Node…) | Libertà totale (ma JAR hell assente) |
| **Division of labor** | Limitato | Team per servizio — scala alto |

---

## Il Modular Monolith: combinare il meglio dei due

```
Integrated System  +  Logical Architecture of Microservices
(deployment unico)     (moduli con API stabili, basso coupling)
       =
  Majestic Modular Monolith
```

### Principi base (guidati da DDD)

**Aggregate root come confine di modulo**: cercare nella codebase blocchi di funzionalità altamente coesi con basso coupling verso l'esterno. L'aggregate root è l'entità attorno cui tutto il modulo ruota — è l'unico punto d'accesso esposto all'esterno.

**API pubblica vs implementazione privata**: ogni modulo espone solo il necessario — idealmente solo le operazioni sull'aggregate root. Tutto il resto è package-private o visibile solo internamente al modulo.

**Dipendenze solo sulle API, mai sulle implementazioni**:
```
✅  API_A   →  API_B
✅  Impl_A  →  API_B
❌  Impl_A  →  Impl_B   (dipendenza diretta sull'implementazione)
```

### Grafo aciclico delle dipendenze

I moduli devono formare un **DAG** (Directed Acyclic Graph) — nessun ciclo. I moduli stabili (fondamentali per il business) stanno in basso; i moduli volatili dipendono dalla fondazione stabile. Questo minimizza l'impatto di una modifica a cascata.

---

## Lo spettro dell'isolamento del codice

| Livello | Meccanismo | Enforcement | Quando usarlo |
|---|---|---|---|
| 1 | Package separato | Convezione | Sistemi semplici, team piccoli |
| 2 | Maven/Gradle module (jar separato) | Build tool — classpath isolation, acyclic graph | La maggioranza dei casi |
| 3 | Repository separato + versioning | VCS + dependency management | Librerie interne riusabili |
| 4 | Servizio separato (microservizio) | Runtime + network | Solo quando livelli precedenti non bastano |

> "Always start with the simplest thing that could possibly work. Only once you reach the limit of that, go to the next one."

**La linea tra monolite e microservizi sta tra il livello 3 e 4** — non è una scelta binaria, è un continuum.

### Vantaggi del livello 2 (Maven modules)

- Classpath isolation per package-private
- Enforcement del DAG al build tool
- Build parziale: `mvn -pl payment -am` ricostruisce solo il modulo e le sue dipendenze
- Diagramma auto-generato in IntelliJ (Ctrl+Shift+U su i moduli Maven)

---

## Gestione delle dipendenze: evitare il JAR hell

| Strategia | Descrizione |
|---|---|
| **Ridurre le dipendenze** | Valutare se copiare un singolo metodo utility invece di importare tutta la libreria |
| **Zero transitive dependencies** | Preferire librerie senza dipendenze transitive — da includere nei criteri di valutazione |
| **Versione unica ovunque** | `maven-enforcer-plugin` con `dependencyConvergence` — stessa versione di ogni lib in tutti i moduli |
| **Shading** | Ultima risorsa: copiare il codice della libreria sotto il proprio namespace (`com.mycompany.shaded.*`) |

**Snippet Maven Enforcer** (versione unica):
```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-enforcer-plugin</artifactId>
  <executions>
    <execution>
      <goals><goal>enforce</goal></goals>
      <configuration>
        <rules><dependencyConvergence/></rules>
      </configuration>
    </execution>
  </executions>
</plugin>
```

---

## Enforcement dell'architettura: CoDesert / ArchUnit

Per Java 8 (senza JPMS), strumenti come **CoDesert** o **ArchUnit** permettono di definire regole di accesso tra moduli come test:

```java
// Esempio concettuale (CoDesert DSL)
class com.hello = subpackages("com.hello")
dependency rules:
  payment.api is allowed to use invoice.api
  payment.impl is allowed to use payment.api, invoice.api
  // invoice.impl non può usare payment.impl — regola violata → test fallisce
```

Eseguito in pochi millisecondi come parte della test suite. Previene il deterioramento silenzioso dell'architettura nel tempo.

---

## Database: isolamento per modulo

**Regola fondamentale:** ogni modulo accede **solo alle proprie tabelle**. Il modulo non tocca le tabelle degli altri — usa la loro API.

**No cross-module JOIN**: i join tra tabelle di moduli diversi creano accoppiamento a livello di schema. Quando un modulo dovrà diventare servizio separato, quei join saranno il principale ostacolo.

```
✅  SELECT * FROM payment.invoices WHERE id = ?   (tabella propria)
✅  invoiceApi.findById(id)                        (API del modulo Invoice)
❌  SELECT p.*, i.* FROM payment.orders p
    JOIN invoice.invoices i ON p.invoice_id = i.id  (join cross-modulo)
```

**Le foreign key sono accettabili**: referential integrity nel database è un vantaggio reale. Quando un modulo viene estratto in servizio, si rimuove la FK e si sposta la tabella — il resto continua a funzionare.

### Spettro dell'isolamento del database

| Livello | Meccanismo | Quando |
|---|---|---|
| 1 | Stesso schema, tabelle con prefisso per modulo | Sistemi piccoli |
| 2 | DB schema separato per modulo (PostgreSQL schemas) | Soluzione standard raccomandata |
| 3 | Stesso DB engine, istanza diversa per ogni schema estratto | Transizione verso microservizi |
| 4 | Tecnologia di persistenza diversa per modulo | Solo se davvero necessario (graph, document, etc.) |

**Migrazione dello schema**: usare uno strumento come **Flyway** per gestire l'evoluzione del database in modo versionato — file SQL ordinati che vengono applicati automaticamente allo startup dell'applicazione.

---

## Scaling del Modular Monolith

### Scaling verticale
AWS offre istanze da t2.nano (1 vCPU, 0.5 GB RAM) fino a x1e.32xlarge (128 vCPU, ~4 TB RAM). Lo scaling verticale può arrivare molto lontano prima di esaurirsi.

### Scaling orizzontale per modulo: Executor Services dedicati

Se due moduli hanno esigenze di capacità diverse, assegnare **thread pool dedicati** per modulo:

```java
// Payment richiede più throughput → più thread
ExecutorService paymentExecutor = Executors.newFixedThreadPool(8);
// Invoice è meno usato
ExecutorService invoiceExecutor = Executors.newFixedThreadPool(2);
```

### Comunicazione asincrona tra moduli: DB come coda

Quando la comunicazione sincrona non basta, il **database relazionale può fungere da coda** — senza infrastruttura aggiuntiva:

```sql
-- Producer: inserisce un messaggio
INSERT INTO module_queue (payload) VALUES (?);

-- Consumer (Postgres): DELETE atomico con SKIP LOCKED
DELETE FROM module_queue
WHERE id = (
  SELECT id FROM module_queue
  ORDER BY id
  FOR UPDATE SKIP LOCKED
  LIMIT 1
)
RETURNING payload;
```

**Come funziona:**
1. Apro una transazione
2. Eseguo il DELETE con RETURNING — ottengo il payload, blocco la riga
3. `SKIP LOCKED` permette a consumer concorrenti di passare al prossimo elemento
4. Eseguo il lavoro
5. `COMMIT` → il messaggio è rimosso atomicamente insieme ai side-effect del lavoro
6. `ROLLBACK` (eccezione) → il messaggio torna in coda automaticamente

**Questo garantisce:** exactly-once processing, nessun messaggio perso, nessuna infrastruttura esterna. Adatto per carichi non elevatissimi.

Se il DB raggiunge i limiti di capacità per questo uso, si passa a una **coda dedicata** (RabbitMQ, SQS, Kafka) — ma come scelta consapevole, non come default.

---

## JVM come piattaforma poliglotta

Sul JVM si può scrivere ogni modulo nel linguaggio più appropriato:

| Interoperabilità | Esempi |
|---|---|
| Java-only (consume altri) | Scala, JRuby |
| Bidirezionale | Groovy, Kotlin |

Un modulo critico per performance può essere Kotlin, uno di scripting può essere Groovy — tutto nella stessa application, senza overhead di rete.

---

## Citazioni notevoli

> "If you fetishize about the infrastructure required by companies with literally 10,000 times your traffic, you will not have fun."  
> — Axel Fontaine (citando un'osservazione sull'hype microservizi)

> "The first law of distributed object design: don't distribute your objects."  
> — Martin Fowler (citato da Fontaine)

> "Always start with the simplest thing that could possibly work for your context. Only once you reach the limit of that, do go to the next one."  
> — Axel Fontaine

> "Go modular and make your monolith majestic again."  
> — Axel Fontaine (conclusione del talk)

---

## Riferimenti e risorse

- **Eric Evans** — *Domain-Driven Design* (2003) — aggregates, aggregate roots come guida per i confini di modulo
- **Martin Fowler** — "First law of distributed object design: don't distribute your objects"
- **CoDesert** / **ArchUnit** — strumenti Java per enforcement delle regole di architettura come test
- **maven-enforcer-plugin** — `dependencyConvergence` per garantire versione unica delle dipendenze
- **Flyway** ([flywaydb.org](https://flywaydb.org)) — tool open-source di Fontaine per schema migration versionata
- **Boxfuse** — tool di Fontaine per deployment su AWS con un solo comando
- **PostgreSQL `SELECT FOR UPDATE SKIP LOCKED`** — primitiva per implementare code sul DB relazionale
- **IntelliJ IDEA** — Ctrl+Shift+U su moduli Maven per diagramma automatico delle dipendenze
