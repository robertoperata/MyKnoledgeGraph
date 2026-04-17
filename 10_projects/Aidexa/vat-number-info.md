# Analisi API: GET /v1/vat-number-info/{vatNumber}

## Sequence Diagram

```
Client                  Controller              VatNumberInfoService        CervedAdapter           BPM Adapter              PromoCodesService        LendingPartnerBff
  |                          |                          |                       |                        |                       |                      |
  |-- GET /v1/vat-number-info/{piva} -->                |                       |                        |                       |                      |
  |                          |-- getVatNumberInfo() --->|                       |                        |                       |                      |
  |                          |                          |                       |                        |                       |                      |
  |                          |                    [1] entitySearch(piva) ------->|                        |                       |                      |
  |                          |                          |<-- companyFormClass ---|                        |                       |                      |
  |                          |                          |                       |                        |                       |                      |
  |                          |                    [2] retrieveCovenantName(piva) |                        |                       |                      |
  |                          |                          |                       |                        |                       |                      |
  |                          |                          |--- OldOnboardingCheckHandler ----------------->|                       |                      |
  |                          |                          |   GET /history/process-instance?key={piva}      |                       |                      |
  |                          |                          |<-- List<HistoryProcessInstance> ----------------|                       |                      |
  |                          |                          |                       |                        |                       |                      |
  |                          |                          |   [se processo <30gg] POST /history/variable-instance                  |                      |
  |                          |                          |<-- OnboardingSupportData (promoCode) ----------|                       |                      |
  |                          |                          |                       |                        |   getPromoCodeSafely -->|                      |
  |                          |                          |                       |                        |<-- PromoCode ----------|                      |
  |                          |                          |                       |                        |                       |                      |
  |                          |                          |--- ActivePartnerCheckHandler (se handler 1 fallisce)                   |                      |
  |                          |                          |                       |                        |   warrantyActive(piva)->|                      |
  |                          |                          |                       |                        |<-- List<Warranty> -----|                      |
  |                          |                          |                       |                        |                       |                      |
  |                          |                          |   [se no warranty] ---|------------------------|---getReservationDetails------------------->|
  |                          |                          |                       |                        |<-- ReservationDetail ----------------------|
  |                          |                          |                       |                        |                       |                      |
  |                          |                    [3] retrieveIsSmallCorporate(piva)                      |                       |                      |
  |                          |                          |   GET /history/process-instance?key={piva} ---->|                       |                      |
  |                          |                          |   POST /history/variable-instance (promoCode) ->|                       |                      |
  |                          |                          |                       |                        |   getPromoCodeSafely -->|                      |
  |                          |                          |<-- isSmallCorporateEnabled --------------------|--------------------<--|                      |
  |                          |                          |                       |                        |                       |                      |
  |                          |                    [4] retrieveActivePromocodes(covenantName) [se covenantName != null]            |                      |
  |                          |                          |                       |                        |   GET /promo-codes/active?partnerName=... ->|
  |                          |                          |<-- requireDocumentsCheck (checkboxDocuments) --|--------------------<--|                      |
  |                          |                          |                       |                        |                       |                      |
  |                          |<-- VatNumberInfoResponse |                       |                        |                       |                      |
  |<-- BasicPayloadResponse -|                          |                       |                        |                       |                      |
```

---

## Tabella chiamate esterne (ordine temporale)

| # | Servizio | Endpoint | Condizione | Dato restituito | Fatturato? |
|---|----------|----------|------------|-----------------|------------|
| 1 | **Cerved** | `GET /v1/entitySearch/live?testoricerca={piva}&filtroescludicc=true` | Sempre | `companyFormClass` | **No** |
| 2a | **BPM (Camunda)** | `GET /history/process-instance?processInstanceBusinessKey={piva}` | Sempre | Lista processi (filtro: COMPLETED/EXTERNALLY_TERMINATED) | No |
| 2b | **BPM (Camunda)** | `POST /history/variable-instance` | Se processo < 30 giorni | `OnboardingSupportData` con promoCode | No |
| 2c | **PromoCode Service** | `GET /promo-codes/{code}` | Se promoCode trovato | `PromoCode` con `officialPartnerName` | No |
| 2d | **PromoCode Service** | `GET /warranties/active/{vatNumber}` | Se handler 1 fallisce | `List<Warranty>` | No |
| 2e | **Lending Partner BFF** | `GET /v1/reservation/{vatNumber}` | Se nessuna warranty | `ReservationDetail` con covenant name | No |
| 3a | **BPM (Camunda)** | `GET /history/process-instance?processInstanceBusinessKey={piva}` | Sempre | Lista processi | No |
| 3b | **BPM (Camunda)** | `POST /history/variable-instance` | Se processo trovato | variabile "promoCode" | No |
| 3c | **PromoCode Service** | `GET /promo-codes/{code}` | Se promoCode trovato | `isSmallCorporateEnabled` | No |
| 4 | **PromoCode Service** | `GET /promo-codes/active?partnerName=...&accountType=...` | Se covenantName trovato | `checkboxDocuments` (requireDocumentsCheck) | No |

---

## Fatturato

**Questa API non recupera alcun dato sul fatturato.** Non chiama ne' `entityProfile` di Cerved ne' il `company-data-service`. La chiamata a Cerved (`entitySearch`) recupera solo la `companyFormClass` (forma societaria), non i dati economico-dimensionali.

---

## Risposta

```kotlin
data class VatNumberInfoResponse(
    val companyFormClass: String,            // da Cerved entitySearch
    val covenantName: String? = null,        // da BPM/Warranty/Reservation
    val requireDocumentsCheck: Boolean = false, // da promo code attivi
    val isSmallCorporateEnabled: Boolean? = false, // da BPM + PromoCode
)
```

---

## Note

- **Chain of Responsibility**: il recupero del `covenantName` usa due handler in catena: `OldOnboardingCheckHandler` (cerca nei processi BPM recenti < 30 giorni) e `ActivePartnerCheckHandler` (cerca nelle warranty attive o nelle reservation)
- **`Random.nextBoolean()`**: il parametro `isBankAccount` passato a `retrieveCovenantName` viene generato randomicamente (`VatNumberInfoService.kt:35`) - probabilmente un comportamento legacy o di A/B testing
- **Chiamate duplicate al BPM**: le call 2a e 3a fanno la stessa richiesta a `/history/process-instance` separatamente (non vengono condivise)

---

## File chiave nel codice

- **Controller**: `src/main/kotlin/it/aidexa/businessdata/vatnumber/VatNumberInfoController.kt`
- **Service**: `src/main/kotlin/it/aidexa/businessdata/vatnumber/VatNumberInfoService.kt`
- **PartnerService**: `src/main/kotlin/it/aidexa/businessdata/partners/PartnerService.kt`
- **OldOnboardingCheckHandler**: `src/main/kotlin/it/aidexa/businessdata/partners/handlers/OldOnboardingCheckHandler.kt`
- **ActivePartnerCheckHandler**: `src/main/kotlin/it/aidexa/businessdata/partners/handlers/ActivePartnerCheckHandler.kt`
- **PromoCodesService**: `src/main/kotlin/it/aidexa/businessdata/promocodes/PromoCodesService.kt`
- **CervedWebClientAdapter**: `src/main/kotlin/it/aidexa/businessdata/providers/cerved/CervedWebClientAdapter.kt`
- **BpmWebClientAdapter**: `src/main/kotlin/it/aidexa/businessdata/providers/bpm/BpmWebClientAdapter.kt`
- **LendingPartnerBffWebClient**: `src/main/kotlin/it/aidexa/businessdata/providers/lendingpartnerbff/LendingPartnerBffWebClient.kt`
