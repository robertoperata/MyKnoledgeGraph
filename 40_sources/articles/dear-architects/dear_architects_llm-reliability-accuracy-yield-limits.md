---
tags:
  - llm
  - affidabilità
  - testing
  - nlp
  - valutazione-modelli
feature:
type: article
author: Pushpdeep Mishra
source: https://www.infoq.com/articles/llm-reliability-layers-limits-accuracy/
date: 2026-11-15
---

# LLM Reliability Layers Under Test: Accuracy, Yield, and the Limits of Verification

## Sunto

Questo articolo di Pushpdeep Mishra e Wejdan Bagais (rivisto da Hien Luu) presenta i risultati di un esperimento sistematico su sei livelli di affidabilità applicati a un LLM open-weight per l'estrazione di reazioni avverse a farmaci dal corpus CADEC (forum posts) con codifica SNOMED CT. L'ipotesi di partenza era che ogni layer aggiuntivo migliorasse l'accuratezza; i risultati reali hanno mostrato qualcosa di più sottile e istruttivo.

**Il paradosso yield/accuracy**: ogni layer di verifica non migliora direttamente la qualità delle risposte — le filtra. Aumenta l'accuratezza di ciò che viene rilasciato trattenendo le risposte incerte, ma riduce il "yield" (la percentuale di input processati con successo). Gli autori hanno quindi misurato sempre entrambe le metriche, rivelando tradeoff espliciti che la sola accuratezza nasconde. Questa distinzione è fondamentale in sistemi di produzione dove coprire tutti i casi ha un costo di business reale.

**Il problema del richiamo vs. riconoscimento**: chiedere al modello di ricordare codici SNOMED produceva codici inventati nel 13-18% dei casi (allucinazioni di identificatori). Chiedere invece di scegliere tra opzioni recuperate (retrieval-augmented) ha aumentato l'accuratezza di circa 10x. La lezione è generale: mai chiedere a un LLM di generare identificatori strutturati quando è possibile un approccio di riconoscimento/selezione.

**La detection è il collo di bottiglia critico**: nessun layer downstream può recuperare una menzione che l'estrattore iniziale ha mancato. Quasi la metà delle menzioni annotate non veniva trovata dal modello, e i guadagni maggiori provenivano dall'allineamento con le convenzioni di annotazione del corpus. Questo suggerisce che in sistemi a pipeline il miglioramento deve iniziare dal componente di detection, non dall'aggiunta di layer di verifica a valle.

**Variabilità e metriche opache**: quattro delle cinque famiglie di modelli testate riproducevano l'output in modo deterministico a temperature zero; la quinta concordava su solo il 70% delle menzioni tra tre esecuzioni. Inoltre, una metrica difettosa non rilevava codici non verificati che venivano rilasciati come verificati — un bug silenzioso che avrebbe impattato la qualità senza trigger visibili.

Gli autori concludono con sei pratiche operative: (1) analizzare i dati prima di aggiungere layer, (2) non chiedere mai al modello un identificatore, (3) ancorare la confidence a controlli deterministici, (4) misurare il "noise floor" del sistema, (5) trattare il disaccordo tra run come una lista di todo, (6) verificare che ogni campo scritto da un layer venga effettivamente letto da quello successivo.
