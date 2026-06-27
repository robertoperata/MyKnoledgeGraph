---
tags:
  - mcp
  - ai-agents
  - browser-automation
  - kubernetes
  - distributed-systems
feature:
type: article
author: Paul Klein
source: https://www.infoq.com/presentations/parallel-agents-production/
date: 2026-06-27
---

# Automating the Web with MCP: Infra That Doesn't Break

## Sunto

Paul Klein, fondatore di Browserbase, affronta le sfide dei sistemi distribuiti nello scalare l'infrastruttura browser cloud-hosted per agenti AI. La tesi centrale è che il browser è il punto di integrazione universale per qualsiasi sito web, rendendo la browser automation un enabler fondamentale per gli agenti AI che devono interagire con il mondo reale.

L'architettura di Browserbase è strutturata su sei livelli distinti: il sandbox (Firecracker VMM), lo scheduler (Kubernetes), il browser (Chromium headless), il protocollo (Chrome DevTools Protocol o VNC), il framework (Puppeteer, Playwright, Stagehand) e il modello (selezione LLM/vLLM). Questo stack multi-layer riflette un approccio difensivo in profondità: ogni livello aggiunge isolamento e controllo indipendente dagli altri. Firecracker viene scelto rispetto ai container tradizionali perché assume già che il browser possa essere compromesso, offrendo isolamento a livello di sistema operativo.

Il Model Context Protocol (MCP) viene posizionato come lo strato di astrazione che unifica l'integrazione degli strumenti tra diversi provider LLM. A differenza di una REST API tradizionale, MCP aggiunge descrizioni in linguaggio naturale, definizioni di funzioni e autenticazione integrata, rendendo gli strumenti comprensibili ai modelli. Klein distingue tra MCP "orizzontali" (strumenti primitivi e cross-domain, come il browser) e MCP "verticali" (specializzati per un singolo dominio, come GitHub MCP).

Per la sicurezza, l'articolo descrive un approccio multi-strato che affronta la prompt injection tramite HTML avvelenato, raccomanda il principio del minimo privilegio per l'accesso degli agenti e propone lo standard "Web Bot Auth" per l'identificazione dei bot. Un problema aperto rimane la gestione dei CAPTCHA. Il framework Stagehand riduce l'overhead dei token accettando input in linguaggio naturale tramite un approccio "subagent", risultando più efficiente rispetto al codice Playwright/Selenium verboso.

Dal punto di vista operativo, le sfide in produzione includono crash di Chromium per esaurimento della memoria, gestione degli iframe out-of-process, dropdown OS nativi non visibili negli screenshot e failover multi-regione per la gestione della capacità. La metrica chiave citata — "92 anni di navigazione" dai clienti in un mese — dà la misura della scala e della necessità di un'infrastruttura robusta. La presentazione conclude che l'intelligenza di un agente senza strumenti affidabili è inutile: l'infrastruttura che non si rompe è il prerequisito per qualsiasi automazione agentistica.
