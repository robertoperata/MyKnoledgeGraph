---
tags:
  - mcp
  - api-gateway
  - multi-agent
  - service-mesh
  - llm-agents
feature:
type: article
author: Alok Srivastava, Deepanshu Mehndiratta, Gaurav Gill
source: https://www.uber.com/gb/en/blog/designing-mcp-gateway/
date: 2026-10-11
---

# Designing MCP Gateway: Uber's MCP Management Platform

## Sunto

L'adozione rapida degli agenti AI in Uber ha evidenziato un problema di frammentazione: ogni team costruiva la propria integrazione MCP (Model Context Protocol) in modo indipendente, creando duplicazioni, difficoltà di discovery e rischi di sicurezza. La risposta è stata l'**MCP Gateway**, un microservizio centralizzato che funge da piano di controllo e di dati per tutti gli agenti AI dell'azienda, ospitando oltre 800 server MCP e 5.000 strumenti.

Il gateway è composto da due piani distinti. Il **piano di controllo** è l'**MCP Registry**, un catalogo che rappresenta la fonte di verità unica per la discovery, la proprietà e l'abilitazione degli strumenti. Il **piano di dati** è il **Proxy Gateway**, che traduce le chiamate MCP in richieste HTTP, gRPC o TChannel verso i servizi downstream e riconverte le risposte nel formato MCP. Questa separazione permette di aggiornare la configurazione senza riavviare il servizio.

Un componente chiave è l'**AutoCrawler**, un workflow Cadence che scansiona il registro IDL di Uber su base programmata. Per i servizi Protobuf o Thrift, analizza le definizioni IDL e usa un LLM per generare descrizioni degli strumenti ottimizzate per gli agenti. I server MCP nativi costruiti con MCPFx vengono invece scoperti tramite metriche heartbeat e una chiamata `listTools`. Tutti gli elementi scoperti partono nello stato "disabilitato" finché i proprietari non li revisionano e abilitano.

La sicurezza è gestita a livello di strumento tramite il sistema di controllo accessi interno di Uber (Access Control System), con policy a livello di server e override opzionali per singolo strumento. Il gateway include anche un meccanismo di redazione automatica che rimuove PII e dati sensibili dalle risposte degli strumenti. Per le integrazioni di terze parti (Jira, Google), il gateway gestisce lo scambio di token utente e applica rate limiting.

Per gestire la scalabilità con migliaia di strumenti, sono state introdotte tre soluzioni: **Omni MCP** (un singolo proxy con quattro strumenti meta per la discovery incrementale che riduce il consumo di contesto), **Response Projection** (un pattern simile a GraphQL per restituire solo i campi necessari), e **Code Mode** (una CLI `aifx` che permette agli agenti di coding di scrivere output su file e caricare solo ciò che serve).

## Immagini

![Architettura MCP Gateway: piano di controllo e piano di dati](https://www.uber.com/gb/en/blog/designing-mcp-gateway/)

## Codice

Questo articolo descrive l'architettura ad alto livello senza fornire snippet di codice pubblici. I pattern chiave includono:

- Il proxy espone un endpoint `/<service-name>/mcp` per ogni virtual server
- La traduzione di protocollo avviene via serializzazione Protobuf/Thrift attraverso Muttley, il service mesh sidecar di Uber
- AutoCrawler usa workflow Cadence per la scoperta schedulata delle API
