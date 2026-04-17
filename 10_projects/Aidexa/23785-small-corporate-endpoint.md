# 23785 — Small Corporate Check via CRIF (company-data-service)

## Obiettivo

Aggiungere un endpoint al `company-data-service` che, dato un VAT number, determini se l'azienda è classificabile come **Small Corporate** controllando il fatturato (turnover) restituito da CRIF.

---

## Endpoint esposto

```
GET /company-data-service/v1/details/{vatNumber}/small-corporate
```

| Risposta | Significato |
|---|---|
| `200 true` | Fatturato > 20.000.000 → è Small Corporate |
| `200 false` | Fatturato ≤ 20.000.000 → non è Small Corporate |
| `204 No Content` | CRIF non ha dati di fatturato per questa azienda (o nessuna azienda attiva trovata) |
| `500` | Errore nella chiamata a CRIF (`COMPANY_DATA_30005`) |

---

## Flusso — pseudo-code

### 1. `CompanyDetailsController` — layer HTTP
```
riceve GET /v1/details/{vatNumber}/small-corporate
chiama companyDetailsPort.getIsSmallCorporate(vatNumber)

se Left(errore)  → HTTP 500 con body dell'errore
se Right(null)   → HTTP 204 No Content
se Right(bool)   → HTTP 200 con true/false nel body
```

### 2. `CompanyDetailsPort` — interfaccia inbound (contratto del dominio)
```
definisce:
  getIsSmallCorporate(vatNumber) → Either<CompanyDataError, Boolean?>
```

### 3. `DefaultCompanyDetailsService` — logica di dominio
```
chiama crifProvider.getTurnover(vatNumber)

se Left(errore)       → propaga l'errore
se Right(null)        → propaga Right(null)   [nessun dato fatturato]
se Right(turnover)    → Right(turnover > 20_000_000.0)
                        true  = small corporate
                        false = non small corporate
```

### 4. `CrifCompanyDataProviderAdapter` — adapter verso CRIF
```
crea CrifCompanyDataRequest.createCompanyDetailsRequest(vatNumber)
chiama CrifMargoClient.getCompanyData(request, CrifResponseForDetails)

se errore HTTP/rete  → Left(CompanyDataError("COMPANY_DATA_30005"))
se Right(response)   → cerca prima azienda attiva nella lista
                       se trovata  → estrae ecofin.turnover (Double?)
                       se non trovata → Right(null)
```

### 5. `CrifCompany.Ecofin` — model della risposta CRIF
```
aggiunto campo:
  turnover: Double? = null   ← nuovo campo mappato dal JSON di CRIF Margo
```

### 6. `CrifMargoClient` — client HTTP verso CRIF Margo
```
POST /v1/prospecting/search con il vatNumber
deserializza la risposta in CrifResponseForDetails
qualsiasi eccezione → Left(CompanyDataError("COMPANY_DATA_30005"))
```

---

## Catena completa

```
HTTP GET /v1/details/{vatNumber}/small-corporate
  → CompanyDetailsController
    → CompanyDetailsPort (contratto)
      → DefaultCompanyDetailsService (regola: soglia 20M)
        → CrifCompanyDataProviderAdapter (cerca azienda attiva)
          → CrifMargoClient (POST /v1/prospecting/search)
            → CrifCompany.Ecofin.turnover (campo JSON)
```

---

## File modificati

| File | Modifica |
|---|---|
| `adapter/api/CompanyDetailsController.kt` | Nuovo endpoint `getIsSmallCorporate` |
| `domain/ports/inbound/CompanyDetailsPort.kt` | Aggiunta firma `getIsSmallCorporate` all'interfaccia |
| `domain/service/DefaultCompanyDetailsService.kt` | Implementazione della logica con soglia 20M |
| `adapter/provider/CrifCompanyDataProviderAdapter.kt` | Nuovo metodo `getTurnover` |
| `adapter/thirdparty/crif/margo/model/response/CrifCompany.kt` | Aggiunto campo `turnover: Double?` in `Ecofin` |

---

## Note tecniche

- L'endpoint usa **CRIF Margo** (non Cerved) — il provider è diverso da `GET /v1/details/{vatNumber}` che usa Cerved
- La soglia `20_000_000.0` è una costante in `DefaultCompanyDetailsService`
- Il caso `null` (204) copre sia "azienda non trovata in CRIF" che "azienda trovata ma senza dati ecofin/turnover"
- Unico error code possibile: `COMPANY_DATA_30005`

---

## Perché `isActiveCompany()` nel filtro CRIF

```kotlin
fun CrifCompany.isActiveCompany(): Boolean = this.companyStatus.activityStatus.code == "A"
```

CRIF restituisce una **lista di aziende** per una stessa partita IVA — non un singolo record. La lista può contenere più voci con stati diversi (attiva, cessata, in liquidazione, ecc.). Il codice `"A"` identifica un'azienda **Attiva**.

Il `firstOrNull { isActiveCompany() }` serve a prendere solo il primo record attivo, scartando la storia pregressa della società. Ha senso calcolare il fatturato solo su un'azienda ancora operativa.

Questo pattern è usato in tutti i metodi di `CrifCompanyDataProviderAdapter` che lavorano su una singola azienda:

| Metodo | Endpoint che lo usa |
|---|---|
| `getCompaniesForDetails()` | `GET /v1/details/{vatNumber}` |
| `getCompaniesForInfo()` | gruppi economici |
| `getTurnover()` | `GET /v1/details/{vatNumber}/small-corporate` |
