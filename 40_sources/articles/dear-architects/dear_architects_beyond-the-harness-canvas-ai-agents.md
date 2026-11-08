---
tags:
  - ai-agents
  - harness-engineering
  - software-architecture
  - context-engineering
  - drift-detection
feature:
type: article
author: Nimisha Asthagiri
source: https://www.infoq.com/presentations/canvas-ai-agents
date: 2026-11-08
---

# Beyond the Harness: Canvas for AI Agents

## Sunto

Nimisha Asthagiri, Technical Advisor di Thoughtworks, presenta a QCon AI Boston 2026 un framework alternativo all'harness engineering per gestire gli agenti AI: il **canvas**. Mentre l'harness si concentra sul controllo e la costrizione, il canvas è uno spazio creativo con confini definiti, struttura e validazioni — meno glue, più architettura.

La conferenza prende le mosse dall'evoluzione terminologica: da prompt engineering a context engineering, fino a harness engineering. Asthagiri cita la definizione di LangChain ("un agente è il modello più tutto il resto") e il framework cybernetico di Birgitta Böckeler, che distingue tra guide in avanzamento (feedforward: contesto, best practice, decision record) e sensori di feedback (CI, linting). Il canvas aggiunge a questo framework un livello di **bounded autonomy**: gli agenti operano entro confini espliciti invece di essere vincolati da regole sempre più numerose.

Il problema principale dell'harness tradizionale è l'instruction bloat: set di regole enormi causano confusione nel contesto, skill fragili e output prevedibili peggiori. Asthagiri propone invece di iniziare piccolo e migliorare il canvas nel tempo. I cinque elementi del canvas sono: boundaries (confini del dominio), scaffolding (struttura di supporto), priming (contesto iniziale), constraints (una "palette" di vincoli essenziali), e validations apprese dai prototipi.

Un aspetto fondamentale è la **continuous garbage collection**: ispirandosi all'esperienza di OpenAI, Asthagiri raccomanda agenti dedicati che puliscono codice non utilizzato, feature toggle obsoleti e drift architetturale. Questo è complementare al testing avversariale: rimuovere deliberatamente codice o strumenti per verificare che gli agenti si riprendano, con dashboard che mostrano decay, staleness e drift.

L'altro pilastro è il modello di **decision records come stigmergia**: le decisioni architetturali, organizzative e di prodotto lasciano tracce per chi eredita il codebase — inclusi gli agenti futuri. Non tutte le decisioni richiedono lo stesso livello di review: la reversibilità dovrebbe determinare quali richiedono approvazione umana. Asthagiri adatta il modello operativo usato a edX (define, build, review) a team di agenti, con cicli di feedback a micro e macro livello.

## Tipi di feedback nel canvas

| Tipo | Meccanismo |
|------|-----------|
| **Strutturale** | Type systems, sandbox, isolamento |
| **Reattivo** | Backpressure, rate limits, linter |
| **Dinamico** | Drift guard, budget guardrail |
| **Riflessivo** | Strategic review periodico, retrospettive |

## Insight chiave

- Mantenere i file di istruzioni per agenti (es. CLAUDE.md) snelli: vengono caricati nel contesto ad ogni sessione
- Usare mappe (C4 livelli 0 e 1, Wardley maps) invece di documentazione sprawl
- I test avversariali (rimozione deliberata di codice/tool) sono il chaos engineering dell'AI
- Trattare gli agenti come senior engineer capaci, non come automatomi, produce output migliori
