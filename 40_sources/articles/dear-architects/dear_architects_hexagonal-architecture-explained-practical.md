---
tags:
  - hexagonal-architecture
  - ports-and-adapters
  - domain-driven-design
  - spring-boot
  - clean-architecture
feature:
type: article
author: Kinle Greshka
source: https://www.thoughtworks.com/insights/blog/architecture/hexagonal-architecture-explained-practical-example
date: 2026-05-31
---

# Hexagonal Architecture Explained Through a Practical Example

## Sunto

L'articolo di Kinle Greshka per Thoughtworks usa un'applicazione di consegna pizza come esempio concreto per spiegare l'architettura esagonale (nota anche come "ports and adapters"). Il problema centrale che questa architettura risolve è la tendenza delle applicazioni in crescita a far annegare le regole di business sotto preoccupazioni infrastrutturali — SDK, integrazioni API, logica di mapping JSON. L'architettura esagonale separa nettamente la logica di business (l'interno) dagli strumenti infrastrutturali (l'esterno), con il principio fondamentale che la logica di business non deve mai dipendere direttamente da componenti infrastrutturali come database o processori di pagamento.

Il meccanismo centrale è la distinzione tra **porte** e **adattatori**. Le porte definiscono ciò di cui l'applicazione ha bisogno senza specificare i dettagli di implementazione — sono interfacce che esprimono le intenzioni del dominio. Gli adattatori connettono l'applicazione a strumenti specifici: gli adattatori *inbound* gestiscono le richieste esterne (controller HTTP, consumer Kafka), mentre gli adattatori *outbound* gestiscono le dipendenze esterne (repository database, client di pagamento). Questa separazione permette di sostituire componenti infrastrutturali senza toccare la logica di business, e di testare workflow completi senza eseguire transazioni reali.

L'organizzazione interna del core si articola in due strati. Il **domain layer** contiene entità e value object e fa rispettare gli invarianti di business: le entità mantengono un'identità nel tempo (es. "Ordine #542"), mentre i value object sono definiti solo dai loro attributi (es. "extra mozzarella"). Gli **aggregate root** sono i punti di accesso singoli che controllano tutte le modifiche alle entità correlate, garantendo che le regole di business rimangano sempre rispettate. L'**application layer** coordina i workflow senza contenere regole di business proprie.

L'integrazione con il **Domain-Driven Design** è un punto centrale dell'articolo. La matrice decisionale fornita dall'autore aiuta a determinare se la logica appartiene alle entità di dominio, ai domain service o agli application service. Le entità di dominio contengono regole che riguardano il loro stato interno; i domain service contengono regole che coinvolgono più entità o richiedono accesso a risorse esterne; gli application service orchestrano il flusso senza contenere logica di business. Questa chiarezza riduce il rischio di anemic domain model e di logica di business fuggita negli strati sbagliati.

I benefici pratici elencati sono tre: sostituire componenti infrastrutturali (es. cambiare database, payment provider) senza impatto sul core; testare workflow completi in isolamento con test unitari veloci e senza I/O; isolare e gestire i fallimenti dei sistemi esterni più efficacemente. L'articolo fa riferimento a un'implementazione completa con Spring Boot disponibile su GitHub nel repository PizzaCo Hexagonal Sample, che fornisce un punto di partenza concreto per applicare i pattern descritti.
