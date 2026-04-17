# Analisi API: POST /dynamic-onboarding-bff-product/v1/quotes/{bpmId}/configuration

## Obiettivo

Tracciare la chiamata POST usata per ricalcolare i preventivi dopo che il cliente ha scelto una **configurazione specifica** (importo, durata, finalità, tipo tasso, eventuale preamortamento). A differenza della GET `/v1/quotes/{bpmId}` — che restituisce l'intera griglia di quotes calcolate da Actico — questa POST restituisce una lista di quotes **ristretta a un singolo `RateType`** corrispondente alla configurazione richiesta.

---

## Endpoint esposto

```
POST /dynamic-onboarding-bff-product/v1/quotes/{bpmId}/configuration
```

### Request body — `QuoteConfiguration`

```kotlin
data class QuoteConfiguration(
    val covenant: String? = null,                          // es. "ACT"
    val requestedAmount: Int,                              // @NotNull, @Positive
    val finality: String,                                  // @NotBlank, es. "INVESTMENT"
    val instalmentsNumber: Int? = null,                    // @Positive
    val rateType: RateType,                                // @NotNull, es. FIXED / VARIABLE
    val financialPreamortizationMonths: Int? = null,
    val planId: String? = null                             // formato "60_6"
)
```

Vincolo di validazione custom: `planId != null || instalmentsNumber != null` (errore `"Either planLabel or instalmentsNumber must be provided"`).

`QuoteConfiguration.resolved()` spacchetta `planId` (`"60_6"`) in `instalmentsNumber=60` e `financialPreamortizationMonths=6` se non già valorizzati.

| Risposta | Significato |
|---|---|
| `200` | `QuotesResponse<RateType>` con le quotes configurate (rateType scalare, non lista) |
| `500` | Errore interno (Actico, BPM, serializzazione, ecc.) |

Body: `BasicPayloadResponse<QuotesResponse<RateType>>`.

---

![[BPM Integration for Quote-2026-04-17-093409.svg|1197]]

## Flusso — pseudo-code

### 1. `QuoteController.getQuoteListByConfiguration` (bff-product) — HTTP layer
```
riceve POST /v1/quotes/{bpmId}/configuration
        body = QuoteConfiguration (validato con @Valid)
resolveQuote = quoteConfiguration.resolved()        // espande planId in instalmentsNumber/preamortization
chiama quoteService.getQuoteList(bpmId, resolveQuote)

se Right(quotes)  → HTTP 200 con BasicPayloadResponse(quotes)
se Left(errore)   → HTTP 500 con errore serializzato
```

### 2. `QuoteService.getQuoteList(bpmId, quoteConfiguration)` (bff-product) — orchestrazione
```
vatNumber                  = bpmWebClientAdapter.getVatNumber(bpmId)       [BPM]
quotes                     = creditDecisionAdapter.getQuotes(bpmId, quoteConfiguration)  [credit-decision]
isEnabledPreamortization   = appConfigService.getIsFeatureFlagEnabled(
                                 ONBOARDING_LENDING_IS_PREAMORTIZATION_ENABLED)   [AppConfig]
filtered                   = se !isEnabledPreamortization:
                                 quotes.filter { financialPreamortizationMonths == null }
                              altrimenti: quotes
guId                       = bpmWebClientAdapter.getGuid(bpmId)            [BPM]
quotesMessenger.sendQuoteConfigured(
    AvroMessageKey(vatNumber, guId, bpmId),
    filtered.quotes
)                                                                           [Kafka]
return filtered
```

> Nota: **rispetto alla GET `/quotes/{bpmId}`**, questo flusso **non** chiama `saveQuoteListOnBpm` (nessun `PROPOSED_QUOTES` salvato) e **non** emette `OnboardingQuoteCalculated`, bensì `OnboardingQuoteConfigured`.

### 3. `CreditDecisionWebClient.getQuotes(bpmId, quoteConfiguration)` (bff-product → credit-decision)
```
POST {credit-decision}/v1/process/{bpmId}/quotes/configuration?userType=CUSTOMER
     body = QuoteConfiguration (JSON)
deserializza in QuotesResponse<RateType>
```

### 4. `QuoteController.getQuoteListByConfiguration` (credit-decision) — HTTP layer
```
riceve POST /v1/process/{bpmId}/quotes/configuration
        body = QuoteConfiguration
        query = userType (default CUSTOMER)
chiama quoteServicePort.getQuoteList(bpmId, quoteConfiguration, userType)
```

### 5. `QuoteService.getQuoteList` (credit-decision) — orchestrazione
```
docId       = bpmWebClientProviderPort.getDocId(bpmId)                     [BPM]
isUnified   = featureFlagProviderPort.isFeatureEnabled(
                  ONBOARDING_LENDING_IS_UNIFIED_LENDING_FLOW_ENABLED)      [AppConfig]
productCode = se !isUnified → bpmWebClientProviderPort.getProductCode(bpmId)   [BPM]

return creditDecisionProviderPort.configureQuote(
    bpmId, docId, productCode, quoteConfiguration, userType)
```

> Diversamente dal flusso `computeQuotes`, qui **non** viene aggiornata la variabile BPM `MAX_REQUESTABLE_AMOUNT`.

### 6. `CreditDecisionProvider.configureQuote` (credit-decision)
```
acticoResponse = acticoClient.configureQuote(
    createConfigureQuoteRequest(docId, bpmId, productCode, quoteConfiguration),
    userType)                                                               [Actico]
handleActicoError(acticoResponse, ERROR_CONFIGURE_QUOTES)
payload = acticoResponse.output.response.payload
   ?: raise CreditDecisionError.PayloadMissingError("Payload is missing ...")
return payload.toQuotes()
```

### 7. `ActicoClient.configureQuote` (credit-decision → Actico)
```
baseUrl = acticoStrategy.getActicoBaseURL(bpmId)
POST {acticoBaseUrl}{acticoStrategy.configureQuotesActicoPath()}
     body = ActicoRequest<ActicoConfigureRequestPayload>
deserializza in ActicoResponse<ActicoConfigureQuotePayloadResponse>
```

`ActicoConfigureQuotePayloadResponse.quotes: List<Quote<String>>` — nota che qui `rateType` è un singolo `String` (non lista), coerente con il tipo di ritorno finale `Quotes<RateType>` (scalare).

---

## Dove viene settato `isFeiEligible`

Identico al flusso GET: proviene da **Actico**.

1. Actico risponde a `configureQuote` con `payload.quotes[].is_fei_eligible` (Boolean).
2. In `credit-decision`, `Quote` (`domain/models/quote/Quotes.kt:72`) deserializza tramite `@JsonAlias("is_fei_eligible")` → `isFeiEligible`.
3. Il mapper `ActicoResponses.toQuoteWithConvertedRateType()` copia il campo tale e quale (`ActicoResponses.kt:244`).
4. Il BFF non trasforma il flag: l'unica manipolazione è il filtro feature-flag sul preamortamento (`financialPreamortizationMonths`).

Unica differenza rispetto alla GET: nel response type `Quote<RateType>` il `rateType` è **scalare** (FIXED o VARIABLE) invece di `List<RateType>`, perché il cliente ha già scelto la tipologia di tasso in input.

---

## Catena completa

```
HTTP POST /dynamic-onboarding-bff-product/v1/quotes/{bpmId}/configuration
  body: QuoteConfiguration
  → bff-product QuoteController.getQuoteListByConfiguration
    → QuoteConfiguration.resolved() (espande planId)
    → bff-product QuoteService.getQuoteList(bpmId, quoteConfiguration)
      → BpmWebClientAdapter.getVatNumber
      → CreditDecisionWebClient.getQuotes (POST)
        → HTTP POST {credit-decision}/v1/process/{bpmId}/quotes/configuration
          → credit-decision QuoteController.getQuoteListByConfiguration
            → credit-decision QuoteService.getQuoteList(..., quoteConfiguration, userType)
              → BpmWebClientProviderPort.getDocId
              → FeatureFlagProviderPort (UNIFIED_LENDING_FLOW)
              → BpmWebClientProviderPort.getProductCode (se !unified)
              → CreditDecisionProvider.configureQuote
                → ActicoClient.configureQuote
                  → Actico → payload.quotes[].is_fei_eligible   ← SORGENTE
      → AppConfigService (PREAMORTIZATION flag) + filter
      → BpmWebClientAdapter.getGuid
      → QuotesMessenger.sendQuoteConfigured (Kafka: OnboardingQuoteConfigured)
  ← BasicPayloadResponse<QuotesResponse<RateType>>
```

---

## Tabella chiamate esterne (ordine temporale)

| # | Servizio | Endpoint / operazione | Condizione | Dato restituito / scopo |
|---|----------|-----------------------|------------|-------------------------|
| 1 | **BPM (Camunda)** | `getVatNumber(bpmId)` | Sempre | Partita IVA del processo |
| 2 | **credit-decision** | `POST /v1/process/{bpmId}/quotes/configuration?userType=CUSTOMER` con body `QuoteConfiguration` | Sempre | `QuotesResponse<RateType>` |
| 2.1 | **BPM (Camunda)** | `getDocId(bpmId)` | Sempre (in credit-decision) | Document ID Actico |
| 2.2 | **AppConfig** | feature flag `ONBOARDING_LENDING_IS_UNIFIED_LENDING_FLOW_ENABLED` | Sempre | Se true → productCode non viene caricato |
| 2.3 | **BPM (Camunda)** | `getProductCode(bpmId)` | Se unified flow disabilitato | Product code per Actico |
| 2.4 | **Actico** | `POST {acticoBaseUrl}{configureQuotesActicoPath}` con `ActicoConfigureRequestPayload` | Sempre | `payload.quotes[]` con `is_fei_eligible`, `rateType` scalare |
| 3 | **AppConfig** | feature flag `ONBOARDING_LENDING_IS_PREAMORTIZATION_ENABLED` | Sempre | Filtro sulle quotes con `financialPreamortizationMonths` valorizzato |
| 4 | **BPM (Camunda)** | `getGuid(bpmId)` | Sempre | Chiave Avro per Kafka |
| 5 | **Kafka** | topic `OnboardingQuoteConfigured` | Sempre | Evento di configurazione quote |

---

## Risposta

```kotlin
data class QuotesResponse<T>(val quotes: List<Quote<T>>)

// T = RateType (scalare), non List<RateType> come nel flusso GET /quotes/{bpmId}
data class Quote<RateType>(
    val quoteId: String,
    val productName: String? = null,
    val amount: Int,
    val maxAmount: Int? = null,
    val instalmentsNumber: Int,
    val instalmentAmount: Double,
    val finality: String?,
    val rateType: RateType,          // scalare: FIXED o VARIABLE
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

## Differenze rispetto a `GET /v1/quotes/{bpmId}`

| Aspetto | GET `/quotes/{bpmId}` | POST `/quotes/{bpmId}/configuration` |
|---|---|---|
| Input utente | Nessuno (importo risolto da promoCode/partner onboarding) | `QuoteConfiguration` nel body |
| Risoluzione importo | `getPromoCode` + `getActiveWarranty` oppure `loadPartnerOnboardingRequest` | `requestedAmount` passato direttamente nel body |
| Chiamata Actico | `computeQuotes` | `configureQuote` |
| Tipo di ritorno | `QuotesResponse<List<RateType>>` (rateType lista) | `QuotesResponse<RateType>` (rateType scalare) |
| Salvataggio BPM | `MAX_REQUESTABLE_AMOUNT` (credit-decision) + `PROPOSED_QUOTES` (bff) | **Nessun** `saveProcessVariable` |
| Evento Kafka | `OnboardingQuoteCalculated` | `OnboardingQuoteConfigured` |
| Chiamata `promoCodes` | Sì (se promo presente) | **No** |
| Telemetria custom | `BpmVariableSaveErrorEvent` su errore save | Nessuna |

---

## Note

- **Nessuna persistenza su BPM**: il flusso `/configuration` è pensato per il _what-if_ interattivo del cliente sulla UI — non salva nulla perché il piano effettivamente scelto viene poi confermato via `PUT/POST /v1/quotes/{bpmId}` (selectQuote).
- **`PayloadMissingError`**: a differenza di `computeQuotes`, qui `payload` può essere `null` e viene esplicitamente gestito come errore `CreditDecisionError.PayloadMissingError` in `CreditDecisionProvider.configureQuote`.
- **`planId`**: stringa nel formato `"{instalments}_{preamortization}"` (es. `"60_6"`). Il metodo `resolved()` la decodifica per valorizzare `instalmentsNumber` e `financialPreamortizationMonths`.
- **`covenant` nel body**: differenzia la configurazione per covenant (es. `"ACT"`); usato da Actico per selezionare la tabella di pricing corretta.
- **`isFeiEligible` non è condizionato da `UNIFIED_LENDING_FLOW`**: il feature flag controlla solo il caricamento del `productCode`. Il flag `isFeiEligible` è sempre deciso da Actico.

---

## File chiave nel codice

### bff-product (`dynamic-onboarding-bff-product`)
- **Controller**: `src/main/kotlin/it/aidexa/product/quotes/QuoteController.kt` (`getQuoteListByConfiguration`, linea ~101)
- **Service**: `src/main/kotlin/it/aidexa/product/quotes/QuoteService.kt` (`getQuoteList(bpmId, quoteConfiguration)`, linea ~65)
- **Request model**: `src/main/kotlin/it/aidexa/product/quotes/QuoteConfiguration.kt`
- **Response model**: `src/main/kotlin/it/aidexa/product/quotes/QuoteResponse.kt`
- **Adapter credit-decision**: `src/main/kotlin/it/aidexa/product/providers/creditdecision/CreditDecisionAdapter.kt`
- **WebClient credit-decision**: `src/main/kotlin/it/aidexa/product/providers/creditdecision/CreditDecisionWebClient.kt` (`getQuotes(..., quoteConfiguration)`)
- **Messenger Kafka**: `src/main/kotlin/it/aidexa/product/datalake/QuotesMessenger.kt` (`sendQuoteConfigured`, linea ~46)

### credit-decision
- **Controller**: `src/main/kotlin/it/aidexa/creditdecision/adapter/api/quote/QuoteController.kt` (`getQuoteListByConfiguration`, linea ~60)
- **Service**: `src/main/kotlin/it/aidexa/creditdecision/domain/service/QuoteService.kt` (`getQuoteList(..., quoteConfiguration, userType)`, linea ~49)
- **Request model**: `src/main/kotlin/it/aidexa/creditdecision/domain/models/quote/QuoteConfiguration.kt`
- **Provider Actico**: `src/main/kotlin/it/aidexa/creditdecision/adapter/provider/creditdecision/CreditDecisionProvider.kt` (`configureQuote`, linea ~94)
- **Actico client**: `src/main/kotlin/it/aidexa/creditdecision/adapter/thirdparty/actico/ActicoClient.kt` (`configureQuote`, linea ~78)
- **Quote model (con `@JsonAlias("is_fei_eligible")`)**: `src/main/kotlin/it/aidexa/creditdecision/domain/models/quote/Quotes.kt`
- **Mapping Actico → domain**: `src/main/kotlin/it/aidexa/creditdecision/adapter/thirdparty/actico/model/response/ActicoResponses.kt` (`ActicoConfigureQuotePayloadResponse.toQuotes()`)
