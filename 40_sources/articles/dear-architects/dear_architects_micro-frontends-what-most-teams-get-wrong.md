---
tags:
  - micro-frontends
  - architettura-frontend
  - luca-mezzalira
  - architettura-distribuita
  - modularita
feature:
type: article
author: Luca Mezzalira
source: https://www.youtube.com/watch?v=PbSYWBx-gVw
date: 2026-05-25
---

# Micro-Frontends — Luca Mezzalira on What Most Teams Get Wrong

## Sunto

Luca Mezzalira, autore di "Building Micro-Frontends" (O'Reilly) e Chief Architect presso Amazon Web Services, esplora in questo talk i principali errori che i team commettono quando adottano l'architettura micro-frontend. L'approccio micro-frontend estende i principi dei microservizi al layer di presentazione, consentendo a team indipendenti di sviluppare, testare e deployare singole parti dell'interfaccia utente in autonomia.

Il primo errore critico che Mezzalira identifica è la scelta errata del modello di composizione. I team spesso optano per la composizione lato client (attraverso JavaScript a runtime) senza considerare le implicazioni in termini di performance, bundle size e dipendenze condivise. Mezzalira illustra come la composizione lato server o edge sia spesso più adatta per applicazioni con requisiti di SEO e performance, mentre la composizione lato client si adatta meglio ad applicazioni SPA altamente interattive. La scelta deve essere guidata dai requisiti non funzionali, non dalle preferenze tecnologiche del team.

Il secondo errore riguarda la gestione dello stato condiviso e la comunicazione tra micro-frontends. I team tendono a replicare pattern di stato globale (come Redux o Zustand condiviso) tra i micro-frontends, creando un accoppiamento implicito che vanifica i benefici dell'autonomia. Mezzalira raccomanda invece la comunicazione tramite custom events del browser, URL state, o integrazione tramite backend-for-frontend, mantenendo ogni micro-frontend stateful solo per la propria porzione di dominio. Il "Single SPA" pattern è utile ma richiede disciplina per evitare di diventare un monolite distribuito.

Un terzo errore fondamentale è la mancata definizione chiara dei confini di dominio prima di decomporre l'interfaccia. I team spesso suddividono l'UI in base alla struttura dei team o alle tecnologie disponibili, invece di seguire i bounded context del dominio. Mezzalira suggerisce di applicare i principi del Domain-Driven Design anche al frontend: ogni micro-frontend dovrebbe corrispondere a un dominio coeso, avere un team dedicato e un contratto stabile con gli altri micro-frontends. La scelta del confine sbagliato porta a micro-frontends che si sovrappongono, con duplicazione di codice e necessità di coordinamento continuo.

Mezzalira dedica parte del talk alle sfide pratiche legate alla developer experience e al testing. I micro-frontends rendono più complesso il testing di integrazione end-to-end, richiedono contratti API stabili tra team e necessitano di strumenti di orchestrazione (shell application) ben progettati. Un design kit condiviso (component library) può garantire coerenza visiva senza creare dipendenze tecnologiche, purché versionato in modo indipendente. Il talk conclude con l'invito a valutare i micro-frontends solo quando i benefici di autonomia dei team superano chiaramente i costi di complessità operativa introdotti.

