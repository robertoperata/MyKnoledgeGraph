	# Analisi API: GET /v1/products/{piva}

## Context

Analisi completa del flusso dell'endpoint `GET /v1/products/{piva}` (e variante `/{piva}/{partner}`) nel progetto `dynamic-onboarding-business-data-bff`. L'obiettivo e' capire le chiamate esterne, le fonti del fatturato, l'ordine temporale delle chiamate, e il ruolo del parametro `isSmallCorporateEnabled`.

---

## Sequence Diagram

```
Client                    ProductController          ProductService           CervedAdapter             FAS Adapter            PromoCodesService         AppConfig
  |                            |                          |                       |                        |                       |                      |
  |-- GET /v1/products/{piva} -->                         |                       |                        |                       |                      |
  |                            |-- products() ----------->|                       |                        |                       |                      |
  |                            |                          |                       |                        |                       |                      |
  |                            |                    [1] entitySearch(piva) ------->|                        |                       |                      |
  |                            |                          |<-- companyForm,       |                        |                       |                      |
  |                            |                          |    idSoggetto --------|                        |                       |                      |
  |                            |                          |                       |                        |                       |                      |
  |                            |                    [2] entityProfile(idSoggetto)->|                        |                       |                      |
  |                            |                          |<-- FATTURATO ---------|                        |                       |                      |
  |                            |                          |                       |                        |                       |                      |
  |                            |                    [3] getLendingProcesses(piva)  |                        |                       |                      |
  |                            |                          |------------------------+----------------------->|                       |                      |
  |                            |                          |<-- List<FinancialApp> +-----------------------|                       |                      |
  |                            |                          |                       |                        |                       |                      |
  |                            |                    [4] warrantyActive(piva) -------------------------------------------->|                      |
  |                            |                          |<-- List<Warranty> --------------------------------------------|                      |
  |                            |                          |                       |                        |                       |                      |
  |                            |                    [5] getPromoData(promoCode) (se warranty trovata) ----->|                      |
  |                            |                          |<-- officialPartnerName -----------------------------|                      |
  |                            |                          |                       |                        |                       |                      |
  |                            |                    [6] retrieveActivePromocodes(partnerName, accountType) ->|                      |
  |                            |                          |<-- List<PromoCode> ----------------------------------|                      |
  |                            |                          |                       |                        |                       |                      |
  |                            |                          |-- filterBySmallCorporate()                     |                       |                      |
  |                            |                          |-- filterCcPromoCodes()                         |                       |                      |
  |                            |                          |-- getEnabledProducts() + filter per partner    |                       |                      |
  |                            |                          |                       |                        |                       |                      |
  |                            |                    [7] isFeatureEnabled(XI_VISIBLE) ------------------------------------------------>|
  |                            |                          |<-- boolean ----------------------------------------------------------------|
  |                            |                          |                       |                        |                       |                      |
  |                            |                          |-- mapProductXWarrant() + associateToPartner()  |                       |                      |
  |                            |                          |-- filterProductsIncompatibleWithRevenue()       |                       |                      |
  |                            |                          |                       |                        |                       |                      |
  |                            |                    [8] isFeatureEnabled(XSACE_ENABLED) --------------------------------------------->|
  |                            |                    [9] isFeatureEnabled(XSACE_ENABLED_FOR_DIGITAL) -------------------------------->|
  |                            |                          |<-- boolean x2 --------------------------------------------------------------|
  |                            |                          |                       |                        |                       |                      |
  |                            |                          |-- filter XSACE (fatturato >= 430 + flags)      |                       |                      |
  |                            |                          |                       |                        |                       |                      |
  |                            |<-- ProductsServiceResponse                       |                        |                       |                      |
  |<-- ProductsResponse -------|                          |                       |                        |                       |                      |
```

---

## Tabella chiamate esterne (ordine temporale)

| # | Servizio | Endpoint chiamato | Condizione | Dato chiave restituito | Contiene info fatturato? |
|---|----------|-------------------|------------|------------------------|--------------------------|
| 1 | **Cerved** | `GET /v1/entitySearch/live?testoricerca={piva}&filtroescludicc=true` | Sempre | `companyForm`, `idSoggetto` | No |
| 2 | **Cerved** | `GET /v1/entityProfile/live?id_soggetto={idSoggetto}` | Solo se `idSoggetto` trovato al passo 1 | **`fatturato`** (in `datiEconomiciDimensionali`) | **Si** - fonte primaria |
| 3 | **Financial Application Service (FAS)** | `GET /lending?vatNumber={piva}` | Sempre | `List<FinancialApplication>` (processi lending ultimo anno) | No |
| 4 | **PromoCode Service** | `GET /warranties/active/{vatNumber}` | Sempre | `List<Warranty>` con `promoCode` | No |
| 5 | **PromoCode Service** | `GET /promo-codes/{promoCode}` | Solo se warranty trovata al passo 4 | `officialPartnerName`, `isSmallCorporateEnabled` | No |
| 6 | **PromoCode Service** | `GET /promo-codes/active?partnerName=...&accountType=...` | Sempre | `List<PromoCode>` con flag `isSmallCorporateEnabled` | No |
| 7 | **Azure App Configuration** | Feature flag `ONBOARDING_IS_XI_PRODUCT_VISIBLE_WITH_PARTNER_ENABLED` | Sempre | `Boolean` | No |
| 8 | **Azure App Configuration** | Feature flag `ONBOARDING_LENDING_IS_XSACE_ENABLED` | Sempre | `Boolean` | No |
| 9 | **Azure App Configuration** | Feature flag `ONBOARDING_LENDING_IS_XSACE_ENABLED_FOR_DIGITAL` | Sempre | `Boolean` | No |

---

## Fonti del fatturato

| Fonte | Servizio | Campo | Dettaglio |
|-------|----------|-------|-----------|
| **Cerved Entity Profile** (unica fonte in questa API) | Cerved API | `datiEconomiciDimensionali.fatturato` (tipo `Int?`) | Valore in migliaia di euro. Estratto al passo 2. Usato per: (1) filtro XSACE (`fatturato >= 430` = 430k EUR), (2) potenziale filtro `revenueGraterThan100K` |

> **Nota**: Il `company-data-service` ha anche dati di fatturato (da CRIF e bilanci), ma **non viene chiamato** da questa API. La fonte del fatturato per `GET /v1/products/{piva}` e' esclusivamente **Cerved**.

### Come viene usato il fatturato nel codice

1. **Linea 76** (`ProductService.kt`): `profile.datiEconomiciDimensionali?.fatturato ?: 0` - estrazione dal profilo Cerved
2. **Linea 156**: `fatturatoCheckPassed = fatturato?.let { it >= 430 } ?: true` - soglia per prodotto XSACE
3. **Linee 158-168**: Filtro XSACE - il prodotto viene rimosso se `!fatturatoCheckPassed` (fatturato < 430k)
4. **Linee 130-135**: `filterProductsIncompatibleWithRevenue(revenueGraterThan100K)` - filtro opzionale basato sul parametro query `revenueGraterThan100K`

---

## Ordine temporale delle chiamate e dati aziendali

Le chiamate avvengono in sequenza (non in parallelo):

```
1. Cerved entitySearch     -> forma societaria, identificativo soggetto
2. Cerved entityProfile    -> FATTURATO (dati economici/dimensionali)
3. FAS getLendingProcesses -> storico processi di lending (ultimo anno)
4. PromoCode warranties    -> garanzie attive / promo code associati
5. PromoCode promoData     -> dettaglio promo code (partner name)
6. PromoCode activePromos  -> lista promo code attivi per partner
7-9. Azure AppConfig       -> feature flags (XI, XSACE)
```

I dati aziendali (fatturato, forma societaria) vengono recuperati ai passi 1-2 tramite Cerved. **Non c'e' accesso diretto a database** in questa API: tutti i dati provengono da chiamate HTTP esterne.

---

## Parametro `isSmallCorporateEnabled`

### Definizione
- **Controller** (`ProductController.kt:40`): `@RequestParam(required = false) isSmallCorporateEnabled: Boolean = false`
- E' un query parameter opzionale che di default vale `false`

### Flusso nel codice

1. Il client invia `GET /v1/products/{piva}?isSmallCorporateEnabled=true` (o `false`/omesso)
2. Il controller lo passa a `ProductService.products()` (linea 50)
3. Dopo aver recuperato i promo code attivi (passo 6), viene applicato il filtro `filterBySmallCorporate()` (linea 101)

### Logica di filtraggio (`ProductService.kt:189-203`)

```kotlin
private fun filterBySmallCorporate(
    promoCodes: List<PromoCode>,
    partner: String?,
    isSmallCorporateEnabled: Boolean,
): List<PromoCode> {
    return if (partner != null) {
        promoCodes.filter { promoCode ->
            promoCode.isSmallCorporateEnabled == isSmallCorporateEnabled ||
                (promoCode.isSmallCorporateEnabled == null && isSmallCorporateEnabled == false)
        }
    } else {
        promoCodes  // nessun filtro se partner non specificato
    }
}
```

**Comportamento**:
- **Se `partner` e' `null`**: nessun filtro applicato, tutti i promo code vengono restituiti
- **Se `partner` e' presente e `isSmallCorporateEnabled = false`** (default): restituisce solo i promo code con `isSmallCorporateEnabled == false` oppure `null` (tratta null come false)
- **Se `partner` e' presente e `isSmallCorporateEnabled = true`**: restituisce solo i promo code con `isSmallCorporateEnabled == true`

In sintesi: il parametro serve a **filtrare i promo code in base alla tipologia di cliente (small corporate vs non-small corporate)**, ma solo quando si specifica un partner. I promo code nel servizio PromoCode hanno un flag `isSmallCorporateEnabled` che indica se sono pensati per clienti "small corporate". Il parametro della query serve a selezionare il set corretto di promo code per il tipo di cliente.

### Uso in altri contesti del progetto

Oltre al flusso products, `isSmallCorporateEnabled` appare in:
- **`PartnerService.retrieveIsSmallCorporate()`**: recupera il flag dalla storia BPM del VAT number
- **`VatNumberInfoService`**: include il flag nella risposta `VatNumberInfoResponse`
- **`BeneficialOwnerServiceV2`**: gestisce il caso `small_corporate_misalignment` come motivo di rigetto onboarding (label `"small_corporate_misalignment"` da Actico -> trigger strategia di cancellazione `ImproperSmallCorporateDeleteStrategy`)
