---
tags:
  - micro-frontends
  - frontend-architecture
  - migration
  - team-topology
  - module-federation
feature:
type: article
author: Luca Mezzalira
source: https://gitnation.com/contents/migrating-to-micro-frontends
date: 2026-07-12
---

# Designing a Migration to Micro-Frontends

## Sunto

Luca Mezzalira, autore di *Building Micro-Frontends*, presenta in questo talk di JSNation 2026 una guida pratica per le organizzazioni che intendono migrare verso l'architettura micro-frontend, basandosi su esperienze reali con centinaia di aziende. Il punto di partenza è una distinzione fondamentale spesso fraintesa: i **componenti** sono progettati per la riusabilità e la coerenza del design system, mentre i **micro-frontend sono ottimizzati per l'indipendenza** dei team e la capacità di effettuare deploy multipli al giorno.

Prima di scrivere una sola riga di codice, Mezzalira raccomanda l'uso del **Micro-Frontends Canvas**, uno strumento open source per la pianificazione architetturale che si concentra sulla definizione dei confini (boundaries), i requisiti di validazione, i vincoli organizzativi, le dipendenze esterne e le metodologie di comunicazione. Le architetture comuni includono server-side rendering con application shell, approcci client-side rendering, pattern UI composer con distribuzione di file statici e integrazione CDN per sicurezza e routing.

La strategia di migrazione fondamentale è evitare i "big bang rewrite" a favore di un approccio incrementale. L'**Edge Compute** permette di routing graduale del traffico verso i nuovi micro-frontend; le **canary release** con key-value store (ad esempio il 10% del traffico verso il nuovo micro-frontend) consentono validazione progressiva; l'**application shell** funge da fondazione stabile dell'intero ecosistema. Per la condivisione dei dati tra micro-frontend, Mezzalira distingue tra query string per dati effimeri, cookie per autenticazione, backend API per preferenze e **event emitter** per comunicazione loosely-coupled (preferiti agli eventi DOM custom per migliore incapsulamento).

Il routing segue un modello a due livelli: il routing globale a livello di application shell gestisce i primi livelli URL, mentre il routing locale all'interno di ciascun micro-frontend gestisce la navigazione interna, riducendo le dipendenze esterne e il coupling al deploy. Le dipendenze condivise (come React) si gestiscono tramite import maps o module federation con scoping, permettendo a team diversi di mantenere versioni diverse della stessa libreria in coesistenza (es. React 17 e React 18).

L'articolo mette anche in evidenza le situazioni in cui i micro-frontend **non** sono appropriati: team piccoli in fase di startup, progetti ancora in ricerca del product-market fit, o situazioni meglio servite da framework monolitici moderni come Next.js, Astro o Qwik. L'integrazione con l'AI generativa viene affrontata tramite harness engineering: feedforward mechanism (guide e sensori), linter, analisi statica e fitness function per vincolare la generazione di codice alla corretta definizione dei boundary. La chiave per l'allineamento manageriale è presentare la riusabilità in termini di valore di business e compliance normativa, piuttosto che benefici tecnici.
