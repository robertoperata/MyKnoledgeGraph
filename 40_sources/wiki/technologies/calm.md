---
title: CALM — Common Architecture Language Model
type: technology
tags: [thread6-security, thread3-cloud]
sources:
  - "[[infoq_secure-connectivity-api-calm]]"
updated: 2026-04-17
related:
  - "[[concepts/architectural-governance]]"
  - "[[technologies/kubernetes]]"
  - "[[concepts/independent-deployability]]"
---

# CALM — Common Architecture Language Model

## Cos'è

Framework open source sviluppato attraverso **FINOS** (Fintech Open Source Foundation) per descrivere architetture sicure come codice. Fornisce un linguaggio comune tra sviluppatori, architetti e team di sicurezza.

> "What CALM does is put an option between the level 3 and 4 diagram of C4 architecture, filling the gap between high-level diagrams and actual infrastructure code."

## Componenti

| Componente | Descrizione |
|---|---|
| **Core Model** | Specifica basata su JSON Schema per descrivere architetture |
| **CLI Tools** | `calm-generate`, `calm-validate` per i developer |
| **Patterns** | Template riutilizzabili con sicurezza integrata |
| **Controls** | Requisiti non funzionali modellati con configurazioni specifiche |

## Concetti chiave

- **Nodes** — componenti (servizi, database, cluster)
- **Relationships** — connessioni tra componenti
- **Controls** — catturano requisiti di sicurezza, performance e compliance

## Workflow

```
Pattern Creation (architetti)
    → calm-generate (developer istanziano il pattern)
    → calm-validate (verifica completezza e correttezza)
    → Template Bundles (generano manifest Kubernetes)
    → Deployment
```

## Il problema che risolve: i due gap

**Gap Sviluppatore–Infrastruttura**: spostare la responsabilità "a sinistra" senza supporto crea attrito.

**Gap Sviluppatore–Sicurezza**: la sicurezza diventa un gate bloccante invece di un principio guida. I developer fanno fatica a "fare la cosa giusta" perché il percorso sicuro non è il percorso più semplice.

**Pattern:** i developer compilano nomi di immagini e fanno selezioni guidate → CALM genera manifest Kubernetes con network policies per la micro-segmentazione, senza richiedere expertise di sicurezza.

## Obiettivo dichiarato

Ridurre i cicli di deployment da **6 mesi a 2 ore** attraverso review di sicurezza automatizzate e percorsi di implementazione chiari.

## Collocazione nell'ecosistema

- Complementare a **Backstage** (Spotify) per il developer portal
- Si inserisce nel movimento Security as Code / Policy as Code (OPA, Kyverno)
- Usato in contesto finanziario altamente regolamentato (Morgan Stanley) → alta rilevanza enterprise

## Connessioni

- [[concepts/architectural-governance]] — CALM è un'implementazione di Architecture as Code
- [[technologies/kubernetes]] — i pattern CALM generano manifest Kubernetes sicuri
- [[concepts/independent-deployability]] — i paved path CALM abilitano deploy self-service con guardrail
