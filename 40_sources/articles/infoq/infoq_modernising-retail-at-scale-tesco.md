---
tags:
  - cloud-to-edge
  - platform-engineering
  - digital-transformation
  - microservices
  - enterprise-architecture
feature:
type: article
author: Jayesh Bhayani
source: https://www.infoq.com/presentations/tesco-enterprise-tech-transformation/
date: 2026-11-28
---

# Modernising Retail at Scale: Architecting a Cloud‑to‑Edge Platform for a Global Enterprise

## Sunto

Jayesh Bhayani, Senior Engineering Director di Tesco, illustra la trasformazione tecnologica decennale del retailer britannico: da un ecosistema frammentato di sistemi vendor-dipendenti a una piattaforma cloud-to-edge unificata. Tesco opera circa 5.000 negozi in cinque paesi, serve 25 milioni di clienti e conta 6.000 professionisti tecnologici. Il punto di partenza era critico: stack separati per canale (grocery, marketplace, abbigliamento, vino), nessun login unificato, software vendor pesantemente personalizzato e cicli di modifica ai sistemi POS di 6-9 mesi.

La trasformazione si è articolata su tre assi paralleli: talento, organizzazione e architettura. Sul fronte del talento, Tesco ha reinsourced le competenze ingegneristiche, ridefinito i percorsi di carriera e aperto hub tecnologici in Ungheria e Polonia. Sul fronte organizzativo, i team sono stati mappati sui domini di business (supply chain, commerciale, canali, clienti) con la stessa squadra responsabile sia del sistema legacy sia di quello nuovo. La decisione build-vs-buy è stata netta: si costruisce tutto ciò che è differenziante (piattaforma retail, POS, dispositivi handheld), si acquista per sistemi back-office non differenzianti. La governance è passata da comitati centrali di revisione a un modello di "libertà entro i guardrail".

Architetturalmente, la piattaforma adotta microservizi componibili con multi-tenancy nativa, sicurezza a strati e astrazioni che riducono il numero di chiamate di rete per ogni operazione. La componente edge è particolarmente rilevante: gli stessi servizi girano nei negozi con fallback offline che garantisce la continuità delle transazioni anche in assenza di rete. La strategia cloud è ibrida: circa l'80% dei workload risiede nel cloud privato e il 20% in quello pubblico, ma la ripartizione dei costi è sostanzialmente inversa. Si privilegia un PaaS portabile per limitare il lock-in, e il costo del cloud viene trattato come una feature di prodotto da gestire.

Guardando al futuro, Bhayani affronta il tema del commercio agentivo: perché gli agenti AI possano operare su sistemi retail, occorrono dati machine-readable, policy di identità e consenso, e nuovi protocolli come UCP e ACP — molti dei quali non sono ancora pronti per la produzione. Le lezioni apprese includono l'importanza di costruire prima le capacità di piattaforma (CI/CD, osservabilità), adottare il portfolio management prima del previsto e evitare la duplicazione di strumenti tra i team.

## Immagini

![QCon London - Tesco Cloud-to-Edge Platform](https://res.infoq.com/presentations/tesco-enterprise-tech-transformation/en/promoimage/qconlondon-1792564944183.jpg)
