---
tags:
  - ai-agents
  - software-architecture
  - large-scale-systems
  - engineering-leadership
  - cognitive-tools
feature:
type: article
author: Julie Qiu
source: https://www.infoq.com/presentations/ai-large-scale-engineering-systems/
date: 2026-05-24
---

# Leveraging AI for Large-Scale Engineering Systems

## Sunto

Julie Qiu, Senior Staff Engineer a Google Cloud, presenta come l'AI funzioni da strumento cognitivo per i leader tecnici che gestiscono sistemi di enorme complessità: il suo team mantiene oltre 400 repository su nove linguaggi di programmazione. Il contributo principale della presentazione è dimostrare che il vero valore dell'AI non è accelerare la produzione di codice, ma aumentare la capacità cognitiva di sintetizzare contesto legacy e architettare redesign di sistema.

Qiu identifica cinque ruoli distinti dell'AI nel lavoro di ingegneria. Come **Archeologo**, l'AI ricostruisce la storia e la struttura di sistemi legacy analizzando basi di codice che sarebbero impossibili da comprendere manualmente in tempi ragionevoli — Gemini ha distillato migliaia di righe di codice fino al loro comportamento essenziale. Come **Sperimentatore**, l'AI serve da motore di simulazione a basso costo per testare ipotesi architetturali senza impegnare risorse di ingegneria. Come **Critico**, pressiona i design identificando debolezze, overengineering e complessità non necessaria.

Come **Autore**, l'AI genera codice in produzione eccellendo in task concreti e ripetitivi con criteri di correttezza espliciti (ordinare alfabeticamente i campi di struct, refactoring di import su repository multipli), ma mostrando vizi ricorrenti: over-commenting, whitespace eccessivo, nil-checking non necessario. Il quarto ruolo, il **Code Reviewer**, è quello dove l'AI cattura errori meccanici prima della revisione umana, ma non può valutare decisioni che richiedono contesto di roadmap o giudizi estetici sull'architettura futura.

Il percorso di redesign ha attraversato due fallimenti prima di trovare un approccio funzionante. Un primo tentativo "steel thread" su un solo linguaggio è diventato troppo biased verso la filosofia di quel linguaggio. Un secondo tentativo con astrazione precoce su due linguaggi ha creato complessità ingestibile. La svolta è arrivata partendo da princìpi fondamentali: un semplice README con tre responsabilità core (gestione dello stato, generazione delle librerie, pubblicazione dei package), branch git separati per linguaggio, prompt di "design partner" consistenti per mantenere la vision.

La scoperta più profonda dell'esperienza di Qiu è che il vero collo di bottiglia non era l'esecuzione (velocità di typing), ma il cognitivo: "tenere il contesto... lo stato di tutto... in oltre 400 repository". L'AI non l'ha resa dieci volte più veloce, ma dieci volte più presente nel lavoro, liberando risorse mentali per il giudizio architetturale mentre gestiva il lavoro di routine che richiedeva expertise di dominio ma non ispirazione creativa.
