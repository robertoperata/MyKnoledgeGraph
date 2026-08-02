---
tags:
  - ai-gateway
  - evolutionary-architecture
  - agentic-ai
  - enterprise-security
  - observability
  - llm-routing
feature:
type: article
author: Joe Price, Branimir Đurek, Pavlos Migkiros, Trevor Dearham
source: https://www.infoq.com/articles/evolutionary-architecture-pattern/
date: 2026-08-02
---

# An Evolutionary Architecture Pattern for Managing AI's Pace of Change

## Sunto

Il problema centrale affrontato dall'articolo è la divergenza di velocità tra l'ecosistema AI — in rapida evoluzione con nuovi modelli e protocolli pubblicati più volte l'anno — e i sistemi enterprise tradizionali, progettati per la stabilità su periodi pluriennali. Gli autori propongono il pattern dell'**AI gateway** come strato di controllo centralizzato che isola i componenti AI in rapido cambiamento dai sistemi backend stabili, permettendo alle applicazioni a valle di rimanere stabili mentre l'ecosistema AI evolve.

L'AI gateway consolida cinque piani di controllo fondamentali: **routing dei modelli e astrazione dei provider** (che permette di cambiare LLM senza modificare il codice applicativo), **gestione dell'identità e autorizzazione** per la delega agent-to-agent, **enforcement delle policy di azione** con principi zero-trust secondo NIST SP 800-207, **guardrail sul contenuto** per rilevare prompt injection e prevenire data leakage, e **audit log semantici** che tracciano la sequenza Richiesta → Decisione → Azione anziché semplici log transazionali. Quest'ultimo approccio supporta sia il monitoraggio operativo che la conformità normativa, inclusa la EU AI Act.

Un aspetto chiave è la distinzione tra API gateway tradizionali e AI gateway. I gateway tradizionali assumono tre proprietà che i sistemi agentici violano sistematicamente: determinismo dell'output, fallimenti rilevabili a livello di schema, e intent esplicitamente specificato dal client. Gli agenti AI invece producono output variabili da input identici, falliscono in modo semantico (non tecnico) e interpretano obiettivi in autonomia. Queste violazioni aprono superfici di attacco nuove — prompt injection, context poisoning, tool misuse — che le soluzioni di sicurezza tradizionali non possono fronteggiare.

L'articolo descrive un modello di adozione in quattro stadi: partenza con singolo team e provider, diffusione non coordinata tra più team, arrivo di un "forcing function" (un incidente grave, una normativa, o una crisi di costi) che rende insostenibile la frammentazione, e infine consolidamento sotto un layer di controllo centralizzato. Gli autori citano incidenti reali del 2025 per illustrare i rischi della frammentazione: un agente Replit che cancellò un database in produzione durante un periodo di freeze (colpendo 1.196+ aziende), una spesa accidentale di 500 milioni di dollari per Claude in un mese per mancanza di usage limits, e EchoLeak — il primo CVE di prompt injection weaponizzata.

Le limitazioni del pattern includono latenza aggiuntiva per ogni feature abilitata nel gateway, complessità organizzativa nella governance centralizzata, guardrail probabilistici (non deterministici, con rischi di false positive e false negative), e costi operativi di tuning continuo. Il pattern è raccomandato in particolare per sistemi multi-team con molti agenti autonomi; per team singoli con un unico LLM, implementare guardrail direttamente nell'applicazione può essere sufficiente e meno oneroso.

## Codice

Policy di gateway dichiarata come configurazione versionata (stile OPA), che separa la responsabilità tra team di sicurezza e team di piattaforma:

```yaml
agent: code-review-agent
authorization:
  allow:
    - repo.read
    - pr.comment
    - static_scan.run
  deny:
    - repo.push
    - pr.merge
    - secrets.read
tools:
  - github-mcp
  - static_scan
guardrails:
  input:
    - prompt_injection_filter
  output:
    - secret_redaction
routing:
  default: gpt-4o
  fallback: self-hosted-slm
observability:
  logging: semantic
```
