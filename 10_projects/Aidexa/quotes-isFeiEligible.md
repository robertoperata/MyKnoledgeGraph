# Analisi API: GET /dynamic-onboarding-bff-product/v1/quotes/{bpmId}

## Obiettivo

Tracciare la chiamata GET dei preventivi (quotes) esposta dal `dynamic-onboarding-bff-product`, evidenziando le chiamate esterne e in particolare la sorgente del flag `isFeiEligible` nel payload di ciascun `Quote`.

---

## Endpoint esposto

```
GET /dynamic-onboarding-bff-product/v1/quotes/{bpmId}
```

| Risposta | Significato |
|---|---|
| `200` | Lista di quotes calcolate (con filtro preamortization applicato) |
| `500` | Errore interno (`BpmVariableSaveException`, errore Actico, errore BPM, ecc.) |

Body di risposta: `BasicPayloadResponse<QuotesResponse<List<RateType>>>`.

---

![[BPM-Driven Quote-2026-04-17-092311.svg|1197]]

## Flusso — pseudo-code

### 1. `QuoteController` (bff-product) — layer HTTP
```
riceve GET /v1/quotes/{bpmId}
chiama quoteService.getQuoteList(bpmId)

se Right(quotes)  → HTTP 200 con BasicPayloadResponse(quotes)
se Left(errore)   → HTTP 500 con errore serializzato
```

### 2. `QuoteService.getQuoteList(bpmId)` (bff-product) — orchestrazione
```
vatNumber     = bpmWebClientAdapter.getVatNumber(bpmId)         [BPM]
requestedAmt  = getRequestedAmount(bpmId, vatNumber)
quotes        = creditDecisionAdapter.getQuotes(bpmId, requestedAmt)   [credit-decision]
filtered      = filtra per feature flag ONBOARDING_LENDING_IS_PREAMORTIZATION_ENABLED  [AppConfig]
saveQuoteListOnBpm(filtered, bpmId)                             [BPM: PROPOSED_QUOTES]
guId          = bpmWebClientAdapter.getGuid(bpmId)              [BPM]
quotesMessenger.sendQuoteCalculated(AvroMessageKey(vatNumber, guId, bpmId), filtered.quotes)  [Kafka]
return filtered
```

### 3. `getRequestedAmount` (bff-product) — risoluzione importo richiesto
```
promoCode = bpmWebClientAdapter.getPromoCode(bpmId)             [BPM]
se promoCode != null:
    warranty = promoCodesService.getActiveWarranty(promoCode, vatNumber, null)  [PromoCodes service]
    return warranty?.maxAmountWithoutExpenses?.toInt()
altrimenti:
    partnerOnboardingRequest = bpmWebClientAdapter.loadPartnerOnboardingRequest(bpmId)  [BPM]
    return partnerOnboardingRequest?.partnerFinancingData?.requestedAmount
```

### 4. `CreditDecisionWebClient.getQuotes` (bff-product → credit-decision)
```
GET {credit-decision}/v1/process/{bpmId}/quotes
    ?userType=CUSTOMER
    &requestedAmount={requestedAmount}
deserializza in QuotesResponse<List<RateType>>
```

### 5. `QuoteController.getQuoteList` (credit-decision) — HTTP layer
```
riceve GET /v1/process/{bpmId}/quotes
chiama quoteServicePort.getQuoteList(bpmId, requestedAmount, userType=CUSTOMER)
```

### 6. `QuoteService.getQuoteList` (credit-decision) — orchestrazione
```
docId        = bpmWebClientProviderPort.getDocId(bpmId)         [BPM]
isUnified    = featureFlagProviderPort.isFeatureEnabled(
                   ONBOARDING_LENDING_IS_UNIFIED_LENDING_FLOW_ENABLED)   [AppConfig]
productCode  = se !isUnified → bpmWebClientProviderPort.getProductCode(bpmId)   [BPM]
quotes       = creditDecisionProviderPort.computeQuotes(
                   bpmId, docId, productCode, requestedAmount, userType)
maxAmt       = quotes.quotes.maxOf { it.maxAmount }
bpmWebClientProviderPort.saveProcessVariable(bpmId, MAX_REQUESTABLE_AMOUNT, maxAmt)  [BPM]
return quotes
```

### 7. `CreditDecisionProvider.computeQuotes` (credit-decision)
```
acticoResponse = acticoClient.computeQuotes(request, userType)  [Actico]
handleActicoError(acticoResponse, ERROR_COMPUTE_QUOTES)
return acticoResponse.output.response.payload.toQuotes()
```

### 8. `ActicoClient.computeQuotes` (credit-decision → Actico)
```
baseUrl = acticoStrategy.getActicoBaseURL(bpmId)
POST {acticoBaseUrl}{getQuotesActicoPath()}
body = ActicoRequest<ActicoComputeQuotesRequestPayload>
deserializza in ActicoResponse<ActicoComputeQuotePayloadResponse>
```

---

## Dove viene settato `isFeiEligible`

Il flag **non è calcolato da Aidexa**: è un output del motore regole **Actico**.

1. Actico risponde a `computeQuotes` con `payload.quotes[].is_fei_eligible` (Boolean).
2. In `credit-decision`, la `data class Quote` (`domain/models/quote/Quotes.kt:72`) deserializza il JSON con `@JsonAlias("is_fei_eligible")` sul campo Kotlin `isFeiEligible`.
3. La conversione `Quote<List<String>> → Quote<List<RateType>>` in `ActicoResponses.kt:244` copia il campo **tale e quale** tramite `toQuoteWithConvertedRateType()`.
4. La risposta HTTP di `credit-decision` serializza `isFeiEligible` in camelCase.
5. In `bff-product`, la `data class Quote` (`quotes/QuoteResponse.kt:26`) contiene un campo omonimo `isFeiEligible: Boolean?` popolato per passaggio JSON diretto (nessun adapter tra le due classi).
6. Lungo il BFF **non c'è alcuna trasformazione** del flag: l'unico filtro applicato è su `financialPreamortizationMonths` (feature flag `ONBOARDING_LENDING_IS_PREAMORTIZATION_ENABLED`).

Conclusione: per capire quando `isFeiEligible = true` bisogna guardare le regole Actico (modello `IFC…` che espone `getQuotesActicoPath()`), non il codice Aidexa.

---

## Catena completa

```
HTTP GET /dynamic-onboarding-bff-product/v1/quotes/{bpmId}
  → bff-product QuoteController.getQuoteList
    → bff-product QuoteService.getQuoteList
      → BpmWebClientAdapter (vatNumber, promoCode | partnerOnboardingRequest, guid)
      → PromoCodesService.getActiveWarranty (se presente promoCode)
      → CreditDecisionWebClient.getQuotes
        → HTTP GET {credit-decision}/v1/process/{bpmId}/quotes
          → credit-decision QuoteController.getQuoteList
            → credit-decision QuoteService.getQuoteList
              → BpmWebClientProviderPort (docId, productCode, saveProcessVariable MAX_REQUESTABLE_AMOUNT)
              → FeatureFlagProviderPort (UNIFIED_LENDING_FLOW)
              → CreditDecisionProvider.computeQuotes
                → ActicoClient.computeQuotes
                  → Actico → payload.quotes[].is_fei_eligible   ← SORGENTE
      → AppConfigService (PREAMORTIZATION flag) + filter
      → BpmWebClientAdapter.saveProcessVariable (PROPOSED_QUOTES)
      → QuotesMessenger.sendQuoteCalculated (Kafka: OnboardingQuoteCalculated)
  ← BasicPayloadResponse<QuotesResponse<List<RateType>>>
```

---

## Tabella chiamate esterne (ordine temporale)

| # | Servizio | Endpoint / operazione | Condizione | Dato restituito / scopo |
|---|----------|-----------------------|------------|-------------------------|
| 1 | **BPM (Camunda)** | `getVatNumber(bpmId)` | Sempre | Partita IVA associata al processo |
| 2 | **BPM (Camunda)** | `getPromoCode(bpmId)` | Sempre | Eventuale promo code |
| 3a | **PromoCodes Service** | `getActiveWarranty(promoCode, vatNumber)` | Se presente promoCode | `maxAmountWithoutExpenses` → requestedAmount |
| 3b | **BPM (Camunda)** | `loadPartnerOnboardingRequest(bpmId)` | Se nessun promoCode | `partnerFinancingData.requestedAmount` |
| 4 | **credit-decision** | `GET /v1/process/{bpmId}/quotes?userType=CUSTOMER&requestedAmount=N` | Sempre | `QuotesResponse<List<RateType>>` |
| 4.1 | **BPM (Camunda)** | `getDocId(bpmId)` | Sempre (in credit-decision) | Document ID Actico |
| 4.2 | **AppConfig** | feature flag `ONBOARDING_LENDING_IS_UNIFIED_LENDING_FLOW_ENABLED` | Sempre | Determina se caricare productCode |
| 4.3 | **BPM (Camunda)** | `getProductCode(bpmId)` | Se unified flow disabilitato | Product code per Actico |
| 4.4 | **Actico** | `POST {acticoBaseUrl}{getQuotesActicoPath}` | Sempre | `payload.quotes[]` incluso `is_fei_eligible` |
| 4.5 | **BPM (Camunda)** | `saveProcessVariable(MAX_REQUESTABLE_AMOUNT)` | Sempre | Persistenza importo massimo |
| 5 | **AppConfig** | feature flag `ONBOARDING_LENDING_IS_PREAMORTIZATION_ENABLED` | Sempre | Filtro sulle quotes con preamortization |
| 6 | **BPM (Camunda)** | `saveProcessVariable(PROPOSED_QUOTES)` | Sempre | Persiste la lista di quotes filtrate |
| 7 | **BPM (Camunda)** | `getGuid(bpmId)` | Sempre | Chiave Avro per il messaggio Kafka |
| 8 | **Kafka** | topic `OnboardingQuoteCalculated` | Sempre | Evento con le quotes calcolate |

---

## Risposta

```kotlin
data class QuotesResponse<T>(val quotes: List<Quote<T>>)

data class Quote<T>(
    val quoteId: String,
    val productName: String? = null,
    val amount: Int,
    val maxAmount: Int? = null,
    val instalmentsNumber: Int,
    val instalmentAmount: Double,
    val finality: String?,
    val rateType: T,
    val tan: Double,
    val maxTan: Double,
    val taeg: Double,
    val maxTaeg: Double,
    val financialPreamortizationMonths: Int? = null,
    val preamortizationDaysForInstallment: Int? = null,
    val amortizationType: String? = null,
    val repaymentFrequency: String? = null,
    val minInstalmentAmount: BigDecimal? = null,
    val maxInstalmentAmount: BigDecimal? = null,
    val isFeiEligible: Boolean? = null,   // ← da Actico (JsonAlias "is_fei_eligible")
)
```

---

## Note

- **Actico è l'unica fonte di verità per `isFeiEligible`**: nessuna logica in Aidexa setta, valida o altera il flag. Per modificarne il comportamento bisogna agire sulle regole Actico.
- **Due filtri feature-flag distinti**:
  - `ONBOARDING_LENDING_IS_UNIFIED_LENDING_FLOW_ENABLED` (credit-decision) → decide se passare `productCode` ad Actico.
  - `ONBOARDING_LENDING_IS_PREAMORTIZATION_ENABLED` (bff-product) → filtra via le quotes con `financialPreamortizationMonths` valorizzato.
- **Error handling** in `QuoteService.getQuoteList` del BFF: un errore su `saveQuoteListOnBpm` emette un custom event su Application Insights (`BpmVariableSaveErrorEvent`) e solleva `BpmVariableSaveException`.
- **Overlap di modelli**: esistono due `data class Quote` omonime (bff-product e credit-decision) con campi sovrapposti; la propagazione è implicita via JSON, non c'è un mapper esplicito.

---

## File chiave nel codice

### bff-product (`dynamic-onboarding-bff-product`)
- **Controller**: `src/main/kotlin/it/aidexa/product/quotes/QuoteController.kt`
- **Service**: `src/main/kotlin/it/aidexa/product/quotes/QuoteService.kt`
- **Response model (con `isFeiEligible`)**: `src/main/kotlin/it/aidexa/product/quotes/QuoteResponse.kt`
- **Adapter verso credit-decision**: `src/main/kotlin/it/aidexa/product/providers/creditdecision/CreditDecisionAdapter.kt`
- **WebClient credit-decision**: `src/main/kotlin/it/aidexa/product/providers/creditdecision/CreditDecisionWebClient.kt`
- **Messenger Kafka**: `src/main/kotlin/it/aidexa/product/datalake/QuotesMessenger.kt`

### credit-decision
- **Controller**: `src/main/kotlin/it/aidexa/creditdecision/adapter/api/quote/QuoteController.kt`
- **Service**: `src/main/kotlin/it/aidexa/creditdecision/domain/service/QuoteService.kt`
- **Provider Actico**: `src/main/kotlin/it/aidexa/creditdecision/adapter/provider/creditdecision/CreditDecisionProvider.kt`
- **Actico client**: `src/main/kotlin/it/aidexa/creditdecision/adapter/thirdparty/actico/ActicoClient.kt`
- **Quote model (con `@JsonAlias("is_fei_eligible")`)**: `src/main/kotlin/it/aidexa/creditdecision/domain/models/quote/Quotes.kt`
- **Mapping Actico → domain**: `src/main/kotlin/it/aidexa/creditdecision/adapter/thirdparty/actico/model/response/ActicoResponses.kt`
