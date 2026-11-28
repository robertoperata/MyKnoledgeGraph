---
tags:
  - mcp
  - ai-agents
  - api-gateway
  - platform-engineering
  - agentic-ai
feature:
type: article
author: InfoQ
source: https://r.mail.infoq.com/mk/cl/f/sh/7nVU1aA2nfy8RKs71MpQi5V8zGwCRgX/iZTQZCs09xrp
date: 2026-11-28
---

# Uber Builds MCP Gateway to Govern Access to Over Eight Hundred Servers and Over Five Thousand AI Agent Tools

## Sunto

Uber ha costruito un MCP Gateway per standardizzare il modo in cui gli agenti AI scoprono e invocano i servizi interni dell'azienda. Il problema affrontato era quello della frammentazione: con la proliferazione degli agenti AI all'interno di Uber, ogni team stava costruendo le proprie integrazioni in modo indipendente, generando infrastrutture duplicate, pratiche di sicurezza incoerenti e costi operativi crescenti. Il gateway centralizzato risolve questi problemi fornendo un livello comune che astrae la complessità di centinaia di servizi backend.

La piattaforma supporta più di 800 server MCP e 5.000 strumenti, offrendo funzionalità trasversali di discovery, traduzione di protocollo, autorizzazione ed esecuzione. Questo approccio incarna il principio della **riusabilità come infrastruttura**: invece di esporre ogni servizio direttamente agli agenti con integrazioni ad hoc, il gateway diventa il contratto condiviso tra il mondo degli agenti e il mondo dei servizi. Il risultato è una superficie di governance unificata — un punto singolo dove applicare policy di sicurezza, audit trail e controllo degli accessi su tutto il parco strumenti agentici.

L'architettura di Uber riflette una tendenza più ampia nell'ingegneria dei sistemi agentici: la necessità di costruire un **capability platform** che disaccoppi il ragionamento degli agenti dall'implementazione dei servizi sottostanti. Questo pattern permette ai team di servizi di evolvere le proprie API senza rompere gli agenti, e agli agenti di accedere a nuove capacità senza dover rinegoziare integrazioni individuali. Il gateway MCP diventa così il punto di estensibilità controllata per l'intera flotta di agenti AI aziendali.
