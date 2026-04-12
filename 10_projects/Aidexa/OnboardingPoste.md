# Onboarding Poste — Analisi Tecnica

## BpmWebClientAdapter

`BpmWebClientAdapter` è uno Spring `@Service` dentro `dynamic-onboarding-bff-psd2` che funge da **client HTTP verso Camunda** (BPM engine). Non è un servizio separato — è una classe che chiama Camunda via REST.

### A cosa punta

Dipende dalla feature flag `features.kong.bpm` in `application.yaml`:

| Flag | URL |
|---|---|
| `true` (default in dev) | `https://api-gw-dev.aidexa.it/bpm-lending` → passa per **Kong** (API Gateway) |
| `false` | `http://engine-manager-svc.onboarding/...` → chiamata **diretta** al pod Kubernetes |

Aggiunge automaticamente il header `kongConsumerSecret` (da `ApiGatewayConfiguration`) a ogni chiamata verso Kong.

### Chiamate REST a Camunda

```
GET  /process-instance/{bpmId}/variables/{variableName}?deserializeValue=false
POST /process-instance/{bpmId}/variables
```

### Percorso completo di una chiamata

```
BanksInfoController
  → BpmWebClientAdapter
    → Kong (api-gw-dev.aidexa.it/bpm-lending)
      → Camunda (engine-manager-svc)
```

---

## BanksInfoController

**Endpoint:** `GET /dynamic-onboarding-bff-psd2/v1/banks-info`

Non richiede body né query params. Richiede header `Authorization: Bearer <JWT>`.

Il `bpmId` viene estratto dal campo **`sub`** del JWT tramite `AuthenticatorFilter`.

### Flusso interno

1. Estrae `bpmId` dal JWT (campo `sub`)
2. `bpmWebClient.getVatNumberOptional(bpmId)` → variabile Camunda `onboardingSupportData`
3. `bpmWebClient.getAccountSummary(bpmId)` → variabile Camunda `accountStepSelectBanks`
4. `bpmWebClient.getPartnerOnboardingRequest(bpmId)` → variabile Camunda `partnerOnboardingRequest` → estrae `company.preferedIBAN`
5. Filtra le banche per IBAN selezionati e ordina mettendo prima la banca con `preferedIBAN`

### Curl di esempio

```bash
curl -X GET \
  'https://<host>/dynamic-onboarding-bff-psd2/v1/banks-info' \
  -H 'Authorization: Bearer <JWT_TOKEN>'
```

Il JWT deve contenere nel payload:
```json
{
  "sub": "<bpmId>",
  "jti": "...",
  "ses": "...",
  "iss": "...",
  "aud": "...",
  "iat": "...",
  "nbf": "...",
  "exp": "..."
}
```

---

## Come viene salvato `preferedIBAN` su Camunda

Il dato viene scritto da **`dynamic-onboarding-business-data-bff`**, non da `bpm-x-instant`.

### Flusso

1. **POST** `dynamic-onboarding-bff-business-data/v1/lending/onboardings`
   - Body contiene `company.preferedIBAN` (validato con regex IBAN)

2. `OnboardingService.startOnboarding()` serializza l'intero `OnboardingRequest` come variabile Camunda:

```kotlin
bpmWebClientAdapter.addVariable(
    bpmId,
    onboardingRequest.toProcessVariable(
        TaskVariable.PARTNER_ONBOARDING_REQUEST,  // → "partnerOnboardingRequest"
        BpmConstants.JSON_TYPE,
    )
)
```

3. Chiamata REST a Camunda:
```
PUT /process-instance/{bpmId}/variables/partnerOnboardingRequest
```
Body: `OnboardingRequest` serializzato in JSON con Jackson.

### File coinvolti

| Ruolo | File |
|---|---|
| HTTP Endpoint | `dynamic-onboarding-business-data-bff/.../OnboardingController.kt` |
| Service Logic | `dynamic-onboarding-business-data-bff/.../OnboardingService.kt` |
| BPM Client | `dynamic-onboarding-business-data-bff/.../BpmWebClientAdapter.kt` |
| Task Variable Enum | `dynamic-onboarding-business-data-bff/.../TaskVariable.kt` |

---

## bpm-x-instant

### Cos'è

È il **Camunda BPM engine** embedded — il cervello che orchestra tutto il flusso di onboarding. Non espone controller REST tradizionali: è Camunda stesso che espone le API come Spring Boot starter.

- **Context path:** `/bpm/v1/finanziamento`
- **Database:** PostgreSQL (Azure) per lo stato dei processi
- **Stack:** Kotlin/Java, Spring Boot, Camunda 7.18

### Cosa fa

Gestisce i workflow di onboarding per prodotti finanziari:
- Prodotti lending: **XI, XW, XG, XC**
- Conto corrente: **CC (ContoAzienda)**
- Cross-onboarding: collegamento banche PSD2 per clienti lending esistenti

### Servizi che chiama

| Servizio | Scopo |
|---|---|
| **Centrico** | Master record onboarding — START, UPDATE, CLOSE |
| **Actico** | Decisione creditizia |
| **StepManager** | Metadata dei widget/componenti UI per step |
| **Business Data BFF** | Cancella record onboarding in caso di rifiuto |
| **STS** | Token JWT per chiamate service-to-service |
| **Azure Blob Storage** | Audit trail delle request verso Centrico |
| **Azure Service Bus** | Pubblica eventi `OnboardingStepChanged` (AVRO) |

### Flusso principale

```
1. START   → POST Centrico → ottiene GUID
2. STEPS   → l'utente compila i form → taskVariables su Camunda
3. UPDATE  → chiama StepManager → costruisce payload → POST Centrico
4. CLOSE   → PUT Centrico → polling contratti firmati
5. ACTICO  → decisione creditizia
6. EVENTI  → ad ogni step pubblica OnboardingStepChanged su Service Bus
```

### Variabili Camunda principali

| Nome variabile | Contenuto |
|---|---|
| `flowOnboardingCode` | Codice prodotto (XI, XW, XG, XC, CC) |
| `guid` | GUID Centrico (generato al START) |
| `businessKey` | Partita IVA (business key del processo) |
| `taskVariables` | Risposte dei form UI per lo step corrente |
| `OnboardingData` | Payload completo onboarding |
| `startConfigFlags` | Feature flags per logica condizionale |
| `centricoJobExecutionStatus` | Risultato operazione Centrico (OK/KO) |
| `closeExecution` | Risultato operazione di chiusura (OK/KO) |

### URL Centrico (dev)

```
POST /v4.1/process/flow/{product}/create          → START
POST /v4.1/process/flow/{product}/{guid}           → UPDATE
PUT  /v4.1/process/flow/{product}/{guid}/close     → CLOSE
```

---

## Relazione tra i servizi

```
dynamic-onboarding-bff-psd2          → legge variabili da Camunda (bpm-x-instant)
dynamic-onboarding-business-data-bff → scrive variabili su Camunda (bpm-x-instant)
bpm-x-instant                        → orchestratore, usa le variabili per chiamare Centrico/Actico
```

---

## Variabili Camunda coinvolte

| Nome variabile | Contenuto | Usata da |
|---|---|---|
| `onboardingSupportData` | VAT number e altri dati di supporto | `BanksInfoController` |
| `accountStepSelectBanks` | Lista account con flag `selected` | `BanksInfoController` |
| `partnerOnboardingRequest` | Intero `OnboardingRequest` incl. `company.preferedIBAN` | `BanksInfoController`, scritto da `business-data-bff` |
