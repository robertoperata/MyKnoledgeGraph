---
tags:
  - software-architecture
  - mvc
  - domain-driven-design
  - separation-of-concerns
  - modularization
feature:
type: article
author: kqr
source: https://entropicthoughts.com/mvc-mistake
date: 2026-05-10
---

# The MVC Mistake

## Sunto

L'articolo di kqr critica l'architettura MVC (Model-View-Controller) come fondamentalmente mal progettata: organizza il codice lungo confini tecnici invece che lungo concetti del dominio di business. Questa scelta viola i principi di separazione delle responsabilità stabiliti da David Parnas negli anni '70, che rimangono validi ancora oggi.

Il problema centrale è il "blast radius": quando si aggiunge una nuova funzionalità in un'architettura MVC classica (Presentation → Business → Persistence → Database), è necessario modificare ogni layer orizzontale. Se si aggiunge la funzionalità "commenti" a un'applicazione, si tocca il controller, il servizio business, il repository e il database schema — quattro livelli indipendenti. Questo non è separazione delle responsabilità: è il contrario. Parnas identificò due insight fondamentali: i concetti del dominio di business cambiano meno frequentemente delle soluzioni tecniche (strategie di caching, meccanismi di storage, algoritmi), e le funzionalità vengono aggiunte e rimosse più spesso dei concetti tecnici, ma rimangono stabili una volta implementate.

La visione di Dijkstra delle architetture a strati è radicalmente diversa da come viene applicata oggi: ogni layer dovrebbe essere una "macchina virtuale" per il layer superiore, con i layer inferiori che contengono definizioni stabili e raramente modificate. I layer superiori compongono funzionalità dai layer immediatamente inferiori. In un approccio domain-driven, l'aggiunta di una funzionalità aggiunge una colonna di logica di business senza toccare le fondamenta tecniche sottostanti.

L'autore sostiene il concetto di "locality of behaviour" proposto da Carson Gross e preferisce il termine "onion architecture" per descrivere correttamente questo stile, contrapposto all'architettura a layer corrotta dall'uso comune. Il termine "MVC" è diventato così generico e mal applicato da perdere il suo significato originale: oggi viene usato per qualsiasi organizzazione tecnica dei file, non per gestire dipendenze e responsabilità.

Un commento di Martin Bernstorff nella discussione offre un contrappunto interessante: l'organizzazione tecnologica può essere efficace per far rispettare i confini dei moduli (l'esempio dello stack undo/redo), suggerendo che approcci ibridi che combinano dimensioni di dominio e tecnologia possono essere appropriati in contesti specifici.

## Immagini

![Architettura MVC a layer orizzontali — ogni feature tocca tutti i layer](https://entropicthoughts.com/mvc-mistake-01.png)

![Architettura alternativa domain-driven — le feature sono colonne verticali indipendenti](https://entropicthoughts.com/mvc-mistake-02.png)
