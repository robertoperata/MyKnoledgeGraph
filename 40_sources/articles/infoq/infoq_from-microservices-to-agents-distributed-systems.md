---
tags:
  - agentic-ai
  - microservices
  - distributed-systems
  - architectural-patterns
  - observability
feature:
type: article
author: Mallika Rao
source: https://r.mail.infoq.com/mk/cl/f/sh/7nVU1aA2ng9SOTtwsmE8wFPzzRuutLd/DdBdwSjyUmIL
date: 2026-11-28
---

# From Microservices to Agents: The Next Evolution of Distributed Systems

## Sunto

Mallika Rao argomenta che l'AI agentica rappresenta la prossima fase nell'evoluzione dei sistemi distribuiti. Mentre i microservizi decompongono la funzionalità in servizi indipendentemente deployabili con bounded context e flusso di controllo prevedibile, i sistemi agentici decompongono le **decisioni**. I percorsi di esecuzione non sono più fissi a design time: emergono a runtime dal contesto, dagli obiettivi e dai risultati intermedi, rendendo questi sistemi più difficili da prevedere, tracciare e debuggare.

Rao identifica tre differenze strutturali rispetto ai microservizi tradizionali. Prima: il **flusso di controllo** è deciso a runtime, quindi richieste diverse possono seguire percorsi completamente diversi attraverso il sistema. Seconda: la **gestione dello stato** diventa continua — la storia della conversazione, i piani, i documenti recuperati e gli output degli strumenti evolvono attraverso i passi dell'esecuzione, assomigliando più a un coordinamento di stato distribuito che a semplici chiamate request-reply. Terza: la **semantica dei fallimenti** cambia radicalmente: invece di errori binari (successo/fallimento), i sistemi agentici producono degradazioni graduali della qualità decisionale che possono passare inosservate finché non causano problemi significativi.

A fronte di queste differenze, emergono nuovi pattern architetturali. La separazione tra **planning ed execution** — dove un planner interpreta l'intento dell'utente mentre executor deterministici gestiscono recupero, trasformazione e ranking — diventa un elemento fondamentale del design. Gli agenti specializzati migliorano la modularità ma aggiungono overhead di coordinazione. I cicli di valutazione all'interno dell'execution loop abilitano raffinamento e retry, ma richiedono guardrail espliciti: budget di latenza, limiti di iterazione e soglie di confidenza. I **tool abstraction layer** disaccoppiano il ragionamento dalle implementazioni dei servizi e possono evolvere in una capability platform condivisa.

Rao descrive i failure mode specifici dei sistemi agentici, molti dei quali derivati dall'esperienza con sistemi di ricerca e raccomandazione: degradazione decisionale a cascata, amplificazione dei retry guidata da giudizi di qualità soggettivi, degradazione silenziosa della qualità mentre latenza e error rate rimangono normali, latenza di coordinazione accumulata nel tempo e state drift attraverso aggiornamenti incrementali del contesto. La difficoltà è che molti di questi problemi non sono rilevabili con le metriche tradizionali di disponibilità e latenza.

Le raccomandazioni pratiche di Rao invitano a trattare la **qualità decisionale** come una preoccupazione di affidabilità di primo livello, al pari di latenza, disponibilità e throughput. Le pratiche consigliate includono: tracing a livello di decisione (selezione degli strumenti e passi di ragionamento), monitoraggio comportamentale dei retry e dei fallback rate come segnali precoci di drift, autonomia limitata e ben definita, architetture ibride con pipeline di fallback deterministiche, e rollout progressivo attraverso shadow execution. La conclusione è che l'architettura agentica estende i principi dei sistemi distribuiti a un nuovo livello di astrazione, combinando flessibilità con disciplina ingegneristica.
