---
title: Modular Monolith
type: pattern
tags: [thread1-microservices, thread5-ddd]
sources:
  - "[[youtube_majestic-modular-monolith-axel-fontaine]]"
updated: 2026-05-08
related:
  - "[[concepts/bounded-context]]"
  - "[[concepts/information-hiding]]"
  - "[[concepts/independent-deployability]]"
  - "[[concepts/legacy-modernization]]"
  - "[[concepts/sync-async-hybrid-architecture]]"
  - "[[patterns/anti-corruption-layer]]"
---

# Modular Monolith

## Problema che risolve

La scelta tra monolite e microservizi è spesso presentata come binaria. I microservizi introducono complessità reale: reti inaffidabili, service discovery, circuit breaker, transazioni distribuite, overhead operativo. Per la maggior parte delle organizzazioni — che non hanno la scala di Amazon o Netflix — questo overhead non è giustificato.

D'altra parte, il monolite tradizionale con architettura a layer tende all'entanglement: i moduli funzionali (customer, invoice, payment) si intrecciano nel tempo attraverso dipendenze trasversali tra repository e service class.

> "The first law of distributed object design: don't distribute your objects." — Martin Fowler

## Soluzione: separazione fisica e logica ortogonali

```
Integrated deployment (un artefatto)
        +
Logical architecture of microservices (moduli con API stabili)
        =
Majestic Modular Monolith
```

La separazione **logica** (moduli ben definiti) è indipendente dalla separazione **fisica** (deployment separati). Un monolite ben modulare ha la chiarezza organizzativa dei microservizi senza il costo operativo della distribuzione.

## Definizione dei confini di modulo (DDD)

I confini dei moduli si trovano cercando blocchi di funzionalità **altamente coesi** con **basso coupling** verso l'esterno. La guida viene dal DDD:

- L'**aggregate root** è il punto d'accesso del modulo verso l'esterno
- Tutte le altre entity e value object sono **private** al modulo
- La **API pubblica** del modulo espone solo operazioni sull'aggregate root

### Regola delle dipendenze

```
✅  API_A   →  API_B        (dipendenza da API)
✅  Impl_A  →  API_B        (dipendenza da API)
❌  Impl_A  →  Impl_B       (dipendenza diretta sull'implementazione altrui)
```

Le dipendenze tra moduli devono formare un **DAG** (grafo aciclico diretto): i moduli stabili (fondamentali al business) stanno in basso; i volatili dipendono dalla fondazione stabile. Nessun ciclo.

## Spettro dell'isolamento del codice

| Livello | Meccanismo | Enforcement | Quando |
|---|---|---|---|
| 1 | Package separato | Convenzione | Sistemi semplici |
| 2 | Maven/Gradle module | Build tool (classpath isolation + DAG) | La maggioranza dei casi |
| 3 | Repository separato + versioning | VCS + dependency management | Librerie interne riusabili |
| 4 | Servizio separato | Runtime + network | Solo quando livelli precedenti non bastano |

> "Always start with the simplest thing that could possibly work. Only once you reach the limit of that, go to the next one." — Axel Fontaine

**La linea tra monolite e microservizi è tra il livello 3 e 4** — esistono molte scelte prima di distribuire.

### Vantaggi del livello 2 (Maven/Gradle modules)

- Classpath isolation applicata
- DAG enforced dal build tool — cicli rilevati al build
- Build parziale: ricostruisce solo il modulo e le sue dipendenze
- Diagramma auto-generato in IntelliJ (Ctrl+Shift+U sui moduli Maven)

## Enforcement architetturale

Per garantire che nessuno dipenda dall'implementazione altrui, strumenti come **ArchUnit** (Java) permettono di scrivere regole come test:

```java
// Regola: payment.impl non può accedere a invoice.impl
ArchRule rule = noClasses()
    .that().resideInAPackage("..payment.impl..")
    .should().accessClassesThat()
    .resideInAPackage("..invoice.impl..");
// Eseguito in ms come parte della test suite
```

Il `maven-enforcer-plugin` con `dependencyConvergence` garantisce una sola versione di ogni libreria in tutti i moduli:

```xml
<plugin>
  <artifactId>maven-enforcer-plugin</artifactId>
  <configuration>
    <rules><dependencyConvergence/></rules>
  </configuration>
</plugin>
```

## Isolamento del database per modulo

**Regola:** ogni modulo accede solo alle proprie tabelle. No cross-module JOIN.

```sql
✅  SELECT * FROM invoice.invoices WHERE id = ?   -- tabella propria
✅  invoiceApi.findById(id)                        -- API del modulo
❌  SELECT * FROM payment.orders p
    JOIN invoice.invoices i ON p.invoice_id = i.id -- join cross-modulo
```

Le **foreign key sono accettabili** e utili (referential integrity): si rimuovono solo quando il modulo viene estratto come servizio separato. Il resto dell'applicazione continua a funzionare invariato.

### Spettro dell'isolamento del database

| Livello | Meccanismo |
|---|---|
| 1 | Stesso schema, tabelle con prefisso per modulo |
| 2 | DB schema separato per modulo (PostgreSQL schemas) |
| 3 | Stesso engine, istanza DB per ogni schema estratto |
| 4 | Tecnologia di persistenza diversa per modulo |

## Comunicazione asincrona tra moduli: DB as Queue

Per comunicazione asincrona senza infrastruttura aggiuntiva, il **database relazionale può fungere da coda** usando `SELECT FOR UPDATE SKIP LOCKED` (PostgreSQL, MySQL 8+):

```sql
-- Consumer: preleva atomicamente un messaggio
DELETE FROM module_queue
WHERE id = (
  SELECT id FROM module_queue
  ORDER BY id
  FOR UPDATE SKIP LOCKED
  LIMIT 1
)
RETURNING payload;
```

**Semantica:**
- `FOR UPDATE`: blocca la riga selezionata
- `SKIP LOCKED`: consumer concorrenti saltano righe già bloccate → nessuna contesa
- `DELETE ... RETURNING`: rimuove e restituisce atomicamente
- `COMMIT` → messaggio rimosso + side-effect del lavoro committed insieme
- `ROLLBACK` → messaggio torna automaticamente in coda

**Adatto per:** carichi moderati dove si vuole coerenza forte tra la coda e gli altri dati senza infrastruttura di messaggistica separata.

**Quando passare a una coda dedicata** (RabbitMQ, SQS, Kafka): solo quando il DB raggiunge i limiti di throughput per questo uso.

## Scaling per modulo

| Tecnica | Come | Quando |
|---|---|---|
| Thread pool dedicati | `ExecutorService` dimensionato per modulo | Moduli con esigenze di throughput diverse |
| Comunicazione asincrona | DB as queue o message broker | Quando il coupling sincrono non è necessario |
| Scale-up verticale | AWS fino a 128 vCPU / 4 TB RAM | Prima di passare a scaling orizzontale separato |

## Trade-off rispetto ai microservizi

| | Modular Monolith | Microservizi |
|---|---|---|
| **Complessità operativa** | Bassa (un deployment) | Alta (service discovery, circuit breaker, ecc.) |
| **Transazioni** | ACID native del DB | Distribuite → saga, eventual consistency |
| **Independent deployability** | Logica (non fisica) | Fisica |
| **Scaling** | Per thread pool o scale-up | Per servizio |
| **Confine ben definito** | Sì (se ben modulare) | Sì (se ben progettato) |
| **Refactoring** | IDE trova tutti i riferimenti | Difficile per API REST/eventi |

## Quando estrarre un modulo in microservizio

Il modular monolith è preparato per l'estrazione: i confini sono già ben definiti, le API pubbliche esistono, i dati sono già isolati. L'estrazione diventa un'operazione chirurgica:
1. Rimuovere le FK verso altri moduli
2. Spostare le tabelle del modulo in un DB separato
3. Convertire le chiamate interne alle API in chiamate remote
4. Deployare separatamente

## Caso reale: Revolut

Revolut ha seguito esattamente questo spettro: MVP monolite (2015, 2 ingegneri) → event-driven monolite (framework "actions + eventi" in-house per eliminare il service spaghetti) → event-driven services (separazione guidata dall'event store centralizzato). Il codice era già modulare internamente quando la separazione fisica è avvenuta — l'estrazione è stata chirurgica.

## Connessioni

- [[concepts/bounded-context]] — gli aggregate roots DDD definiscono i confini dei moduli; il modular monolith è DDD applicato a livello intra-applicazione
- [[concepts/information-hiding]] — API pubblica vs implementazione privata è information hiding applicato al livello di modulo
- [[concepts/independent-deployability]] — il modular monolith fornisce separazione logica; quando la separazione fisica diventa necessaria, i confini sono già pronti
- [[concepts/legacy-modernization]] — il modular monolith è lo step intermedio naturale nello spettro prima di estrarre servizi con Strangler Fig
- [[concepts/sync-async-hybrid-architecture]] — DB as queue è un'applicazione del principio di comunicazione asincrona interna tra moduli
- [[patterns/anti-corruption-layer]] — quando un modulo deve integrarsi con sistemi a semantiche diverse, l'ACL protegge il suo modello interno
