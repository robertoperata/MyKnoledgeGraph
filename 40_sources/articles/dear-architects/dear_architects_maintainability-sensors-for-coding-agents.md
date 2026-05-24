---
tags:
  - ai-agents
  - code-quality
  - static-analysis
  - software-architecture
  - developer-tooling
feature:
type: article
author: Birgitta Böckeler
source: https://martinfowler.com/articles/sensors-for-coding-agents.html
date: 2026-05-24
---

# Maintainability Sensors for Coding Agents

## Sunto

Gli agenti di codice AI possono generare rapidamente grandi quantità di codice, ma senza meccanismi di feedback automatico tendono ad accumulare debito tecnico in modo silenzioso. L'articolo introduce il concetto di "sensori" — controlli automatici che forniscono all'agente il feedback necessario per valutare e correggere il proprio output, trasformando la domanda vaga "sembra giusto?" in segnali concreti e ripetibili.

Birgitta Böckeler ha condotto il caso di studio su un'applicazione TypeScript/NextJS ricostruita interamente con assistenza AI. I sensori sono organizzati in tre livelli temporali: feedback immediato durante la sessione di coding (type checking, ESLint, Semgrep, dependency-cruiser, test di copertura, GitLeaks), esecuzione identica nella pipeline CI su infrastruttura pulita, e analisi periodiche pianificate per rilevare derive nel tempo (revisioni di sicurezza AI-driven, analisi di modularità, report sulla freschezza delle dipendenze).

Per le regole di linting, l'articolo evidenzia che gli agenti AI hanno fallure mode specifici che richiedono configurazioni ESLint dedicate: conteggio massimo degli argomenti di funzione, soglie di lunghezza dei file, complessità ciclomatica. Un'intuizione chiave è che i messaggi di lint personalizzati che guidano l'agente a fare scelte ragionate (con possibilità di sopprimere o aggiustare le soglie motivando la decisione) sono più efficaci delle regole binarie.

Le regole di dipendenza tramite dependency-cruiser hanno definito un'architettura a strati (`routes → services → clients + domain`) con regole che impediscono violazioni. L'analisi di accoppiamento con strumenti CLI ha però rivelato che i dati grezzi di coupling non sono sufficienti: l'agente interpretava male i pattern, etichettando come problematiche factory di dependency injection del tutto legittime. L'analisi semantica basata su LLM ("Modularity Skills") ha invece individuato problemi reali: duplicazione di codice, logica di autenticazione nel layer architetturale sbagliato, parametri core ripetuti in oltre 40 file.

Una scoperta critica dell'analisi di modularità basata su AI è che l'esecuzione ripetuta dell'analisi produce risultati diversi ogni volta, suggerendo che più passaggi LLM portano a una copertura più completa dei problemi. La combinazione di sensori computazionali (deterministici) e analisi semantica AI crea un effetto "garbage collection" che previene l'accumulo di debito tecnico.

## Immagini

![Layered architecture with dependency rules](https://martinfowler.com/articles/sensors-for-coding-agents/dependency-rules.png)

## Codice

Regola dependency-cruiser che vieta ai client di importare dai service:

```javascript
{
  name: "clients-no-services",
  from: { path: "^server/clients/" },
  to: { path: "^server/services/" }
}
```

Messaggio ESLint personalizzato che guida l'agente a una scelta ragionata invece di bloccare:

```
Make a judgment call about this. If you choose to not introduce a type,
suppress it with: // eslint-disable-next-line @typescript-eslint/no-explicit-any
If you decide to introduce a type, do so.
```
