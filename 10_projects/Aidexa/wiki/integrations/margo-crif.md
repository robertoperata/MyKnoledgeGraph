---
title: Margo / Crif
type: integration
tags:
  - margo
  - crif
  - turnover
  - small-corporate
  - ecofin
updated: 2026-04-13
related:
  - "[[concepts/small-corporate]]"
  - "[[concepts/fei-eligible-loans]]"
  - "[[tasks/teams-chat-fei-tan-discount]]"
---

# Margo / Crif

## Cos'è

Margo è il servizio di dati aziendali basato su Crif. OB Backend lo chiama per ottenere informazioni sull'azienda richiedente, inclusi dati economico-finanziari.

## Endpoint

```
POST /prospecting/search
```

Request: filtri per VAT code + lista di `dataPacketList` (ecofin, employees, atecoClassification…).

## Dati rilevanti per Aidexa

### ecofin.turnover

Fatturato annuo dell'azienda in euro.

- Usato per determinare se l'azienda è [[concepts/small-corporate]] (`> 20.000.000`).
- Usato come campo `revenue` nella request IFC002 v8 di [[integrations/actico]] per eligibilità FEI.
- Può essere **null** — il codice OB lo gestisce già come nullable.

```json
"ecofin": {
  "turnover": 1233399.0,
  "turnoverRange": { "code": "TR7", "description": "1000000 - 4999999" },
  "turnoverTrend": 42.35,
  "turnoverYear": 2024
}
```

### employees

Numero dipendenti — usato come campo `number_of_employees` nella request IFC002 v8.

## Note

- La documentazione YAML completa è `margo_1-0-0_reference.yaml` (condivisa da Guglielmo).
- Per accesso e dettagli contattare Francesco (OB Backend già la chiama).
- Actico usa internamente Margo/Crif per i propri calcoli, ma la sorgente per i nuovi campi IFC002 deve venire esplicitamente da OB Backend via Margo/Crif (non da Cerved).
