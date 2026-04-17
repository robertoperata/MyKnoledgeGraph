---
title: "Chat Teams: TAN Discount FEI Eligible Loans & Small Corporate"
type: task
tags:
  - fei
  - tan-discount
  - small-corporate
  - actico
  - ifc002
  - ifc015
  - margo-crif
updated: 2026-04-13
related:
  - "[[integrations/actico]]"
  - "[[concepts/fei-eligible-loans]]"
  - "[[concepts/small-corporate]]"
  - "[[integrations/margo-crif]]"
---

# Chat Teams: TAN Discount FEI & Small Corporate

Sintesi di una conversazione Teams tra Viorel Ieremias, Guglielmo Pelino e Simone Squillace.

---

## 1. passiveInquiryFeePercentagePerAmount — inequality operator (task #20406)

L'operatore di disuguaglianza nei tier è `>` (stretto), **non** `>=`.

**Motivazione (Ennio):** la soglia di cambio tier è stata spostata: prima 100.000 cadeva nel tier a fee ridotta, ora ci cade 100.001. Quindi si usa `>` per escludere il valore soglia stesso dal tier ridotto.

---

## 2. TAN Discount per prestiti FEI-eligible (task #20426, #21216, #21214)

Feature **solo B2C** (al momento).

### 2a. IFC002 Check Company — API v8

La versione dell'endpoint Actico è stata portata a **8**:

```
POST /external/actico/engine/v1/executions/aidexa.ifc-facade/8/aix-ifc-incoming-facade/AIX_IFC_Incoming_Facade/IFC002_Check_Company/START_IFC002_Check_Company
```

Il body della request ora include 2 nuovi campi nel blocco `company_data`:

| Campo | Tipo | Note |
|---|---|---|
| `revenue` | number | Fatturato annuo (in euro); era `fatturato` in bozza |
| `number_of_employees` | number | Numero dipendenti; era `numDipendenti` in bozza |

> **Convenzione nomi:** Guglielmo ha richiesto di usare nomi inglesi coerenti con il resto dell'endpoint. La proposta `annual_turnover` / `number_of_employees` è stata scartata a favore di `revenue` / `number_of_employees` (coerente con quanto usato in IFC004/IFC005).

### 2b. IFC015 Compute Quotes — is_fei_eligible

Simone aggiunge il campo `is_fei_eligible: bool` a ciascuna quota nella response di IFC015 (e di IFC018, IFC013, IFC008, IFC021):

- Se `is_fei_eligible = true`, la **prima quota** della lista è quella FEI con TAN agevolato.
- I valori FEI sono già codificati nei campi standard della quota:
  - `amount`: 25.000 (massimo FEI)
  - `tan` / `max_tan`: 5.9%
  - `taeg` / `max_taeg`: calcolato su TAN 5.9%
- **Non vengono aggiunti** campi separati `feiDiscountTan` o `feiMaxAmountForPromo` — le informazioni sono ridondanti con `amount` e `tan`.

---

## 3. Small Corporate — determinazione all'onboarding

### Come Actico la determina

IFC002 restituisce `output_type` nel payload. I valori rilevanti:

| Codice | Significato |
|---|---|
| `06p` | Small Corporate Proactive |
| `06r` | Small Corporate Reactive |
| `01`–`05` | Non Small Corporate (Digital, Confidi, Partners…) |

### Come OB Backend la determina (senza Actico)

**Non serve chiamare Actico** per capire se un'azienda è Small Corporate: OB Backend usa il **fatturato da Margo/Crif**.

- Campo: `ecofin.turnover` nella response di Margo/Crif
- Soglia: `turnover > 20.000.000` → Small Corporate
- La decisione avviene **prima** di chiamare Actico; se il promocode non ha `smallCorporateEnabled = true`, il backend non deve nemmeno procedere.

> Guglielmo: *"back-end should not even call Actico in my opinion if the promocode association is wrong — reading Actico's IFC002 response would be too late"*

### turnover null in Margo/Crif

Esiste la possibilità che `turnover` sia null nella response Margo/Crif (il codice OB lo gestisce già come nullable). In quel caso la decisione di Actico (che usa Margo internamente) rimane la fonte di verità. Da chiarire con Francesco.

---

## 4. Versioning Actico — commento di Francesco (PR)

Francesco ha commentato che il backend mantiene versioni Actico tramite feature flag e suggerisce di eliminare il pattern:

> *"I don't think we need so many versions. Please have a quick catchup with data team to clean this pattern matching."*

**Posizione team Actico (Simone/Guglielmo):**
- Tutte le versioni esistenti sono ancora funzionanti.
- Non è elegante gestirle con feature flag lato OB.
- La raccomandazione è di mantenere sempre le **ultime 2 versioni** e fare clean-up del codice FF dopo il rilascio in PROD.

---

## Entità coinvolte

- [[integrations/actico]] — IFC002 (v8), IFC015, IFC018, IFC013, IFC008, IFC021
- [[integrations/margo-crif]] — campo `ecofin.turnover`
- [[concepts/fei-eligible-loans]] — eligibility, TAN agevolato
- [[concepts/small-corporate]] — soglia fatturato, output_type Actico
- [[concepts/inquiry-fee]] — inequality `>` nei tier
