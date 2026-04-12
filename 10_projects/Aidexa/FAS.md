# Financial Application Service (FAS)

## Cos'è

Il FAS è un microservizio di **aggregazione e persistenza** per le domande di finanziamento. Non orchestra flussi di business — è il **registro centralizzato** delle domande, consultato dal backoffice e dai BFF per avere una vista unificata del processo.

---

## Cosa fa

### 1. Aggrega dati
Interroga più sistemi e compone una vista unica di ogni domanda:
- **Camunda** → storico processi, variabili BPM, task attivi
- **Centrico** → status del flusso (`getFlowList`), conti bancari (`searchRegistry` + `accountBySubject`), contratti digitali
- **DB interno** → credit line, counter-offer, KYC, network activity

### 2. Persiste dati locali
Mantiene record propri che non vivono altrove:
- `LendingApplicationEntity` → record base per ogni onboarding
- Sessioni KYC Namirial (digital identification)
- Credit line e counter-offer
- Network activity per fraud detection

### 3. Espone API al backoffice
- `GET /lending`, `GET /applications/{processType}` → lista domande
- `GET /lending/{id}`, `GET /applications/{processType}/{id}` → dettaglio domanda
- `PATCH /lending/{id}` → aggiornamento campi
- `POST /applications/{onboardingId}/digital-identification/session` → avvio sessione KYC
- `POST /applications/{onboardingId}/digital-identification/callback/session/{type}/namirial` → callback Namirial

---

## Servizi chiamati in uscita

| Servizio | Scopo |
|---|---|
| **Centrico** | Status onboarding, conti bancari (Registry), lista flussi |
| **Camunda** | Storico processi, variabili, task |
| **Namirial (KYC)** | Avvio e recupero sessioni di identificazione digitale |
| **Azure Service Bus** | Invio eventi Avro (es. `ONBOARDING_STARTED`) |

---

## Cosa NON fa

- Non gestisce (ancora) una closing phase propria
- Non è un orchestratore di flussi BPM
- Non chiama direttamente il BFF contract o il BFF KYC
