---
tags:
  - sre
  - agentic-ai
  - incident-management
  - reliability
  - devops
feature:
type: article
author: Stevan Malesevic, Christopher Heiser
source: https://cloud.google.com/blog/products/devops-sre/how-google-sre-is-using-agentic-ai-to-improve-operations/
date: 2026-05-31
---

# AI in SRE: Come Google sta usando l'AI agentiva per migliorare le operazioni

## Sunto

Google SRE utilizza l'ingegneria della affidabilità del sito da oltre 20 anni per mantenere operativi servizi come Search, Gmail, Maps, YouTube e Google Cloud. L'articolo descrive come l'emergere dell'AI stia provocando un cambiamento di scala nella complessità dei sistemi — architetture a microservizi più distribuite geograficamente, maggiore diversità hardware, normative più complesse, e la capacità dei modelli generativi di produrre ordini di grandezza più codice. La risposta di Google è **SRE AI**: sfruttare l'AI come moltiplicatore di forza mantenendo il controllo umano.

Le aree di applicazione dell'AI in SRE si articolano in cinque domini principali. Il primo è la **Reliability Design**: agenti AI monitorano e migliorano continuamente playbook e documentazione di produzione basandosi sull'uso reale degli incidenti, e possono generare nuovi playbook direttamente dagli incidenti passati. Il secondo è l'**Anomaly Detection e Alerting**: invece delle soglie statiche SLI/SLO, agenti raccolgono segnali e li alimentano a modelli time-series come TimesFM per rilevare anomalie comportamentali, con handler autonomi in grado di mitigare categorie di problemi senza intervento umano.

Il terzo dominio è l'**Incident Management** (IMAG), un layer di orchestrazione agentiva che monitora superfici di comunicazione (chat, video, documenti degli incident response), consolida e sintetizza dati, supporta i passaggi di consegna tra SRE con documenti contestuali, genera automaticamente bozze di postmortem e gestisce le comunicazioni interne ed esterne durante gli incidenti. Il quarto è l'**Incident Investigation**: agenti investigano incidenti autonomamente usando dati di osservabilità (logging, monitoring, tracing), topologia del sistema e dati di dipendenza, navigando ed eseguendo playbook e rilevando anomalie. Il quinto è **Insights e Risk Management**: un sistema continuo che estrae informazioni da incidenti noti tramite embedding Gemini e vector database, categorizzando i rischi per guidare le decisioni degli agenti.

L'infrastruttura tecnica si basa su Gemini (con versioni fine-tuned su dati interni), Vertex AI come piattaforma full-stack, il framework Agent Development Kit (ADK) per lo sviluppo, MCP server su infrastruttura Google API standard, BigQuery e vector database per capacità AI/ML. I principi di design enfatizzano trasparenza (gli agenti devono spiegare le azioni e le alternative considerate), identità forte (ruoli e permissioni degli agenti), SLO di alta affidabilità con fallback ben definiti, e piani di continuità operativa che includano scenari di fallimento dell'AI.

I criteri di successo definiti da Google richiedono che un sistema SRE AI consegua almeno uno dei seguenti obiettivi: alleggerire gli ingegneri da operazioni ripetitive e laboriose; migliorare qualità e velocità del decision-making; abilitare prevenzione, rilevamento e mitigazione migliori; abilitare feedback loop agentici autonomi per la reliability del servizio; ridurre i costi operativi complessivi.

## Immagini

![Fasi SDLC con coinvolgimento SRE](https://storage.googleapis.com/gweb-cloudblog-publish/images/image1_3Jp6s6J.max-1100x1100.png)
