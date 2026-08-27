---
tags:
  - genai
  - software-architecture
  - cognitive-debt
  - microservices
  - ai-coding-agents
feature:
type: article
author: Chris Richardson
source: https://microservices.io/post/architecture/2026/08/04/speed-limits-genai-coding-agents-autobahns-part-1.html
date: 2026-08-27
---

# Speed limits, GenAI coding agents and Autobahns - part 1: how fast should you go?

## Sunto

Gli agenti di coding basati su GenAI consentono ai team di generare codice a velocità straordinarie. Tuttavia, ogni membro del team è un essere umano con capacità cognitive limitate, e la velocità con cui può leggere, comprendere e verificare il codice generato è intrinsecamente vincolata. Quando un agente genera codice più rapidamente di quanto il team riesca ad assimilarlo, si accumula un **debito cognitivo**: codice viene committato e deployato senza che nessuno lo comprenda davvero, e il gap tra ciò che il sistema fa e ciò che il team sa che fa continua ad allargarsi.

L'articolo affronta la domanda centrale che ogni team dovrebbe porsi: **a che velocità dovremmo lasciare andare l'agente sul nostro codice?** Chris Richardson utilizza la metafora della guida automobilistica per strutturare il ragionamento. Così come un guidatore non porta un'auto alla sua velocità massima per tre ragioni fondamentali — sicurezza stradale, limiti di velocità imposti, e congestione del traffico — allo stesso modo un team deve calibrare la velocità di generazione del codice rispetto al contesto specifico.

La metafora dell'**Autobahn** tedesco viene usata per illustrare che velocità elevate possono essere sicure quando le condizioni lo permettono: infrastruttura stradale progettata per alte prestazioni, standard veicolo adeguati, e corrispondenza tra velocità e circostanze. La chiave è che l'Autobahn non è privo di regole, ma di regole rigide universali: è la situazione contestuale a determinare il limite appropriato. Questo si traduce in software nel concetto che certi tipi di codice (boilerplate, adapter, test, configurazioni) possono essere generati ad alta velocità, mentre il codice di dominio complesso, la logica di business critica e il design architetturale innovativo richiedono un ritmo molto più lento.

La **Formula 1** viene invece scartata come modello per lo sviluppo software: in F1 la velocità massima è l'obiettivo assoluto, ma si opera in condizioni altamente controllate, con expertise professionale specializzata e senza l'obiettivo di produrre qualcosa di nuovo e permanentemente manutenuto. Nessuna di queste condizioni è applicabile allo sviluppo software tipico.

Il framework proposto per determinare la velocità sicura si basa sulla comprensione delle caratteristiche del codice da generare: la complessità del dominio, il livello di accoppiamento, i pattern architetturali adottati (come l'architettura esagonale), e la classificazione del lavoro secondo il framework **Cynefin** (lavoro chiaro, complicato o complesso). Un componente di tipo "clear" con alta maturità e bassa complessità può essere generato rapidamente; un componente nel dominio "complex" richiede un approccio esplorativo e un ritmo molto più controllato.

## Immagini

![A speedometer whose dial is divided into a green zone for generated boilerplate, adapters, tests and config, an amber zone for supporting code and business logic, and a red zone for core domain, complex logic and novel design](https://microservices.io/i/genai/code-velocity-speedometer.jpg)
