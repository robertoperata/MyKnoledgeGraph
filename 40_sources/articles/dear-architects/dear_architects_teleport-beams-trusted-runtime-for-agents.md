---
tags:
  - ai-agents
  - security
  - infrastructure
  - runtime
  - micro-vm
feature:
type: article
author: Dear Architects
source: https://www.beams.run/
date: 2026-11-15
---

# Teleport Beams: Trusted Runtime for Agents in Production

## Sunto

Teleport Beams è una piattaforma che fornisce runtime sicuri per agenti AI che devono accedere a infrastrutture aziendali. Il problema centrale che affronta è la gestione dell'identità e dei privilegi per agenti autonomi: come garantire che un agente possa raggiungere database, Kubernetes, cloud e altri servizi senza dover memorizzare credenziali persistenti o disporre di permessi eccessivi?

La soluzione si basa su quattro pilastri architetturali. Il primo è la **connettività senza segreti**: gli agenti si connettono alle risorse cloud attraverso un gateway protocol-aware che supporta MCP, LLM e traffico Kubernetes, con auto-discovery delle risorse disponibili. Il secondo pilastro è la **sicurezza deterministica**: i controlli limitano esplicitamente ciò che un agente può fare, mentre il monitoraggio identifica comportamenti rischiosi e thread di esecuzione divergenti.

Il terzo pilastro è la **delega di identità granulare**: gli utenti possono delegare un sottoinsieme dei propri ruoli agli agenti, e le richieste di accesso possono essere sottoposte a revisione da un giudice LLM o umano prima dell'esecuzione. Questo risolve il problema del "principio del minimo privilegio" per sistemi agentici dove i permessi necessari variano dinamicamente.

Il quarto pilastro è la **virtualizzazione con micro-VM**: ogni agente gira in una VM effimera isolata con rete virtuale, supporto per snapshot e workflow di fork/branch. L'infrastruttura usa Cloud Hypervisor, OverlayFS e gVisor; le VM scadono dopo 12-24 ore e possono ripartire da un checkpoint. Questo pattern permette workflow potenti come l'esecuzione parallela di esperimenti con fork di stato o il miglioramento ricorsivo del comportamento dell'agente.

L'architettura prevede che l'harness dell'agente sia integrato in un container image ricostruito periodicamente, mentre i dati, gli MCP server e l'inferenza rimangono nell'infrastruttura del cliente. Teleport supporta oltre 165 integrazioni tra harness agentici (Claude Code, Codex), provider di inferenza (Anthropic, AWS Bedrock, OpenAI), database, CI/CD e strumenti di osservabilità. Il sistema è certificato SOC 2 Type 2 e HIPAA.

## Immagini

Nessuna immagine tecnica rilevante disponibile sulla pagina.
