---
tags:
  - mcp
  - cloudflare
  - ai-agents
  - token-optimization
  - api-integration
feature:
type: article
author: Leela Kumili
source: https://www.infoq.com/news/2026/04/cloudflare-code-mode-mcp-server/
date: 2026-04-25
---

# Cloudflare Launches Code Mode MCP Server to Optimize Token Usage for AI Agents

## Sunto

Cloudflare ha introdotto un MCP (Model Context Protocol) server basato su **Code Mode**, una soluzione che riduce drasticamente il consumo di token quando gli agenti AI interagiscono con API complesse. Questa innovazione affronta una sfida fondamentale nell'AI agentica: le limitazioni della finestra di contesto che vincolano quante definizioni di tool un LLM può gestire contemporaneamente.

Il problema risolto è quello delle implementazioni MCP tradizionali, dove ogni endpoint API viene esposto come una definizione di tool separata. Questo approccio è intuitivo ma costoso: ogni specifica di tool consuma token dalla finestra di input del modello, riducendo lo spazio disponibile per il ragionamento effettivo sul task dell'utente. Nel caso di Cloudflare, con oltre 2.500 endpoint API, la specifica completa richiederebbe più di 1,17 milioni di token — un overhead insostenibile.

La soluzione Code Mode espone invece solo due tool: `search()` e `execute()`, supportati da un SDK type-aware. Anziché caricare le specifiche API nel contesto, il modello genera snippet di JavaScript che orchestrano le operazioni API necessarie, eseguiti in un sandbox V8 sicuro. La funzione `search()` interroga le OpenAPI spec senza caricarle in contesto; la funzione `execute()` gestisce paginazione, logica condizionale, e chiamate concatenate in un unico ciclo. Il risultato è una riduzione da 1,17 milioni di token a circa 1.000 token — una diminuzione del 99,9% — con overhead fisso indipendente dalla dimensione della superficie API.

Dal punto di vista della sicurezza, il codice generato viene eseguito in un ambiente sandbox senza accesso al filesystem e con richieste outbound controllate, minimizzando la superficie di attacco. Cloudflare ha open-sourcato il Code Mode SDK e reso disponibile il server MCP per gli sviluppatori.

Questo approccio potrebbe influenzare significativamente il modo in cui i server MCP e i framework agentici vengono progettati in futuro: invece di esporre migliaia di tool specializzati, il pattern emergente è fornire al modello la capacità di **scrivere codice** per accedere alle funzionalità necessarie, combinando la flessibilità della generazione di codice con i guardrail di un ambiente di esecuzione controllato.

## Codice

Il server MCP espone solo due strumenti all'agente AI, riducendo drasticamente il consumo di token:

```javascript
// Ricerca negli endpoint API disponibili senza caricarli in contesto
search("list all DNS records for a zone")

// Esecuzione di operazioni API complesse, con paginazione e logica condizionale
execute(`
  const zones = await cloudflare.zones.list();
  const zone = zones.find(z => z.name === 'example.com');
  return await cloudflare.dns.records.list({ zone_id: zone.id });
`)
```
