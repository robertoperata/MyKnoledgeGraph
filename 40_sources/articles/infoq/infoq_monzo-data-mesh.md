---
tags:
  - data-mesh
  - dbt
  - data-warehouse
  - data-governance
  - software-architecture
  - platform-engineering
feature:
type: article
author: Renato Losio
source: https://www.infoq.com/news/2026/05/monzo-data-mesh/
date: 2026-08-01
---

# Neobank Monzo Builds Governed Data Mesh across 100 Teams and 12,000 dbt Models

## Sunto

Monzo, una banca digitale britannica, ha riprogettato il suo data warehouse per supportare oltre 100 team che lavorano su più di 12.000 modelli dbt. L'iniziativa ha raggiunto risultati misurabili: riduzione dei costi del warehouse di circa il 40% e miglioramento dei tempi di atterraggio dei dati del 25%. Il punto di partenza era la sfida classica della crescita: la proprietà distribuita dei dati è potente, ma estremamente difficile da gestire correttamente a scala.

L'architettura implementata segue tre principi fondamentali: applicare standard chiari e uniformi a tutti i modelli dati, formalizzare la condivisione dei dati attraverso interfacce esplicite, e affidarsi all'automazione e ai controlli CI per l'assicurazione della qualità. Questi principi si traducono in una struttura a quattro livelli di modellazione: Landing Layer (modelli automatizzati che appiattiscono gli eventi grezzi), Normalized Layer (modelli generati che rappresentano entità con storia completa), Logical Layer (logica di business che combina più entità), e Presentation Layer (modelli adattati per applicazioni downstream specifiche).

Il meccanismo di governance chiave è Modelgen, un'utilità da riga di comando che genera modelli SQL e YAML da definizioni di oggetti, garantendo coerenza nel sistema distribuito. Ogni modello deve includere una chiave unica, test di freschezza, elaborazione incrementale come default, team proprietario dichiarato, documentazione e convenzioni rigorose di denominazione. Questi requisiti vengono applicati tramite controlli CI automatizzati, non tramite processi manuali di revisione.

L'aspetto più significativo dell'architettura è la formalizzazione delle "interface models": modelli espliciti che governano le dipendenze di dati tra team. Prima di questa iniziativa, i team consumavano direttamente i modelli interni degli altri, creando dipendenze implicite e fragilità. Con le interfacce dichiarate, ogni team espone solo ciò che intende rendere pubblico, riducendo query ridondanti e ricalcoli non necessari.

La migrazione è attualmente al 30% di completamento nell'organizzazione, ma ha già dimostrato risultati concreti. La lezione principale è che trattare le interfacce dati come codice di prima classe — con gli stessi standard di qualità e governance del codice applicativo — è ancora sorprendentemente raro nell'industria, ma produce miglioramenti tangibili sia in termini economici che di velocità di delivery.
