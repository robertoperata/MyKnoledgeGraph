---
tags:
  - software-modularity
  - software-architecture
  - coupling
  - bounded-contexts
  - technical-debt
feature:
type: article
author: Vlad Khononov
source: https://github.com/vladikk/modularity
date: 2026-04-19
---

# Modularity Skills: Progettare Sistemi Modulari nell'Era dell'AI

## Sunto

Il progetto "Modularity Skills" di Vlad Khononov è un plugin per Claude Code che applica il **Balanced Coupling Model** per analizzare e progettare sistemi software modulari. Il contesto di nascita è significativo: nell'era dello sviluppo assistito da AI, il debito tecnico si accumula molto più rapidamente che in passato, e la mancanza di una disciplina architetturale esplicita porta rapidamente a sistemi ingestibili.

Il problema centrale che il plugin risolve è la **gestione del coupling** tra componenti software. Il coupling non è intrinsecamente negativo — è necessario per permettere ai componenti di collaborare — ma diventa problematico quando è sbilanciato. Il Balanced Coupling Model valuta il coupling su tre dimensioni: **Integration Strength** (la quantità di conoscenza condivisa tra componenti, da intrusive a functional, model, contract), **Distance** (il costo sociotecnico della co-evoluzione, valutato su struttura del codice, team e runtime), e **Volatility** (la probabilità business-driven che un componente cambi).

La regola di bilanciamento è espressa come formula logica: `(STRENGTH XOR DISTANCE) OR NOT VOLATILITY`. In pratica: se due componenti si accoppiamo strettamente (alta integration strength), devono essere vicini (bassa distance); se sono lontani, il coupling deve essere leggero. I componenti ad alta volatilità devono essere isolati con coupling minimo per permettere cambiamenti indipendenti. Quando questa regola è violata, si accumulano costi di manutenzione sproporzionati.

Il plugin offre due skill principali: `/modularity:review` per analizzare codebase esistenti e identificare gli sbilanciamenti di coupling, e `/modularity:design` per progettare architetture modulari a partire da requisiti. Entrambe le skill operano a livello architetturale — non al livello di singole linee di codice — e producono report Markdown con raccomandazioni concrete e contratti di integrazione tra moduli. Richiedono Claude Opus 4.5 o successivo per la qualità di ragionamento necessaria, segno che questo tipo di lavoro architetturale rappresenta uno dei task genuinamente complessi per un LLM.
