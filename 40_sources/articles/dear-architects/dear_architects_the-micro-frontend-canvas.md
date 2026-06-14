---
tags:
  - micro-frontends
  - software-architecture
  - frontend-architecture
  - boundaries
  - design-tools
feature:
type: article
author: Luca Mezzalira
source: https://lucamezzalira.medium.com/the-micro-frontend-canvas-a-practical-tool-for-better-boundaries-99da1a7b858d
date: 2026-06-14
---

# The Micro-Frontend Canvas: A Practical Tool for Better Boundaries

## Sunto

Luca Mezzalira, dopo vent'anni di lavoro con oltre cento team globali, identifica la causa principale del fallimento nei progetti micro-frontend: non l'incompetenza tecnica, ma l'aver saltato completamente la fase di design. I team spesso saltano direttamente all'implementazione senza rispondere a domande fondamentali: questa architettura è appropriata per il problema? Come si definiscono confini di dimensione corretta? Come comunicano i componenti senza creare accoppiamento stretto? Il risultato sono sistemi apparentemente indipendenti ma con interdipendenze nascoste nel codice.

Il Micro-Frontend Canvas è uno strumento strutturato su una singola pagina che cattura le decisioni chiave per ogni singolo micro-frontend. Il principio cardine è: un micro-frontend richiede un canvas. Lo strumento opera in due modalità: in fase di pianificazione, forza il team a esaminare confini, vincoli e dipendenze prima di scrivere codice, creando allineamento tra engineering, product e leadership; in modalità di assessment, funziona diagnosticamente per sistemi esistenti con problemi, identificando leak di confini, dipendenze nascoste e gap di ownership.

Le sezioni principali del canvas affrontano le domande più critiche: la business capability identificata (non solo la UI resa), la validazione dei confini (è possibile fare deploy senza coordinarsi con altri team? quali dati possiede vs. prende in prestito?), le dipendenze, la metodologia di comunicazione con gli altri componenti incluso il contract testing, e la governance in produzione. La sezione sulla validazione dei confini è la più potente: applicata al checkout in un e-commerce, rivela immediatamente se il componente ha dipendenze runtime nascite verso il catalogo prodotti, suggerendo di usare un backend-for-frontend dedicato per aggregare i dati di pricing.

L'approccio operativo privilegia l'asincronia: una breve sessione di identificazione dei confini con stakeholder cross-funzionali, poi drafting asincrono da parte di una o due persone, revisione del team in uno o due giorni, e trattamento del canvas come documento vivente da riaprire quando emerge attrito. Non è un artefatto da workshop one-shot, ma un riferimento evolutivo. Questo approccio emergeva dal testing con grandi organizzazioni che trovavano i workshop one-time inefficaci.

Il canvas si integra anche con gli strumenti di sviluppo AI: quando i team forniscono definizioni di confini chiare agli assistenti AI, questi scaffold in modo corretto; senza il canvas, gli agenti tendono ai default dei tutorial che dissolvono i confini — cross-import tra micro-frontend, store globali condivisi, contratti sovradimensionati. Il canvas determina dove vanno i confini; skills complementari per gli assistenti AI ne mantengono l'integrità nel codice generato. Lo strumento è disponibile su GitHub con licenza Creative Commons per uso non commerciale.
