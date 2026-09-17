---
tags:
  - jpa
  - hibernate
  - performance-tuning
  - connection-pooling
  - spring-data-jpa
feature:
type: article
author: Vlad Mihalcea
source: https://vladmihalcea.com/meet-jos-roseboom/
date: 2026-09-17
---

# Meet Jos Roseboom

## Sunto

Vlad Mihalcea ha intervistato Jos Roseboom, contractor indipendente specializzato in ottimizzazione delle prestazioni Java, dopo aver assistito al suo talk su JPA performance tuning a JavaZone 2026. L'articolo esplora i principali errori di performance nelle applicazioni che utilizzano JPA e Hibernate, con particolare attenzione alla gestione delle connessioni al database e ai problemi causati da un utilizzo superficiale dell'ORM.

Il primo caso reale discusso riguarda un sistema rimasto completamente irresponsivo perché tutte le connessioni al database erano acquisite da richieste in attesa di un sistema esterno in difficoltà. Questo scenario mette in evidenza come la scarsità di connessioni possa diventare un punto di failure a cascata: se il connection pool si esaurisce, nessuna operazione sul database può procedere, anche quelle non correlate al sistema esterno problematico.

Il secondo problema comune, ancora più frequente, nasce dal misconcezione che l'utilizzo di Hibernate esoneri i developer dalla conoscenza di SQL. Jos descrive applicazioni che diventano inaccettabilmente lente perché i dati vengono raccolti seguendo le associazioni tra entità — un pattern che genera decine o centinaia di query dove basterebbe una sola query SQL efficiente. Questo problema, invisibile su macchina di sviluppo, diventa un incubo in produzione con dati reali.

Jos introduce il concetto di "budget di connessioni": il numero massimo di connessioni al database deve essere considerato una risorsa condivisa tra tutte le istanze dell'applicazione. Non è possibile configurare ogni istanza con un pool ampio assumendo che tutte le connessioni siano disponibili simultaneamente. È fondamentale mantenere le connessioni il minor tempo possibile, rilasciandole non appena l'interazione con il database è completata.

Per nuovi progetti su database relazionali, Jos sceglierebbe ancora Spring Data JPA con Hibernate, apprezzandone la validazione delle query al momento dello startup dell'applicazione (che trasforma typo in JPQL da errori in produzione a fail-fast) e la leggibilità di JPQL. La raccomandazione chiave rimane però quella di non considerare Hibernate come un modo per evitare di imparare SQL: comprendere cosa accade al livello inferiore è indispensabile per scrivere applicazioni performanti.

## Codice

Esempio di annotazione `REQUIRES_NEW` che può causare deadlock nel connection pool quando usata con propagazione di transazioni annidate:

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void methodWithNewTransaction() {
    // Una nuova transazione viene aperta, richiedendo una seconda connessione dal pool
    // Se il pool è al limite, questo può causare un deadlock
}
```

Gestione corretta del `DataSource` per non trattenere connessioni inutilmente — evitare operazioni remote o lente all'interno di una transazione attiva:

```java
@Transactional
public void saveData(Data data) {
    // Operazioni DB qui — connessione acquisita e rilasciata rapidamente
    repository.save(data);
    // Non fare chiamate a sistemi esterni qui dentro!
}

// Meglio: chiamata esterna fuori dalla transazione
public void processAndSave(Data data) {
    ExternalResult result = externalService.call(); // fuori dalla transazione
    saveData(transform(result));
}
```
