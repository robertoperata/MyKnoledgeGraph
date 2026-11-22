---
tags:
  - cloud-architecture
  - edge-computing
  - platform-engineering
  - microservices
  - retail-tech
feature:
type: article
author: Jayesh Bhayani
source: https://www.infoq.com/presentations/tesco-enterprise-tech-transformation/
date: 2026-11-22
---

# Modernising Retail at Scale: Architecting a Cloud‑to‑Edge Platform for a Global Enterprise

## Sunto

Jayesh Bhayani, Senior Engineering Director di Tesco, descrive la trasformazione decennale dell'infrastruttura tecnologica dell'azienda: da sistemi frammentati e isolati a una piattaforma retail unificata che copre sia il cloud che i dispositivi fisici nei punti vendita. Tesco opera circa 5.000 negozi in cinque paesi, con circa 50.000 dispositivi POS e terminali portatili. Il punto di partenza era caotico: stack separati per grocery, marketplace e abbigliamento, nessuna identità cliente unificata, nessuna fonte unica di verità per stock e prezzi, software aggiornato tramite supporti fisici.

La piattaforma retail oggi si articola in oltre 60 domini e centinaia di microservizi. Il modello di governance è evoluto da board di change centralizzati a un approccio "freedom within guardrails": ogni team ha chiari confini di dominio e standard condivisi, ma autonomia operativa. La struttura organizzativa rispecchia le unità di business (supply chain, commerciale, canali, clienti), seguendo il principio di Conway's Law.

Una delle scelte architetturali più significative riguarda la strategia ibrida cloud: circa l'80% dei workload gira su private cloud e il 20% su public cloud, ma la ripartizione dei costi è invertita (circa l'80% della spesa è nel public cloud). Questa anomalia ha spinto a rimpatriare workload dopo che la spesa iniziale non controllata ha rivelato un costo 8-10x superiore per configurazioni equivalenti. L'approccio attuale predilige PaaS portatile per limitare il vendor lock-in e tratta il costo come una caratteristica di design.

L'aspetto più innovativo è l'architettura cloud-to-edge: gli stessi microservizi girano sia nel cloud che sui dispositivi edge nei negozi, con la stessa logica di business e consistenza dei dati. L'operatività offline è un requisito fondamentale: quando la rete cade, il sistema privilegia il cliente, continuando a identificare i loyalty member e accettare pagamenti. Anche i tag di prezzo elettronici sugli scaffali devono funzionare durante i guasti. Un unico team possiede sia la piattaforma edge che quella cloud, inclusi i sistemi legacy in migrazione.

Guardando al futuro, Tesco sta valutando il commercio agentivo (agentic commerce), dove gli agenti AI effettuano acquisti per conto dei clienti. I requisiti tecnici includono gateway, gestione dell'identità, consenso e controlli di policy. Bhayani introduce il concetto di AEO (Agentic Engine Optimization): i dati di prodotto devono essere leggibili dai crawler degli agenti. Protocolli come ACP, UCP, A2A e MCP sono in valutazione ma non ancora pronti per la produzione alla scala di Tesco. Una nota pratica: la codifica agentica guidata dagli sviluppatori può generare spese cloud significative e difficili da giustificare (cita circa £500 per sviluppatore al giorno).

## Immagini

![Modernising Retail at Scale - thumbnail presentazione](https://imgopt.infoq.com/fit-in/100x100/filters:quality(80)/presentations/tesco-enterprise-tech-transformation/en/smallimage/JayeshBhayani-thumbnail-1792654742095.jpg)

![QCon London - immagine promozionale](https://res.infoq.com/presentations/tesco-enterprise-tech-transformation/en/promoimage/qconlondon-1792564944183.jpg)
