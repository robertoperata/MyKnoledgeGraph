---
tags:
  - service-catalog
  - mcp-server
  - platform-engineering
  - system-topology
  - rappitech
feature:
type: article
author: Dear Architects
source: https://medium.com/rappitech/cartographer-a-living-map-of-everything-we-run-3590f1c2270f
date: 2026-10-04
---

# Cartographer: A Living Map of Everything We Run

## Sunto

Cartographer è il sistema sviluppato da RappiTech per mantenere una mappa vivente e interrogabile di tutto il loro ecosistema di servizi: un ecosistema composto da quasi tremila servizi in continua evoluzione. Il problema affrontato è uno dei più critici nelle organizzazioni a larga scala: come mantenere una rappresentazione accurata e aggiornata dell'intero parco applicativo, quando i servizi cambiano continuamente e nessun documento manuale riesce a stare al passo con la realtà.

La metafora del cartografo è centrale nel design del sistema: il territorio è l'ecosistema reale — i servizi che girano in produzione, le loro dipendenze, i loro metadati — e la mappa è la sua rappresentazione interrogabile. Il cartografo è il sistema che lavora continuamente per mantenere la mappa allineata al territorio, raccogliendo informazioni dai diversi sistemi sorgente e consolidandole in una rappresentazione coerente.

Un elemento architetturale particolarmente rilevante è l'esposizione di Cartographer come **MCP server** (Model Context Protocol). Questo significa che non si tratta solo di un catalogo statico o di un portale di documentazione, ma di un servizio attivamente interrogabile da assistenti AI, coding agent e piattaforme interne usando un protocollo comune e standardizzato. La scelta di MCP consente agli strumenti di sviluppo basati su AI di accedere ai metadati del sistema in modo programmatico e contestuale.

L'architettura risponde a una sfida tipica delle piattaforme di ingegneria: come evitare che il catalogo dei servizi diventi obsoleto appena pubblicato. Cartographer affronta questo problema con un approccio di raccolta continua dei dati, mantenendo la mappa sincronizzata con il territorio reale attraverso aggiornamenti automatici. Questo la distingue dai cataloghi tradizionali basati su aggiornamenti manuali, che tendono a deteriorarsi rapidamente nelle organizzazioni con alta velocità di sviluppo.

> **Nota**: L'articolo completo non era accessibile al momento dell'elaborazione (HTTP 403). Questo sommario è basato sulla descrizione presente nella newsletter Dear Architects #310.
