# Fatturato nel company-data-service

## Endpoint 1: `GET /v1/details/{vatNumber}`

Restituisce l'oggetto `Company` completo. Il fatturato si trova in **due campi**:

**A) `economicAndDimensionData.annualRevenueInThousands`** (Double?, in migliaia di EUR)
- E' il fatturato "sintetico" a livello aziendale
- Fonte: CRIF (`ecofin.annualRevenueInThousands`) oppure Cerved (`datiEconomiciDimensionali.fatturato * 1000`)

**B) `balanceSheets[].annualRevenueInThousands`** (Double?, in migliaia di EUR)
- Disponibile solo se si passa il query param `?additionalInformation=balanceSheets`
- Estratto dalle voci di bilancio cercando la voce `"Ricavi netti"` nella sezione `INCOME_STATEMENT`
- Ogni elemento della lista corrisponde a un anno di bilancio diverso

---

## Endpoint 2: `GET /v1/details/{fiscalCode}/balance-sheets`

Restituisce direttamente `List<BalanceSheetData>` con il campo `annualRevenueInThousands` per ogni anno di bilancio, piu' tutte le voci contabili dettagliate (`entries`).

---

## Strategia di fallback CRIF -> Cerved

La logica e' in `FallbackCompanyDataStrategy` (`companyDataStrategies.kt`):

```
1. Prova CRIF  →  ecofin.annualRevenueInThousands (gia' in migliaia)
       |
       ↓ (se CRIF fallisce o ritorna null)
       |
2. Prova CERVED  →  datiEconomiciDimensionali.fatturato × 1000
       |
       ↓
3. Persiste il risultato nel DB (tabella company, colonna annual_revenue_in_thousands)
```

Entrambi i provider convergono nello stesso modello `EconomicAndDimensionData(annualRevenueInThousands, numberOfEmployees)`.

---

## Differenza rispetto a quello che fa oggi la Products API

| | Products API (BFF) | company-data-service |
|---|---|---|
| **Fonte fatturato** | Solo Cerved (entityProfile) | CRIF (primaria) + Cerved (fallback) |
| **Caching/DB** | Nessuno | Persiste in PostgreSQL |
| **Bilanci dettagliati** | No | Si (voci contabili con "Ricavi netti") |
| **Storico anni** | Solo ultimo disponibile | Lista di bilanci multi-anno |
| **Campo** | `datiEconomiciDimensionali.fatturato` (Int) | `annualRevenueInThousands` (Double, in migliaia) |

---

## Chiamata da fare come fallback

Se Cerved non restituisce il fatturato nella Products API, la chiamata al company-data-service sarebbe:

```
GET /company-data-service/v1/details/{vatNumber}
```

Il fatturato si troverebbe in `response.economicAndDimensionData.annualRevenueInThousands`. Se servisse anche il dettaglio bilanci:

```
GET /company-data-service/v1/details/{vatNumber}?additionalInformation=balanceSheets
```

---

## Modelli dati chiave

### Company
```kotlin
data class Company(
    val vatNumber: String,
    val companyName: String?,
    val atecoCode: String?,
    val legalForm: LegalForm?,
    val activityStatus: String?,
    val constitutionDate: LocalDate?,
    val balanceSheetDate: LocalDate?,
    val balanceSheets: List<BalanceSheetData>?,
    val isActive: Boolean,
    val economicAndDimensionData: EconomicAndDimensionData?,
    // ... address, contact, etc.
)
```

### EconomicAndDimensionData
```kotlin
data class EconomicAndDimensionData(
    val annualRevenueInThousands: Double? = null,  // FATTURATO in migliaia
    val numberOfEmployees: Int? = null,
)
```

### BalanceSheetData
```kotlin
data class BalanceSheetData(
    val balanceType: String,
    val closingDate: String,
    val reclassificationScheme: String,
    val legalDuration: Int,
    val incomeStatementTotalValue: Double,
    val assetsBalanceSheetTotalValue: Double,
    val liabilitiesBalanceSheetTotalValue: Double,
    val equitySummaryTotalValue: Double,
    val annualRevenueInThousands: Double?,  // FATTURATO da "Ricavi netti"
    val entries: List<BalanceSheetEntryData>?,
)
```

### Campi fatturato nei vari livelli

| Location | Field | Source | Type | Note |
|----------|-------|--------|------|------|
| Company | `economicAndDimensionData.annualRevenueInThousands` | CRIF/Cerved | `Double?` | Fatturato sintetico |
| BalanceSheetData | `annualRevenueInThousands` | Voci di bilancio | `Double?` | Calcolato da entries |
| CompanyEntity (DB) | `annual_revenue_in_thousands` | Domain | `BigDecimal?` | Valore persistito |
| CRIF | `ecofin.annualRevenueInThousands` | CRIF API | `Double?` | Gia' in migliaia |
| Cerved | `datiEconomiciDimensionali.fatturato` | Cerved API | `Double?` | Va moltiplicato x1000 |
| BalanceSheetEntryData | `itemValue` (dove itemDescription = "Ricavi netti") | Bilancio | `Double?` | Va moltiplicato x1000 |
