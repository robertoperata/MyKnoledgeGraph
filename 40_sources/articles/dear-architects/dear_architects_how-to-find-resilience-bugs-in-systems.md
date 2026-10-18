---
tags:
  - formal-methods
  - resilience
  - distributed-systems
  - architectural-bugs
  - state-space-exploration
feature:
type: article
author: Hillel Wayne
source: https://www.infoq.com/presentations/formal-methods-principles/
date: 2026-10-18
---

# How to Find Resilience Bugs in Systems that Don't Exist

## Sunto

Hillel Wayne, autore di *Logic for Programmers* e *Learn TLA+*, presenta in questa conferenza (QCon London, ottobre 2026) come i principi dei metodi formali possano aiutare i software engineer a individuare bug architetturali critici prima ancora che il sistema venga costruito. L'approccio si basa su un semplice esempio di "elaboratore di task" per mostrare come retry, marcatura dei lavori come completati e worker concorrenti introducano ciascuno difetti distinti e difficilmente rilevabili con i test tradizionali.

Il concetto centrale è quello di **proprietà**: un'affermazione booleana su un sistema corretto. Le proprietà forti (es. "nessuna doppia delivery") implicano quelle deboli, quindi una proprietà debole che fallisce consente di isolare i bug più velocemente di un singolo test forte che fallisce. Questo spiega anche la struttura della testing pyramid: molti unit test verificano proprietà granulari, ciascuna isolando una causa specifica.

La tecnica di **esplorazione dello spazio degli stati** modella il design come variabili macchina (sotto il controllo del sistema) e variabili mondo (esterne al sistema). Esplorando ogni stato raggiungibile si rivelano violazioni delle proprietà di safety e liveness. Wayne suggerisce di immaginare un "gremlin" che sceglie sempre il peggior risultato possibile ad ogni scelta non-deterministica: failure, concorrenza, outage. Questa mentalità avversariale aiuta a scoprire scenari che i test normali ignorano.

Un aspetto fondamentale riguarda le **assunzioni sull'ambiente**: ogni proprietà si regge su ipotesi sul contesto. Assumere strong consistency è diverso da assumere eventual consistency, e indebolire un'assunzione può silenziosamente rompere proprietà che sembravano garantite. Questo è particolarmente critico nelle architetture distribuite dove le garanzie del sistema sottostante cambiano (es. migrazione da un database ACID a un sistema NoSQL).

Per l'applicazione pratica, Wayne menziona strumenti come TLA+, Alloy, P, Quint e PRISM. Nell'era degli LLM, questi possono aiutare nella scrittura delle specifiche, ma con limitazioni: l'AI può generare specificazioni che sembrano corrette ma non catturano l'invariante voluto. Il vero valore dei metodi formali non è automatizzare la verifica, ma costringere il progettista a rendere esplicite le assunzioni implicite del design.

## Immagini

Non sono disponibili immagini pubbliche estratte dalla presentazione.
