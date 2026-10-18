---
tags:
  - system-design
  - distributed-systems
  - stateful-components
  - caching
  - event-driven
feature:
type: article
author: Sean Goedecke
source: https://www.seangoedecke.com/good-system-design/
date: 2026-10-18
---

# Everything I know about good system design

## Sunto

Sean Goedecke condensa in questo articolo (giugno 2025) una visione pragmatica e controcorrente del system design. La tesi centrale è che il buon design sia "noioso" e auto-cancellante: un sistema complesso è quasi sempre il segnale di design mancante, mentre i sistemi complessi che funzionano sono quasi sempre evoluti da sistemi semplici che funzionavano. Il design del sistema è all'architettura quello che il design del codice è alla programmazione: assemblare servizi nel modo giusto.

Il capitolo sullo **stato** è il più importante dell'articolo. I componenti stateful sono la parte più difficile del design perché possono entrare in uno stato non valido e non si ripristinano automaticamente come i servizi stateless. La regola pratica è: minimizzare i componenti stateful, tenere le scritture in un solo servizio e far chiamare gli altri quel servizio. Non duplicare lo stato tra database diversi se non strettamente necessario.

Per i **database**, i consigli sono operativi e concreti: mantenere gli schema leggibili, aggiungere indici che corrispondano alle query comuni, lasciare che il database faccia il lavoro (usare JOIN invece di caricare dati in memoria), evitare query in loop, mandare le letture alle repliche e throttlare i picchi di scrittura. L'anti-pattern più comune è spostare logica fuori dal database per "flessibilità", finendo per replicare inefficientemente ciò che il database fa già bene.

Sul tema del **caching**, Goedecke è controcorrente rispetto alla saggezza convenzionale: il caching aggiunge stato (e quindi complessità), quindi va evitato finché non si sono esaurite le ottimizzazioni sull'operazione sottostante. I senior engineer cachano il meno possibile; i junior tendono a cachare tutto come prima soluzione. Analogamente, gli eventi (event hub) vanno usati quando i produttori non si preoccupano dei consumatori; altrimenti, una chiamata API diretta è più semplice e più comprensibile.

Le sezioni su **hot path**, **logging** e **gestione dei failure** completano il quadro. I percorsi critici ad alto volume hanno meno soluzioni praticabili e falliscono in modo più drammatico, quindi vanno considerati per primi. Per il failure handling: killswitch, retry attenti con jitter, idempotency key e circuit breaker. Il rate limiting dovrebbe fallire open, l'autenticazione fallire closed.
