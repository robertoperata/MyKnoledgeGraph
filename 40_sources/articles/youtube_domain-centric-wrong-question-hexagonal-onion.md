---
tags:
  - architecture
  - ddd
  - microservices
  - developer-experience
type: article
author: Oliver Drotbohm (Spring Engineering Team, VMware/Broadcom)
source: https://www.youtube.com/watch?v=co3acmgP2Ng
date: 2026-05-05
---

# Domain Centric? Why Hexagonal, Onion and Clean Architecture Are Answers to the Wrong Question

**Speaker:** Oliver Drotbohm (Spring Engineering Team, 15+ anni; Spring Data, Spring Modulith)  
**Durata:** 48:02  
**Lingua originale:** inglese

---

## Sintesi

Oliver Drotbohm (noto come "Oli", Spring Engineering Team) presenta un'analisi critica delle architetture di separazione dei concern — Hexagonal (2005, Alistair Cockburn), Onion (2008), Clean Architecture — sostenendo che non siano sbagliate in sé, ma che rispondano a una domanda secondaria trascurando quella primaria: **come decomporre il dominio**.

Il filo conduttore è la definizione di costo del software. Partendo da Kent Beck (DDD Europe) e risalendo al libro di Constantine & Yourdon del 1979, il costo del software è il costo di cambiarlo, e il costo del cambio è il **coupling**: quando per modificare un elemento dobbiamo modificarne un altro. Il design software è quindi un'attività di scommessa sulle future direzioni di cambiamento — creare gruppi di elementi ad alta **coesione** che siano **loosely coupled** tra loro.

Il punto critico: le architetture hexagonal/onion/clean si occupano di **decomposizione tecnica** (adapters, ports, rings) ma non offrono alcuna guida sulla **decomposizione del dominio**. Il termine "domain-centric" indica solo che il dominio è al centro dell'immagine — non che l'architettura aiuti a capire o strutturare il dominio stesso. Nella pratica, tutti si entusiasmano per la struttura tecnica e nessuno vuole fare il lavoro difficile di acquisire conoscenza del dominio.

La proposta alternativa: **sliced onion** o **vertical slices** — moduli verticali funzionalmente coesi, dove la complessità interna è determinata dalla complessità del dominio di quel modulo. Un modulo semplice può avere JDBC direttamente nel controller; uno complesso con pagamenti e API di terze parti merita ports e adapters. L'encapsulamento via visibilità Java (package-private) è lo strumento chiave.

---

## Punti chiave

- **Il costo del software è il costo di cambiarlo** (Kent Beck, DDD Europe): le due uniche attività sul software sono aumentare il valore o ridurre il costo. Il costo dominante è il costo del cambiamento.

- **Coupling = costo del cambiamento**: definizione di Constantine (1979): due elementi sono accoppiati se per modificare uno devo modificare l'altro. Non tutto il coupling è uguale — dipende dalla *natura del cambiamento*, non dalla presenza di relazioni tra elementi.

- **Cohesion = coupling nei posti giusti**: la coesione è coupling nelle direzioni volute. Molti team si concentrano ossessivamente sul rimuovere coupling e trascurano la coesione — il risultato è bassa coesione con basso coupling, che è altrettanto problematico.

- **Il design software è una scommessa formalizzata**: creare strutture nel codice significa scommettere su quali cambiamenti arriveranno in futuro. Non è una scienza esatta.

- **Hexagonal/Onion = Layered architecture con due differenze**: inversione della direzione di una freccia (da outside-in invece di top-down) e insistenza sulle interfacce. Non è molto di più. Cockburn e Palermo risolvevano un problema reale degli anni 2000: il deployment su EJB application server rendeva impossibile testare la business logic senza bundlare e deployare. Spring ha risolto questo problema con dependency injection e IoC.

- **"Domain-centric" è fuorviante**: il dominio è al centro dell'*immagine*, non dell'*architettura*. Queste architetture non offrono guida su come decomporre il dominio. La decomposizione tecnica è enfatizzata; la comprensione del dominio è ignorata.

- **Il vero problema è la decomposizione del dominio**: se hai più parti del dominio (ordini, inventario, pagamenti), come interagiscono? Le architetture hexagonal/onion non rispondono — ti lasciano con "metti un'onion accanto all'altra", il che porta a domande su come attraversare i confini.

- **Sliced Onion / Vertical Slices**: "taglia" l'onion verticalmente — moduli con coesione funzionale invece che tecnica. Ogni modulo espone solo ciò che serve all'esterno (bean/eventi nella terminologia Spring). L'interno del modulo è irrilevante per gli altri.

- **La complessità interna è determinata dalla complessità del dominio**: un modulo che legge due tabelle e restituisce JSON può avere JDBC nel controller. Un modulo con pagamenti, API di terze parti, e regole di business complesse merita ports, adapters, e abstrazioni. **Non serve uniformità a tutti i costi**.

- **Encapsulamento > organizzazione**: usare la visibilità Java (package-private) per nascondere implementazioni. Il problema del layered architecture classico: i repository sono `public`, quindi qualsiasi parte del codebase può bypassare il service interface. Con moduli verticali package-private, solo l'interfaccia esposta è visibile.

- **J-Molecules e Spring Modulith Tooling**: libreria per annotare il codice con ruoli architetturali (primary port, aggregate root, ecc.) senza vincolare l'implementazione. L'IDE può poi presentare il progetto a un livello di astrazione più alto — non come albero di file ma come grafo di moduli con i loro ruoli.

---

## La critica al "domain-centric" spiegata con un'analogia

> "Our goal is to find reasonable groupings of elements in a way that when we start a change, the effects of the change stay in these groups."

Il design a layer tecnici (controllers / services / repositories) è come mettere "tutte le sedie in una stanza, tutti i tavoli in un'altra, tutti gli armadi in una terza". Funziona se passi la giornata a rispondere alla porta e dare sedie ai visitatori. Non funziona se vuoi invitare amici a cena — devi portare insieme le cose che insieme servono un scopo.

I package tecnici (`adapters`, `ports`, `services`, `repositories`) creano unità di bassa coesione funzionale. Un OrderRepository e un CustomerRepository sono nello stesso package solo perché entrambi sono repository — non perché abbiano qualcosa da dirsi.

---

## Evoluzione visiva del problema (struttura dei package)

| Approccio | Struttura | Problema |
|---|---|---|
| **Layered** | `web/`, `service/`, `persistence/` | Bassa coesione funzionale; tutto è `public` |
| **Hexagonal** | `adapters/`, `ports/`, `application/` | Stessa bassa coesione; nomi diversi |
| **Vertical Slices** | `order/`, `customer/`, `payment/` | Alta coesione funzionale; encapsulamento via package-private |

Con i vertical slices: solo l'interfaccia pubblica del modulo è visibile all'esterno. Il resto è package-private. Se `OrderRepository` è package-private, nessuno può bypassare `OrderService`.

---

## La domanda giusta da fare

Invece di: *"Come separo i concern tecnici nel mio codice?"*

La domanda giusta è: *"Quali sono i cambiamenti che affronterò regolarmente, e come struttura il codice in modo che quei cambiamenti rimangano contenuti in unità coese?"*

La risposta quasi sempre porta a **decomposizione funzionale verticale** — non tecnica.

---

## Raccomandazioni pratiche

1. **Favorire la decomposizione funzionale sulla decomposizione tecnica** — non partire mai dalla struttura tecnica
2. **La complessità intrinseca determina la complessità accidentale** — non imporre hexagonal architecture dove non serve
3. **Usare l'encapsulamento come primo strumento** — non tutto deve essere `public`; Spring Modulith e Java modules aiutano
4. **Va bene avere JDBC nel controller** — se il modulo è semplice, non forzare ports e adapters per uniformità
5. **Hexagonal/Onion sono validi** — ma solo dopo aver risposto alla domanda sulla decomposizione del dominio

---

## Citazioni notevoli

> "What we actually do when we design software is that we create a formalized bet on the changes that we're going to see in the future."  
> — Oliver Drotbohm

> "Domain-centric means the domain is in the center of the image. There is no decomposition strategy, no guidance, no help with that. Everyone is enthusiastic about the technical decomposition, but domain knowledge acquisition — maybe not."  
> — Oliver Drotbohm

> "You changed the direction of one arrow and insisted on interfaces. That's it. That's literally it. So every time I read the books and blog posts about onion architecture it left me underwhelmed — we use interfaces and dependency injection and that's an architecture now?"  
> — Oliver Drotbohm (sull'hexagonal e onion rispetto al layered)

> "If that vertical slice just needs to read from two tables and return JSON, why would I go ahead and have my adapter and port and my what-have-you? I could just wire a JDBC template into a controller and be done with it."  
> — Oliver Drotbohm

> "The system is never the sum of its parts but the product of their interactions."  
> — Russell Ackoff (citato nel talk)

---

## Riferimenti e risorse

- **Kent Beck** — talk a DDD Europe sulla definizione di costo del software come costo del cambiamento
- **Larry Constantine + Liz Yourdon** — "Fundamentals of Discipline of Computer Program and System Design" (1979) — fonte originale della definizione di coupling e cohesion; contiene già riferimenti a Conway's Law
- **Alistair Cockburn** — [Hexagonal Architecture](https://alistair.cockburn.us/hexagonal-architecture/) — proposta originale del 2005 (nome ufficiale: Ports & Adapters)
- **Jeffrey Palermo** — Onion Architecture (2008)
- **Russell Ackoff** — systems thinker; "the system is the product of interactions, not the sum of parts"
- **Simon Brown** — lavoro su C4 model e Software Architecture for Developers; influenza nella visualizzazione della struttura dei package
- **Spring Modulith** — framework per modularizzazione funzionale di applicazioni Spring Boot
- **J-Molecules** — libreria Java per annotare il codice con ruoli architetturali (aggregate root, primary port, ecc.)
- **Spring Tools (IntelliJ + Spring Tooling)** — tooling per visualizzare il progetto per moduli funzionali invece che per file system
- **Conway's Law** — menzione storica: già presente nel libro di Constantine del 1979, prima della popularizzazione nei talk su microservizi
