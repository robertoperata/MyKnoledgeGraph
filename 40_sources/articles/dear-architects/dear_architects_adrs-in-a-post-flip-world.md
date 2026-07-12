---
tags:
  - architecture-decision-records
  - agentic-coding
  - software-architecture
  - ai-assisted-development
  - technical-documentation
feature:
type: article
author: Garry Shutler
source: https://gshutler.com/2026/06/adrs-in-a-post-flip-world/
date: 2026-07-12
---

# ADRs in a Post-Flip World

## Sunto

L'articolo di Garry Shutler esplora come il coding agente abbia trasformato in modo fondamentale le pratiche legate agli Architecture Decision Records (ADR). L'autore riprende il concetto del "Pareto Flip" che ha introdotto in precedenza: nell'era del coding assistito dall'AI, l'implementazione è diventata economicamente irrisoria, spostando il peso cognitivo verso la fase di pensiero e progettazione.

Tradizionalmente gli ADR svolgevano una doppia funzione: conservare il contesto delle decisioni per riferimento futuro e fungere da gate di controllo dei costi, imponendo una pianificazione rigorosa prima di impegnarsi in lavori di implementazione dispendiosi. Shutler descrive questo meccanismo come un sistema di "ordini d'acquisto" in cui i team confrontano le opzioni sulla carta per evitare sprechi di risorse reali.

Con l'implementazione diventata un costo quasi trascurabile, la necessità di gate di autorizzazione preventivi svanisce. I team possono creare e scartare più spike esplorativi senza significative perdite di risorse. Shutler afferma che, paradossalmente, rimuovere questi gate non riduce il rigore decisionale — al contrario, lo aumenta: confrontare tre spike realmente costruiti produce decisioni più solide rispetto a tre opzioni confrontate solo in astratto. "La decisione è ora ancorata a cose che abbiamo costruito, non a cose che abbiamo immaginato."

Un'evoluzione notevole riguarda l'estensione dello spiking al codice lato integratore: è ora possibile costruire empiricamente anche il codice che gli sviluppatori esterni scriverebbero utilizzando un'API. Questa forma di valutazione della developer experience non era precedentemente praticabile a causa dei costi. Anche la produzione degli ADR si trasforma: gli agenti AI possono ora bozzare i documenti a partire da conversazioni registrate, con gli esseri umani che contribuiscono giudizio e raffinamento.

La struttura del formato ADR rimane invariata — ciò che cambia è il *quando* viene scritto e la qualità dell'evidenza che lo supporta. Shutler conclude evidenziando che la tempistica della scrittura è sempre stata guidata dall'economia, non dall'essenza dell'artefatto stesso. Il valore del record persiste indipendentemente dal momento in cui viene prodotto, servendo ora ugualmente bene sia i futuri lettori umani che gli agenti AI.

## Codice

Nessun esempio di codice presente nell'articolo.
