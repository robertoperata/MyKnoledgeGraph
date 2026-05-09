---
tags:
  - architecture
  - ddd
  - microservices
  - distributed-systems
type: article
author: claytonsiemens77
source: https://learn.microsoft.com/en-us/azure/architecture/patterns/anti-corruption-layer
date: 2022-07-28
---

# Anti-corruption Layer pattern

## Sunto

L'*Anti-corruption Layer* (ACL) è un pattern architetturale che consiste nell'inserire uno strato di façade o adapter tra due sottosistemi con semantiche diverse, in modo che ciascuno possa evolversi indipendentemente senza "inquinare" il modello dell'altro. Il pattern è stato originariamente descritto da **Eric Evans** in *Domain-Driven Design* ed è uno dei pattern fondamentali per le migrazioni graduali e per l'integrazione con sistemi legacy o sistemi esterni non controllabili dal team.

Il problema che risolve nasce da una situazione comune: un'applicazione moderna deve continuare a interagire con un sistema legacy che ha schemi dati contorti, API obsolete o modelli concettuali incompatibili. Se la nuova applicazione si adatta direttamente alle strutture del sistema legacy, il suo design viene progressivamente corrotto — costretta ad adottare concetti, protocolli e modelli che non farebbe propri in un contesto pulito. Questo vale non solo per i sistemi legacy, ma per qualunque sistema esterno su cui il team non ha controllo.

La soluzione è **isolare i due sottosistemi tramite uno strato dedicato alla traduzione**. L'Anti-corruption Layer contiene tutta la logica di conversione tra i due modelli: le richieste provenienti dal sistema moderno usano il modello e l'architettura del sistema moderno; le chiamate verso il sistema legacy si conformano al modello del legacy. Nessuno dei due sistemi deve conoscere le strutture interne dell'altro. Lo strato può essere implementato come componente interno all'applicazione oppure come servizio indipendente deployato separatamente.

Il pattern si applica tipicamente in due scenari: (1) migrazioni graduali in cui funzionalità vengono spostate da un sistema legacy a uno moderno in più fasi, mantenendo nel frattempo l'integrazione operativa; (2) integrazione tra sottosistemi con semantiche fondamentalmente diverse che devono comunque comunicare. **Non è adatto quando le differenze semantiche tra i sistemi sono minime**, perché in quel caso il costo dello strato aggiuntivo non è giustificato.

I trade-off principali da considerare riguardano la latenza introdotta (ogni chiamata transita attraverso lo strato), il costo di mantenimento (è un servizio aggiuntivo che va monitorato, rilasciato e configurato), la scalabilità separata, e la possibile necessità di decomporlo in più ACL se la complessità cresce o se coinvolge tecnologie eterogenee. Va anche deciso esplicitamente se l'ACL è una struttura permanente o temporanea destinata ad essere eliminata al completamento della migrazione.

---

## Struttura

```
Subsystem A  ──────────►  Anti-Corruption Layer  ──────────►  Subsystem B
  (modello A)           (traduzione A ↔ B)              (modello B)
```

- Le comunicazioni tra **Subsystem A** e l'ACL usano sempre il data model e l'architettura di Subsystem A.
- Le chiamate dall'ACL verso **Subsystem B** si conformano al data model e ai metodi di Subsystem B.
- Tutta la logica di traduzione risiede nell'ACL — nessun dettaglio del sistema opposto trapela.

---

## Considerazioni e decisioni di design

| Aspetto | Domande da porsi |
|---|---|
| **Latenza** | Quanto impatta l'hop aggiuntivo? È accettabile nel contesto d'uso? |
| **Scalabilità** | L'ACL deve scalare indipendentemente? È un collo di bottiglia? |
| **Decomposizione** | Serve un unico ACL o più ACL per tecnologie/domini diversi? |
| **Osservabilità** | Come viene integrato nel monitoring, nei processi di release e di configurazione? |
| **Consistenza dati** | Le transazioni e la consistenza sono mantenibili attraverso lo strato? |
| **Scope** | L'ACL gestisce tutta la comunicazione tra i sistemi o solo un sottoinsieme di feature? |
| **Ciclo di vita** | È permanente o temporaneo (da rimuovere a migrazione completata)? |

---

## Relazione con Well-Architected Framework (Azure)

| Pilastro | Contributo del pattern |
|---|---|
| **Operational Excellence** | Il design dei nuovi componenti rimane indipendente dalle implementazioni legacy (modelli dati diversi, business rule diverse). Riduce il technical debt nei nuovi componenti pur mantenendo il supporto ai componenti esistenti. |

---

## Link esterni

- [Strangler Fig pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/strangler-fig) — pattern complementare per migrazioni graduali: rimpiazza progressivamente le funzionalità del sistema legacy
- [Messaging Bridge pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/messaging-bridge) — pattern correlato per l'integrazione tramite messaggistica
- *Domain-Driven Design* di Eric Evans — libro in cui il pattern è stato originariamente descritto

---

## Immagini

- ![Diagramma Anti-Corruption Layer](https://learn.microsoft.com/en-us/azure/architecture/patterns/_images/anti-corruption-layer.png) — Schema con Subsystem A, ACL e Subsystem B: le frecce mostrano che il modello di ciascun sistema rimane confinato al proprio lato dello strato
