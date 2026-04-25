---
tags:
  - ai-coding-assistants
  - context-engineering
  - ai-agents
  - harness-engineering
  - sicurezza-ai
feature:
type: article
author: Birgitta Böckeler
source: https://www.infoq.com/presentations/ai-coding-assistants
date: 2026-04-25
---

# State of Play: AI Coding Assistants

## Sunto

In questo talk del QCon London, Birgitta Böckeler (Global Lead for AI-assisted Software Delivery at Thoughtworks) analizza l'evoluzione rapida degli agenti di coding AI nell'arco dell'ultimo anno. Il focus non è più sul semplice autocompletamento o sul "vibe coding", ma su sistemi agentici sofisticati che richiedono un approccio strategico alla gestione del contesto — ciò che la relatrice chiama **context engineering**.

Il context engineering rappresenta il cuore della presentazione: non si tratta semplicemente di scrivere prompt migliori, ma di orchestrare dinamicamente quali informazioni vengono fornite al modello e quando. Questo include il caricamento selettivo di skill file, l'integrazione con MCP server, e la gestione just-in-time del contesto per massimizzare l'efficacia degli agenti autonomi, evitando di saturare la finestra di contesto con dati irrilevanti.

Böckeler approfondisce il concetto di **harness engineering**: la costruzione di "reti di sicurezza" architetturali che permettono agli agenti AI di operare in modo autonomo mantenendo la qualità del codice. Queste includono constraint architetturali, tool di analisi statica, e cicli di feedback deterministici che validano le uscite degli agenti prima dell'integrazione. Senza questi guardrail, la velocità guadagnata con l'automazione rischia di tradursi in debito tecnico insostenibile.

Un aspetto critico trattato è l'evoluzione dell'autonomia: dai tool CLI agli agenti cloud-based, fino all'orchestrazione di agenti paralleli. Man mano che il livello di autonomia aumenta, crescono anche i rischi — prompt injection, estrazione di segreti, e costi operativi significativi (da $0.12 a oltre $380 al giorno per sviluppatore). La relatrice sottolinea la necessità di valutare con attenzione dove concedere maggiore autonomia agli agenti, bilanciando velocità di delivery con manutenibilità a lungo termine.

Infine, Böckeler affronta il tema della fiducia e del mantenimento delle competenze: l'adozione acritica degli strumenti AI rischia di causare atrofia delle skill nei team di sviluppo, con conseguenze negative sulla capacità di revisionare e correggere il codice generato. La sua raccomandazione è di investire in framework deliberati per l'adozione dell'AI, dove la velocità non sacrifichi la comprensione profonda del sistema.
