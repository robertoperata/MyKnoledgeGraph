# Credit Risk — Analisi Endpoint KPI

## `GET /partners/credit-risk/v1/transactions/kpis/last?fiscalCode=`

### Panoramica

Questo endpoint calcola i **KPI anti-frode/rischio** sulle transazioni bancarie di un'azienda, a partire dal suo codice fiscale. Orchestrata da `KpiService.kt`, la chiamata è un pipeline sequenziale con fasi parallele interne.

---

## Flusso principale

### 1. Validazione input (Controller)
`KpiController.kt:25-38` — Il `fiscalCode` viene validato via regex prima di entrare nel servizio:
- Formato persona fisica: `BNCADX00A00A000A` (16 caratteri)
- Formato P.IVA: `01100100111` (11 cifre)

---

### 2. Recupero dati di onboarding — FAS (servizio esterno)
`FasAdapter.kt` — Due chiamate GET al servizio *Financial Application Service*:

| Chiamata | Endpoint |
|---|---|
| Lista domande per codice fiscale | `/lending/applications/by-fiscal-code/{fiscalCode}` |
| Dettaglio domanda selezionata | `/lending/{applicationId}` |

**Logica di selezione**: se esistono più domande per lo stesso codice fiscale, viene privilegiata quella con status `ONBOARDING_COMPLETED`, altrimenti la più recente per data.

**Dati estratti e validati**: `vatNumber`, `documentId`, `onboardingId`, `financedAmount`, `beneficialOwners`, dati aziendali. Se uno di questi manca → errore `FasError.missingRequiredData`.

---

### 3. Recupero dati aziendali — CompanyData (servizio esterno)
`CompanyDataAdapter.kt` — GET `/v1/details/{vatNumber}?additionalInformation=balanceSheets`

Recupera bilanci e codice ATECO dell'impresa. Il codice ATECO (prime 2 cifre) viene poi usato per caricare le soglie KPI specifiche per settore merceologico.

---

### 4. Recupero transazioni — Actico (servizio esterno)
`ActicoAdapter.kt` — POST paginato al motore Actico:

- Prima pagina → determina il numero totale di pagine
- Pagine successive → richieste **in parallelo** con limite di concorrenza configurabile (`acticoMaxConcurrentCalls`)
- Tutte le transazioni vengono poi unite in un'unica lista

---

### 5. Calcolo dei 12 KPI (in parallelo)
`KpiService.kt:257-368` — I 12 KPI vengono calcolati in parallelo via coroutine Kotlin:

| KPI | Descrizione |
|---|---|
| `HighRoundNumberTransactions` | Operazioni con cifre tonde elevate |
| `HighCashInFlow` | Elevate operazioni di contante in entrata |
| `HighCashOutFlow` | Elevate operazioni di contante in uscita |
| `InternationalTransfer` | Bonifici da/verso estero |
| `RepeatedTransactions` | Operazioni ripetute |
| `SameCreditDebitTransaction` | Importi credito/debito identici |
| `RepeatedInvoicePayments` | Pagamenti fatture ripetuti |
| `HighAmountTransactions` | Transazioni ad importo elevato |
| `PrepaidTransaction` | Utilizzo elevato di prepagate |
| `MemberRefund` | Rimborso soci |
| `RevenueGrowthAnomaly` | Crescita anomala del fatturato |

La configurazione di ogni KPI e le soglie di alert vengono caricate dal **database** (tabelle `kpi_thresholds`, `kpi_ateco`). Le soglie supportano gli operatori: `GREATER`, `LESS`, `GREATER_EQUAL`, `LESS_EQUAL`, `EQUAL`, `NOT_EQUAL`.

**Filtro ATECO**: prima cerca soglie specifiche per settore, poi fallback a soglie di default.

---

### 6. Persistenza
`KpiRepositoryAdapter.kt` — Salva su DB:
- Il record KPI con tutti i 12 valori calcolati
- Le transazioni Actico associate (batch insert)

---

### 7. Notifica asincrona — Azure Service Bus
`MessengerProvider.kt` — Invio fire-and-forget del messaggio `OnboardingKPICalculationCompleted` con `onboardingId`, `fiscalCode`, `kpiId`. Notifica altri sistemi (es. middleware) del completamento del calcolo.

---

## Mappa dei servizi coinvolti

```
fiscalCode
    │
    ▼
KpiController
    │
    ▼
KpiService ──► FAS (2x GET)          ← dati onboarding
           ──► CompanyData (1x GET)   ← bilanci + ATECO
           ──► Actico (N x POST)      ← transazioni (paginato/parallelo)
           ──► DB: kpi_ateco          ← sezione ATECO
           ──► DB: kpi_thresholds     ← soglie per KPI
           ──► [calcolo 12 KPI in parallelo]
           ──► DB: save KPI
           ──► DB: save transactions
           ──► Azure Service Bus      ← notifica completamento
```

---

## File chiave nel repository

| File | Ruolo |
|---|---|
| `adapter/api/KpiController.kt` | Routing e validazione input |
| `domain/service/KpiService.kt` | Orchestrazione principale |
| `adapter/thirdparty/fas/FasAdapter.kt` | Client FAS (lending applications) |
| `adapter/thirdparty/companydata/CompanyDataAdapter.kt` | Client Company Data (bilanci, ATECO) |
| `adapter/thirdparty/actico/ActicoAdapter.kt` | Client Actico (transazioni) |
| `infrastructure/persistence/adapter/KpiRepositoryAdapter.kt` | Persistenza KPI |
| `infrastructure/persistence/adapter/KpiThresholdsRepositoryAdapter.kt` | Soglie KPI da DB |
| `infrastructure/persistence/adapter/KpiAtecoRepositoryAdapter.kt` | Sezioni ATECO da DB |
