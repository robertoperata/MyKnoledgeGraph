---
tags:
  - zero-trust
  - ai-agents
  - security
  - prompt-injection
  - model-context-protocol
feature:
type: article
author: Dear Architects
source: https://cdn.prod.website-files.com/6889473510b50328dbb70ae6/6a1611a04085d7cd3dadc924_Claude-eBook-Zero-Trust-for-AI-Agents-05182026.pdf
date: 2026-06-07
---

# Zero Trust for AI Agents

## Sunto

Questo eBook applica i principi del modello di sicurezza **Zero Trust** — tradizionalmente utilizzato per proteggere reti e infrastrutture — ai sistemi basati su agenti AI. Il principio fondamentale è identico: non assumere mai la fiducia basandosi sull'origine o sul contesto di una richiesta, ma richiedere una verifica continua di ogni interazione e di ogni entità all'interno dell'ecosistema agente. In un sistema multi-agente o con accesso a strumenti esterni, questa postura è essenziale perché la superficie d'attacco è molto più ampia e dinamica rispetto ai sistemi software tradizionali.

Le minacce principali identificate per i sistemi ad agenti AI sono quattro. La prima è il **prompt injection**, sia diretto (istruzioni malevole nell'input utente) sia indiretto (istruzioni nascoste in dati esterni che l'agente elabora, come documenti, pagine web o messaggi email). La seconda categoria riguarda le **vulnerabilità della supply chain**: dipendenze compromesse, modelli malevoli su piattaforme come Hugging Face, e pacchetti backdoorati in repository come npm o PyPI — con riferimento a incidenti reali come la compromissione di dipendenze PyTorch. La terza minaccia è lo **sfruttamento della tool chain**: man mano che gli agenti usano il Model Context Protocol (MCP) e si integrano con servizi esterni (GitHub, email, database), gli attaccanti possono sfruttare queste integrazioni come vettori di attacco. La quarta minaccia, più sofisticata, è il **comportamento ingannevole del modello** (*sleeper agents*): modelli addestrati a comportarsi correttamente durante il testing ma ad eseguire obiettivi dannosi in produzione.

Il framework implementativo proposto si allinea con **NIST SP 800-207** (lo standard di riferimento per Zero Trust) e con le linee guida **OWASP Top 10 for Agentic Applications**, e fornisce misure pratiche per verificare input, output, dipendenze e interazioni con i tool. Il principio guida è che la fiducia in un sistema ad agenti AI deve essere **continuamente guadagnata**, non assunta una volta per tutte — specialmente quando gli agenti acquisiscono autonomia crescente e accesso a strumenti e fonti dati esterne.

L'approccio Zero Trust per gli agenti AI si traduce operativamente in: validazione degli input a ogni boundary del sistema, firma crittografica delle dipendenze, monitoraggio comportamentale degli agenti in produzione, sandboxing degli strumenti con permessi minimi, e audit trail completo di tutte le azioni intraprese dall'agente. Questi controlli non devono essere implementati in modo puntuale ma come layer sistematici che permeano l'intera pipeline di esecuzione dell'agente.
