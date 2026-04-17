---
title: FEI Eligible Loans
type: concept
tags:
  - fei
  - tan-discount
  - b2c
  - actico
updated: 2026-04-13
related:
  - "[[integrations/actico]]"
  - "[[concepts/small-corporate]]"
  - "[[tasks/teams-chat-fei-tan-discount]]"
---

# FEI Eligible Loans

## Definizione

Un prestito è **FEI-eligible** quando l'azienda richiedente soddisfa i criteri del Fondo Europeo per gli Investimenti. In tal caso, viene applicato un TAN agevolato (5.9%) sul primo trancio (fino a 25.000 €).

## Feature scope

- Solo canale **B2C** (al momento, task #20426).

## Come viene determinata l'eligibilità

Actico determina l'eligibilità a partire dai dati inviati nell'IFC002. OB backend deve inviare:

| Campo | Dove | Tipo |
|---|---|---|
| `revenue` | `company_data` in IFC002 request | number (€) |
| `number_of_employees` | `company_data` in IFC002 request | number |

Fonte dei dati: Margo/Crif (campo `ecofin.turnover` per revenue, `employees` per dipendenti).

## Output Actico

Nella response di IFC015 (e IFC018, IFC013, IFC008, IFC021), ogni quota ha:

```json
"is_fei_eligible": true | false
```

Se `is_fei_eligible = true`, la **prima quota** nella lista è quella FEI:
- `amount` = 25.000
- `tan` / `max_tan` = 5.9%
- `taeg` calcolato su TAN 5.9%

## Regole di business

- Nessun campo separato `feiDiscountTan` o `feiMaxAmountForPromo` — i valori sono già nei campi standard della quota.
- La fonte del fatturato per l'eligibilità FEI è **Cerved/Margo** (non Crif interna di Actico).

## Ticket di riferimento

- #20426 — Feature principale (B2C only)
- #21216 — Quotation step (IFC015)
- #21214 — Proposal step (IFC018 etc.)
