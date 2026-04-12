# Small Corporate

## Cos'è lo "Small Corporate"
Un'azienda è classificata **Small Corporate** se supera una soglia di fatturato (circa 20M€). Questa classificazione permette al back office di proporre **condizioni di finanziamento migliorative** rispetto allo standard.

---

## Due modalità di ingresso

| Tipo | Come si attiva | Meccanismo |
|---|---|---|
| **Proattivo** | Il cliente ha un **promo code** legato a una Small Corporate | Flag nel promo code riconosce la classificazione all'ingresso |
| **Proattivo** | Il cliente arriva da una **pagina partner** | Query parameters contengono `partnerID` + `isSmallCorporate=true` |
| **Reattivo** | Il cliente arriva senza nessuno dei segnali precedenti | Nessun parametro in ingresso; la classificazione avviene a posteriori tramite Actico |

In tutti i casi il **flusso di onboarding è identico** — la distinzione serve solo per tracciatura. Il back office poi agisce di conseguenza.

---

## Flusso di selezione prodotti (pagina contatti)

Questo è un passaggio fondamentale da capire.

Quando il cliente atterra sulla **pagina dei contatti** e inserisce un promo code (o arriva con `partnerID` nei query params), **non viene associato direttamente a quel promo code**. Il promo code (o il partnerID) serve esclusivamente a identificare il **partner** di riferimento — un partner può avere N promo code.

Il flusso è il seguente:

1. Il cliente inserisce il promo code **oppure** arriva con `partnerID` nei query params
2. Il cliente inserisce l'OTP e va avanti
3. Viene eseguita una chiamata **Lending** che restituisce l'elenco dei prodotti disponibili

> [!warning] Da verificare
> L'endpoint ipotizzato per la chiamata Lending è il seguente:
> `GET https://dev.aidexa.it/dynamic-onboarding-bff-business-data/v1/products/55378992949?accountType=LENDING&revenueGraterThan100K=true&isSmallCorporateEnabled=false`
> Verificare che sia effettivamente questo l'endpoint utilizzato e che i query parameter corrispondano a quelli reali del flusso.

La chiamata Lending valuta una serie di parametri, tra cui:
- `partnerID`
- fatturato e tipo di fatturato
- se l'azienda è Small Corporate

> **Caso critico:** se viene passato un `partnerID` con `isSmallCorporate=true` ma non viene trovato nessun promo code corrispondente a quella combinazione, Lending non restituisce alcun prodotto → il cliente finisce sulla **pagina viola** (nessun prodotto disponibile).

### Analisi dell'endpoint `GET v1/products/{vatNumber}`

```
GET v1/products/{vatNumber}?accountType=LENDING&revenueGraterThan100K=true&isSmallCorporateEnabled=false
→ ProductController → ProductService.products()
```

Il servizio esegue in sequenza le seguenti operazioni:

1. **Recupera i dati aziendali da Cerved** — forma giuridica, fatturato, profilo economico
2. **Controlla la storia creditizia via FAS client** — verifica se ci sono richieste rifiutate o recenti
3. **Recupera i promo code attivi** — filtrati per partner e `accountType` (`LENDING` in questo caso)
4. **Applica i filtri sui promo code:**
   - `isSmallCorporateEnabled=false` → esclude i promo code riservati alle Small Corporate
   - Restringe i promo code CC in base alla storia creditizia
5. **Seleziona i prodotti eleggibili** — in base alla forma giuridica e all'`accountType`
6. **Filtra per fatturato** — `revenueGraterThan100K=true` esclude i prodotti incompatibili con aziende ad alto fatturato
7. **Applica feature flag** — per la visibilità di prodotti specifici (`XSACE`, `XI`)
8. **Restituisce un `ProductsResponse`** — con i prodotti disponibili e le relative garanzie

> In sintesi: dato un numero di partita IVA e alcuni flag di contesto, restituisce la lista dei prodotti finanziari per cui l'azienda è eleggibile, tenendo conto di profilo aziendale, storia creditizia, promo code e feature flag.

#### Note sui Query Parameter

| Parametro | Tipo | Default | Note |
|---|---|---|---|
| `accountType` | `AccountType` enum (`CC` \| `LENDING`) | `LENDING` | Optional — se omesso viene usato `LENDING` |
| `isSmallCorporateEnabled` | boolean | `false` | Optional — se `false` esclude i promo code Small Corporate. Rilevante **solo se è presente un partner**; senza partner non esistono promo code Small Corporate da mostrare |

---

## Problema tecnico in sospeso
Viorel (in ferie) ha lasciato un commento su Portal indicando che la variabile usata per determinare l'eleggibilità restituisce il valore `ND 99` — da approfondire.

**Azione:** Roberto deve contattare **Simone o Guglielmo** per capire come **Actico** definisce l'eleggibilità Small Corporate.

---

## Servizio da costruire
Un endpoint leggero chiamabile dal front-end:

```
INPUT:  partita IVA (+ eventuale partnerID / isSmallCorporate)
OUTPUT: is_small_corporate_eligible → true / false
```

- Se arriva con **partnerID + isSmallCorporate=true** → proattivo, classificazione diretta
- Senza parametri → il controllo lo fa comunque Actico, ma va **anticipato** nel passaggio "dati azienda"
- Il controllo viene quindi eseguito **due volte**: una da loro, una da Actico (doppio check intenzionale)

### Comportamento di Actico nel flusso proattivo
Quando Actico esegue il proprio controllo sulla revenue dell'azienda per determinare l'eleggibilità Small Corporate, il comportamento dipende dalla modalità:

- **Flusso proattivo** — se il controllo dà esito **negativo** (revenue insufficiente), il flusso viene **interrotto con un errore**. L'azienda non può proseguire l'onboarding come Small Corporate.
- **Flusso reattivo** — il controllo fallito non blocca il flusso; la classificazione semplicemente non viene applicata.

---

## Architettura
- Ivano propone di posizionare il servizio nel **Business Data BFF**
- Serve **conferma di Francesco** (che ha la visione architetturale completa)

---

## Prossimi passi

1. Roberto verifica il commento di Viorel su Portal
2. Roberto parla con Simone/Guglielmo per capire la logica di eleggibilità di Actico
3. Roberto chiede a Francesco dove posizionare il servizio
4. Implementazione del servizio entro lunedì
5. La parte front-end viene affrontata in un secondo momento con Ivano

---

## Actico e Small Corporate — Analisi del codice

### Progetti che chiamano Actico

Actico viene chiamato da tre progetti:

| Progetto | Client | Chiamate principali |
|---|---|---|
| `dynamic-onboarding-business-data-bff` | `ActicoWebClient` (Spring WebClient) | `checkCompany`, `createApplication`, `cancelApplication`, `computeBusinessOutcome`, `prescoreRiskAssessment` |
| `dynamic-onboarding-bff-product` | `ActicoWebClient` (Spring WebClient) | `checkCompanyRisk`, `pricingSustainability`, `rateBasedPricingAndSustainability`, `computeBusinessOutcome`, `sustainableDurationAndPricing` |
| `credit-decision` | `ActicoClient` con Strategy pattern | `createApplication`, `computeQuotes`, `configureQuote`, `quoteDecision` |

Non sono stati trovati riferimenti a Small Corporate in `credit-decision`, `financial-application-service` e `company-data-service`.

---

### La chiamata Actico rilevante: `IFC002_Check_Company`

La chiamata direttamente coinvolta nella logica Small Corporate è **`IFC002_Check_Company`**, eseguita da `dynamic-onboarding-business-data-bff`.

#### URL completa per ambiente

La base URL è configurata in `application.yaml` (per ambiente) e viene combinata con il path fisso della chiamata. La versione (`/5`, `/6`, etc.) viene risolta dinamicamente in `resolveUrlVersionToCall()` in base ai feature flag del processo BPM.

| Ambiente | URL completa da cercare nei log |
|---|---|
| **DEV** | `https://api-dev.aidexa.it/external/actico/engine/v1/executions/aidexa.ifc-facade/{version}/aix-ifc-incoming-facade/AIX_IFC_Incoming_Facade/IFC002_Check_Company/START_IFC002_Check_Company` |
| **PRE** | `https://api-gw-pre.aidexa.it/external/actico/engine/v1/executions/aidexa.ifc-facade/{version}/aix-ifc-incoming-facade/AIX_IFC_Incoming_Facade/IFC002_Check_Company/START_IFC002_Check_Company` |

> [!tip] Ricerca nei log
> La stringa più efficace da cercare nei log è il path distintivo, indipendente dall'ambiente e dalla versione:
> `IFC002_Check_Company`

La versione `{version}` viene risolta così:

| Condizione | Versione |
|---|---|
| `isTanDiscountFeiEnabled = true` | `8` |
| `isTotalOverrideEnabled = true` | `7` |
| `isPreamortizationEnabled = true` | `6` |
| Default | `5` |

#### Struttura della response

La response ha sempre questa forma:

```json
{
  "trace": {
    "version": "1.0",
    "modelPath": "IFC002_Check_Company/START_IFC002_Check_Company",
    "requestId": "<uuid>",
    "modelName": "AIX_IFC_Incoming"
  },
  "output": {
    "response": {
      "messages": [
        { "message": "Your request has been processed", "message_type": "INFO" }
      ],
      "status": "In_Progress | Declined",
      "response_type": "Success",
      "payload": {
        "metascore": 125.0,
        "outcome": "OK | KO",
        "ref_timestamp": "2022-08-05T14:13:26.205Z",
        "outcome_reason": [],
        "output_type": ""
      },
      "document_id": null
    }
  }
}
```

I campi rilevanti per capire l'esito sono nel `payload`:

| Campo | Valori possibili | Significato |
|---|---|---|
| `outcome` | `OK` / `KO` | Esito complessivo della valutazione |
| `status` | `In_Progress` / `Declined` | Stato del processo in Actico |
| `outcome_reason` | array di stringhe | Motivi dell'esito — può essere vuoto se tutto OK |

#### Valori noti di `outcome_reason`

Dai test presenti nel progetto, i valori conosciuti sono:

| Valore | Significato | Effetto nel BFF |
|---|---|---|
| _(array vuoto)_ | Nessuna anomalia rilevata | Flusso prosegue normalmente |
| `"small_corporate_misalignment"` | L'azienda è marcata come Small Corporate ma Actico non la riconosce come eleggibile | **Onboarding cancellato** con requestor `RISK` → `OnboardingRejectedError` |
| `"ultimo_bilancio"` | Bilancio mancante o non recente | Onboarding cancellato (stesso meccanismo, condizionato al feature flag `ONBOARDING_BANKACCOUNT_IS_BALANCE_SHEET_CHECK_ENABLED`) |
| `"tenure"` | Anzianità aziendale insufficiente | Onboarding cancellato con requestor `RISK` |

> [!note]
> Il campo `outcome_reason` è un **array** — possono essere presenti più motivi contemporaneamente. Il BFF prende il primo motivo trovato che corrisponde alla mappa interna e cancella l'onboarding.

---

### Gestione del `small_corporate_misalignment` — `BeneficialOwnerServiceV2.kt`

Il file che gestisce la risposta di `IFC002_Check_Company` è:

```
dynamic-onboarding-business-data-bff
└── src/main/kotlin/it/aidexa/businessdata/beneficialowner/BeneficialOwnerServiceV2.kt
```

**Come funziona (linee 527–654):**

1. Dopo la chiamata `checkCompany()`, il servizio ispeziona `outcomeReason` nella risposta
2. Incrocia i valori trovati con una mappa interna `deletionReasonAndRequestor`
3. Se trova `small_corporate_misalignment`, lo mappa a `OnboardingDeletionRequestor.RISK`
4. Invoca `onboardingService.deleteOnboarding(bpmId, RISK)`
5. Restituisce `OnboardingRejectedError` — il flusso si blocca

```kotlin
// BeneficialOwnerServiceV2.kt — line 648
private const val SMALL_CORPORATE_MISALIGNMENT_LABEL = "small_corporate_misalignment"

// Mappa: reason → requestor di cancellazione
deletionReasonAndRequestor = mapOf(
    SMALL_CORPORATE_MISALIGNMENT_LABEL to OnboardingDeletionRequestor.RISK,
    ...
)

// Se trovato in outcomeReason:
onboardingService.deleteOnboarding(companyRiskContext.bpmId, OnboardingDeletionRequestor.RISK)
→ OnboardingRejectedError(setOf("small_corporate_misalignment"))
```

Questo è il meccanismo concreto del **blocco del flusso proattivo**: non è Actico a bloccare direttamente, ma è il BFF che legge l'esito e cancella l'onboarding con requestor `RISK`.

---

### Il flag `isSmallCorporateEnabled` nel ciclo di vita del prodotto

Il flag percorre tutto lo stack dal frontend al backend:

```
URL query param (contacts/index.tsx)
  → isSmallCorporate (router.query)
  → isSmallCorporateEnabled (useState)
  → GET /v1/products/{vatNumber}?isSmallCorporateEnabled=true|false
  → ProductController.kt (@RequestParam, default false)
  → ProductService.products(isSmallCorporateEnabled)
  → filterBySmallCorporate(promoCodes, partner, isSmallCorporateEnabled)
```

**Logica del filtro** (`ProductService.kt` — linee 182–196):

```kotlin
private fun filterBySmallCorporate(
    promoCodes: List<PromoCode>,
    partner: String?,
    isSmallCorporateEnabled: Boolean
): List<PromoCode> {
    return if (partner != null) {
        promoCodes.filter { promoCode ->
            promoCode.isSmallCorporateEnabled == isSmallCorporateEnabled ||
                (promoCode.isSmallCorporateEnabled == null && isSmallCorporateEnabled == false)
        }
    } else {
        promoCodes  // senza partner, il filtro non si applica
    }
}
```

Punti chiave:
- Il filtro ha effetto **solo se è presente un partner** — senza partner non esistono promo code Small Corporate
- `isSmallCorporateEnabled == null` su un promo code viene trattato come `false`
- Il flag viaggia anche nel modello `PartnerCodeResponse` (`isSmallCorporateEnabled: Boolean?`) restituito da `PartnerService`
