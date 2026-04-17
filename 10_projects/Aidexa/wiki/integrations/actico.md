---
title: Actico — Credit Decision Engine
type: integration
tags:
  - actico
  - ifc
  - credit-decision
  - pricing
  - fei
  - small-corporate
updated: 2026-04-13
related:
  - "[[concepts/fei-eligible-loans]]"
  - "[[concepts/small-corporate]]"
  - "[[concepts/inquiry-fee]]"
  - "[[tasks/teams-chat-fei-tan-discount]]"
---

# Actico — Credit Decision Engine

## Cos'è

Actico è il motore esterno di credit decision e pricing. Espone un'API REST versionata. Il modello principale è `AIX_IFC_Incoming_Facade`.

Base URL (test): `https://cdp.tst.aidexa.actico.cloud`

---

## Endpoint principali (IFC)

### IFC002 — Check Company

```
POST /external/actico/engine/v1/executions/aidexa.ifc-facade/{version}/aix-ifc-incoming-facade/AIX_IFC_Incoming_Facade/IFC002_Check_Company/START_IFC002_Check_Company
```

**Versione corrente:** 8 (dalla feature FEI, precedente era 7)

**Scopo:** verifica l'azienda, assegna `output_type` (canale), calcola metascore.

**Request body — `company_data`:**

| Campo | Tipo | Note |
|---|---|---|
| `name` | string | |
| `activity_status` | string | |
| `ateco_cd` | string | |
| `constitution_date` | string | |
| `detailed_form_type` | string | |
| `form_type` | string | |
| `subject_id` | string | |
| `revenue` | number | **Nuovo (v8)** — fatturato annuo in €, da Margo/Crif |
| `number_of_employees` | number | **Nuovo (v8)** — da Margo/Crif |

**Response — `payload`:**

| Campo | Tipo | Note |
|---|---|---|
| `output_type` | string | Canale assegnato (01–06p/06r). Vedi [[concepts/small-corporate]] |
| `outcome` | string | OK / KO |
| `metascore` | number | |
| `overall_pd` | number | nullable |
| `ms_class` | number | nullable |

---

### IFC015 — Compute Quotes

Calcola le quote di finanziamento.

**Response — ogni elemento di `quotes`:**

| Campo | Tipo | Note |
|---|---|---|
| `quote_id` | string | |
| `amount` | number | |
| `tan` / `max_tan` | number | |
| `taeg` / `max_taeg` | number | |
| `is_fei_eligible` | bool | **Nuovo** — true se quota FEI agevolata |
| ... | | altri campi esistenti |

Se `is_fei_eligible = true`, la prima quota nella lista è la quota FEI (TAN 5.9%, amount ≤ 25.000).

---

### Altri endpoint con is_fei_eligible

- IFC018
- IFC013
- IFC008
- IFC021

---

## Versioning

Actico aggiunge una nuova versione dell'API per ogni feature major. OB Backend gestisce la versione con feature flag — Francesco ha suggerito di fare clean-up dopo ogni rilascio in PROD e mantenere solo le ultime 2 versioni.

---

## Output Type — mappatura completa

| Codice | Canale |
|---|---|
| `01` | Digital |
| `02` | Confidi |
| `03` | Gold & Green Partners |
| `04` | Partners |
| `05` | Upsell |
| `06p` | Small Corporate Proactive |
| `06r` | Small Corporate Reactive |
