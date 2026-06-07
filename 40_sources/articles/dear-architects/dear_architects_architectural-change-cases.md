---
tags:
  - software-architecture
  - evolutionary-architecture
  - architecture-decision-records
  - agile
  - ai-generated-code
feature:
type: article
author: Pierre Pureur, Kurt Bittner
source: https://www.infoq.com/articles/architectural-change-cases/
date: 2026-06-07
---

# Architectural Change Cases: a Practical Tool for Evolutionary Architectures

## Sunto

Il concetto di **architectural change case** (caso di cambiamento architetturale) nasce dall'osservazione che nessuna architettura software rimane invariata nel tempo: le esigenze di business evolvono, le tecnologie cambiano e l'ambiente esterno si trasforma continuamente. Pierre Pureur e Kurt Bittner propongono un framework metodologico per anticipare e gestire proattivamente queste evoluzioni, spostando il team da una postura reattiva a una proattiva rispetto ai cambiamenti architetturali.

Un change case identifica potenziali modifiche alle assunzioni su cui si basa l'architettura corrente, che potrebbero avere un impatto significativo sulle decisioni già prese. A differenza degli Architecture Decision Records (ADR), che documentano decisioni passate, e dell'Architecture Tradeoff Analysis Method (ATAM), che valuta l'architettura attuale rispetto ai requisiti presenti, i change case guardano verso il futuro: il loro scopo è articolare la *possibilità* di un cambiamento, senza necessariamente definire la soluzione alternativa. Ogni change case include: la variazione probabile nei requisiti di qualità, la probabilità che si verifichi, l'elenco delle decisioni architetturali impattate, le opzioni di risoluzione possibili e una stima relativa del costo (S, M, L, XL).

L'articolo identifica cinque categorie principali di change case: cambiamenti alle interfacce di sistemi esterni, sostituzione di sottosistemi maggiori per obsolescenza o cambio fornitore, transizioni infrastrutturali (migrazione cloud, variazioni di latency), cambi di modello di business (fallimento di un MVP, nuovi mercati), e vulnerabilità di sicurezza introdotte da fattori esterni. L'esempio pratico presentato riguarda una compagnia assicurativa che lancia un prodotto on-demand per case vacanza: i change case analizzati includono l'adozione 50% superiore alle proiezioni (costo L), l'espansione della copertura a camper e barche con necessità di riscrivere il motore di underwriting (costo XL), e l'espansione multi-stato con requisiti regolatori eterogenei (costo L).

I change case si integrano naturalmente nello sviluppo iterativo agile: durante la pianificazione delle iterazioni, il team valuta sia i trade-off architetturali sia la loro reversibilità. Il framework si allinea con le pratiche di *continuous architecture* e *evolutionary architecture*, promuovendo la costruzione di flessibilità deliberata piuttosto che l'accumulo di debito tecnico latente. Le **fitness function** forniscono il meccanismo di misurazione empirica per validare se gli esperimenti raggiungono i miglioramenti di qualità attesi senza effetti collaterali negativi.

Un paragrafo significativo è dedicato ai rischi specifici del codice generato da AI: la dipendenza da un modello o fornitore di AI che cessa l'attività è esplicitamente menzionata come change case da considerare. La strategia di mitigazione consigliata è mantenere un repository documentale completo (requisiti, design, esempi di codice, test di accettazione, decisioni precedenti) che serve sia come contesto arricchito per gli assistenti AI sia come garanzia di riproducibilità in caso di sostituzione del modello.

## Immagini

![Architectural Change Cases Framework](https://res.infoq.com/articles/architectural-change-cases/en/resources/1architectural-change-cases-a-practical-tool-1748970697440.jpg)
