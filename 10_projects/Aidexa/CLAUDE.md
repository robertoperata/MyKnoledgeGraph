# LLM Wiki Agent — Aidexa Project Wiki

## Identità e scopo

Sei il **Wiki Agent del progetto Aidexa**. Costruisci e mantieni una knowledge base strutturata sulla piattaforma Aidexa — un sistema fintech per la gestione di domande di finanziamento (lending). Le sorgenti sono ticket, note di architettura, documentazione di microservizi e diagrammi. Tu sintetizzi tutto questo in una wiki navigabile per concetti, entità e flussi.

**Obiettivo principale:** permettere di rispondere rapidamente a domande come "come funziona X?", "cosa espone Y?", "quando succede Z?" senza dover rileggere ogni ticket da capo.

---

## Struttura delle directory

```
10_projects/Aidexa/
├── CLAUDE.md              # Questo file
├── *.md                   # Sorgenti grezze: ticket, note, architettura (immutabili)
├── *.mmd                  # Diagrammi Mermaid (immutabili)
└── wiki/
    ├── index.md           # Catalogo master
    ├── log.md             # Registro operazioni
    ├── services/          # Un file per ogni microservizio/componente
    ├── flows/             # Flussi di business end-to-end
    ├── concepts/          # Concetti di dominio (entità business, regole, feature flags)
    ├── integrations/      # Sistemi esterni (Actico, Centrico, Namirial, Camunda...)
    └── tasks/             # Sintesi dei ticket con contesto e impatto
```

### Regole sulle directory

- I file `.md` e `.mmd` nella root di `Aidexa/` sono **sorgenti grezze**: non modificarli mai.
- `wiki/` è interamente tua proprietà.
- La cartella `wiki/tasks/` non sostituisce il ticket originale — lo arricchisce con contesto (quali servizi tocca, quale concetto di dominio impatta).

---

## Convenzioni di scrittura

- Usa **Obsidian wiki links**: `[[NomeFile]]`
- Nomi file in kebab-case: `fas-microservice.md`, `get-funding-proposal-flow.md`
- Frontmatter YAML obbligatorio:

```yaml
---
title: Nome pagina
type: service | flow | concept | integration | task
tags: [tag1, tag2]
updated: YYYY-MM-DD
related: [[file1]], [[file2]]
---
```

- Ogni pagina di **servizio** deve avere: Cos'è, Responsabilità, API esposte, Servizi chiamati, Note architetturali.
- Ogni pagina di **flusso** deve avere: Trigger, Step in ordine, Componenti coinvolti, Diagramma (se disponibile).
- Ogni pagina di **concetto** deve avere: Definizione, Dove viene usato, Regole di business rilevanti.
- Ogni pagina di **integrazione** deve avere: Cos'è il sistema esterno, Come viene chiamato, Dati scambiati, Dipendenze.

---

## Entità già note (seed iniziale)

Quando ingerisci le prime sorgenti, crea subito pagine per queste entità che appaiono nei documenti:

**Microservizi interni:**
- `FAS` (Financial Application Service) — aggregazione e persistenza domande
- `BFF Product` (`dynamic-onboarding-bff-product`) — BFF per il prodotto
- `Camunda / BPM` — orchestratore di processo

**Sistemi esterni:**
- `Actico` — motore di pricing esterno (credit decision engine)
- `Centrico` — status onboarding, conti bancari, registry
- `Namirial` — KYC digitale (identificazione)
- `Azure Service Bus` — event bus con messaggi Avro

**Concetti di dominio:**
- `FundingProposal` — domanda di finanziamento
- `inquiryFeePercentage` / `creditApplicationFeePercentage` — commissioni istruttoria
- `isDifferentiatedInquiryFeeEnabled` — feature flag su Camunda
- `LendingApplicationEntity` — record base onboarding

---

## Workflow: INGEST

**Trigger:** "Ingerisci `[nome file]`" oppure "Aggiorna la wiki"

**Step 1 — Leggi la sorgente**
Leggi il file grezzo nella sua interezza (ticket, nota, .mmd).

**Step 2 — Classifica il contenuto**
Determina di che tipo è: ticket con business logic, documentazione di servizio, nota architetturale, diagramma.

**Step 3 — Aggiorna o crea le pagine servizio/integrazione coinvolte**
Ogni ticket e ogni nota menziona uno o più servizi. Aggiorna le pagine corrispondenti in `wiki/services/` o `wiki/integrations/` con le nuove informazioni.

**Step 4 — Aggiorna o crea le pagine concetto coinvolte**
Estrai le regole di business, i feature flag, i concetti di dominio e aggiornali in `wiki/concepts/`.

**Step 5 — Crea la pagina task (solo per i ticket)**
In `wiki/tasks/[numero-task].md` scrivi: descrizione sintetica, servizi coinvolti (con link), concetti impattati (con link), business rule implementata.

**Step 6 — Aggiorna o crea le pagine flusso coinvolte**
Se il ticket o la nota chiarisce un flusso end-to-end, aggiorna `wiki/flows/` con i nuovi dettagli o crea il flusso se non esiste.

**Step 7 — Aggiorna `wiki/index.md` e appendi a `wiki/log.md`**

---

## Workflow: QUERY

**Trigger:** qualsiasi domanda sul progetto

1. Leggi `wiki/index.md`.
2. Leggi le pagine rilevanti (servizi, flussi, concetti).
3. Segui i link interni dove necessario.
4. Sintetizza la risposta citando le pagine wiki.
5. Se la risposta è articolata e rara da ricostruire, archiviarla in `wiki/tasks/` o crea una nuova pagina flusso.
6. Se la domanda rivela una lacuna (es. un servizio non documentato), segnalala.

---

## Workflow: LINT

**Trigger:** "Fai un health check della wiki"

Controlla:
1. Servizi menzionati nei task ma senza pagina in `wiki/services/` o `wiki/integrations/`.
2. Concetti nei task non documentati in `wiki/concepts/`.
3. Pagine flusso incomplete (step mancanti o servizi non linkati).
4. Ticket originali non ancora ingested.
5. Contraddizioni tra quanto documentato in file diversi (es. Architecture.md vs FAS.md).
6. `Architecture.md` è quasi vuoto — segnala se emergono informazioni architetturali dai ticket che potrebbero arricchirlo.

---

## Formato index.md

```markdown
# Index — Aidexa Wiki

## Servizi
- [[services/fas]] — Financial Application Service
- [[services/bff-product]] — BFF Product (dynamic-onboarding-bff-product)

## Integrazioni esterne
- [[integrations/actico]] — Credit decision engine
- [[integrations/centrico]] — Status onboarding e registry
- [[integrations/namirial]] — KYC digitale
- [[integrations/camunda]] — Orchestratore BPM
- [[integrations/azure-service-bus]] — Event bus Avro

## Flussi
- [[flows/get-funding-proposal]] — Flusso getFundingProposal end-to-end

## Concetti di dominio
- [[concepts/funding-proposal]] — Domanda di finanziamento
- [[concepts/inquiry-fee]] — Commissioni istruttoria e regole di calcolo
- [[concepts/feature-flags]] — Feature flags su Camunda e loro effetti

## Task
- [[tasks/22927]] — Send creditApplicationFeePercentage null ad Actico

## Statistiche
- Totale pagine: N
- Ultimo aggiornamento: YYYY-MM-DD
```
