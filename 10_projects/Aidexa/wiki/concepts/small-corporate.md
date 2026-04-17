---
title: Small Corporate
type: concept
tags:
  - small-corporate
  - actico
  - margo-crif
  - output-type
  - onboarding
updated: 2026-04-13
related:
  - "[[integrations/actico]]"
  - "[[integrations/margo-crif]]"
  - "[[tasks/teams-chat-fei-tan-discount]]"
---

# Small Corporate

## Definizione

Una categoria di cliente/azienda con fatturato superiore a **20.000.000 €**. Viene trattata con un percorso di onboarding dedicato (proactive o reactive).

## Come si identifica un'azienda Small Corporate

### Via Actico (IFC002)

Il campo `output_type` nella response di IFC002 indica il canale assegnato:

| Codice | Significato |
|---|---|
| `01` | Digital |
| `02` | Confidi |
| `03` | Gold & Green Partners |
| `04` | Partners |
| `05` | Upsell |
| `06p` | **Small Corporate Proactive** |
| `06r` | **Small Corporate Reactive** |

### Via Margo/Crif (OB Backend)

OB Backend può determinare Small Corporate **prima di chiamare Actico** usando:

- Endpoint Margo: POST `/prospecting/search`
- Campo response: `ecofin.turnover`
- Soglia: `turnover > 20.000.000` → Small Corporate

> Guglielmo: *"take turnover from the response of Margo Crif — if that is above 20MLN, it is Small Corporate"*

**Vantaggio:** evita di chiamare Actico se il promocode non ha `smallCorporateEnabled = true`.

## Feature flag

Il promocode deve avere `smallCorporateEnabled = true` per abilitare il percorso Small Corporate.

## Caso turnover null

Se `ecofin.turnover` è null nella response Margo/Crif (il codice OB già gestisce il nullable), la fonte di verità rimane la risposta Actico. Da approfondire con Francesco.
