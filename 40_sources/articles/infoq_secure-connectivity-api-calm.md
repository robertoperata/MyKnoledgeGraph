# Platforms for Secure API Connectivity with Architecture as Code

**Fonte:** [InfoQ - Jim Gough](https://www.infoq.com/presentations/secure-connectivity-api/)  
**Data lettura:** 2026-04-11  
**Tipo:** Presentation Transcript  
**Tag:** #api #security #architecture-as-code #CALM #kubernetes #developer-experience #threat-modeling

---

## Contesto

Jim Gough presenta la sua esperienza a **Morgan Stanley**, dove ha evoluto il ruolo da sviluppatore solitario ad architetto accidentale del programma API dell'organizzazione. La presentazione si articola su tre temi principali:

1. Gestire la complessità della connettività API
2. Implementare approcci di secure design
3. Introdurre i pattern architetturali come framework di soluzione

---

## La Sfida della Complessità

Sviluppare una semplice API richiede poco sforzo. Ma aggiungere autenticazione, autorizzazione, gestione degli accessi, deployment automatizzati, gateway e requisiti non funzionali moltiplica rapidamente la complessità.

### Requisiti non funzionali critici identificati:
- Sicurezza
- Performance
- Disponibilità (Availability)
- Osservabilità (Observability)
- Audit logging
- Compliance
- Gestione dei costi

Questi elementi non sono più "nice-to-have": sono necessità operative critiche.

---

## I Due Gap Fondamentali

### 1. Gap Sviluppatore–Infrastruttura
Se si sposta troppo "a sinistra" la responsabilità — dicendo ai developer "ecco gli strumenti, fai da te" — senza adeguato supporto, i developer si scontrano con la complessità degli strati di astrazione.

### 2. Gap Sviluppatore–Sicurezza
Esiste un disallineamento di linguaggio e conoscenza tra i professionisti della sicurezza e i developer. La sicurezza tende a diventare un **gate bloccante** piuttosto che un principio guida, apparendo solo al momento del rilascio in produzione.

> **Problema centrale:** I developer fanno fatica a "fare la cosa giusta" perché il percorso sicuro non è il percorso più semplice.

---

## Secure Design: Threat Modeling con STRIDE

Gough presenta un esempio pratico con un'architettura di sito web per conferenze (attendee services), usando il framework **STRIDE** per il threat modeling:

| Lettera | Minaccia |
|---|---|
| S | Spoofing |
| T | Tampering |
| R | Repudiation |
| I | Information Disclosure |
| D | Denial of Service |
| E | Elevation of Privilege |

### Demo pratico — Il problema reale:
In un deployment insicuro, un container compromesso può accedere direttamente ai servizi backend sfruttando le assunzioni di networking di Kubernetes, **bypassando i controlli di sicurezza previsti**. I developer possono creare sistemi vulnerabili pur avendo buone intenzioni, semplicemente perché il path di default non è sicuro.

---

## La Soluzione: CALM — Common Architecture Language Model

CALM è un framework sviluppato attraverso **FINOS** (Fintech Open Source Foundation) in collaborazione con istituzioni finanziarie. Fornisce un linguaggio comune per descrivere architetture sicure come codice.

### Componenti principali:

| Componente | Descrizione |
|---|---|
| **Core Model** | Specifica basata su JSON Schema per descrivere architetture |
| **CLI Tools** | Interfaccia a riga di comando per i developer |
| **Patterns** | Template riutilizzabili con sicurezza integrata |
| **Controls** | Requisiti non funzionali modellati con configurazioni specifiche |

### Concetti chiave:
- **Nodes**: componenti (servizi, database, cluster)
- **Relationships**: connessioni tra componenti
- **Controls**: catturano requisiti di sicurezza, performance e compliance con opzioni di configurazione

> *"What CALM does, and architecture as code, is it really puts an option between the level 3 and 4 diagram of C4 architecture, filling the gap between high-level diagrams and actual infrastructure code."*  
> — Jim Gough

---

## Workflow Pratico con CALM

```
Pattern Creation → Generation → Validation → Templating → Deployment
```

1. **Pattern Creation**: Gli architetti definiscono pattern riutilizzabili con sicurezza integrata
2. **`calm-generate`**: I developer generano istanze architetturali dai pattern
3. **`calm-validate`**: Verifica completezza e correttezza dell'architettura
4. **Template Bundles**: Generano configurazioni di deployment (es. manifest Kubernetes)
5. **Deployment**: Infrastructure as code applicata in produzione

Le **Pattern Options** permettono personalizzazione tramite selezione, senza dover creare nuovi pattern da zero per ogni variante.

---

## Risultati Dimostrati

Con questo approccio i developer possono:
- Fare deploy di sistemi sicuri con minima expertise di sicurezza
- Compilare semplicemente nomi di immagini e fare selezioni guidate
- Generare deployment Kubernetes completi e compliant con **network policies** per la micro-segmentazione

**Obiettivo dichiarato di Gough:**
> Ridurre i cicli di deployment da **6 mesi a 2 ore** attraverso review di sicurezza automatizzate e percorsi di implementazione chiari.

---

## Punti Chiave e Takeaway

- **"Paved paths"** chiari verso la produzione con sicurezza integrata by-design
- **Fast feedback loops**: eliminare i ritardi da approval manuali
- **Self-service deployments** con guardrail automatici
- **Processi di approvazione integrati** basati su standard di settore
- **Documentazione strutturata** seguendo il framework **Diátaxis** (concetti, riferimenti, how-to)

> *"Make it really easy for developers to do the right thing"* — la filosofia guida dell'intero approccio.

---

## Analisi e Riflessioni

### Punti di forza del framework CALM:
- Risolve un problema reale e diffuso: il gap tra intenzioni di sicurezza e implementazione
- Approccio "paved path" riduce l'attrito per i developer
- Integrazione con l'ecosistema esistente (Kubernetes, IaC) evita di reinventare la ruota
- Sviluppato in contesto finanziario altamente regolamentato → alta rilevanza enterprise

### Domande aperte / limiti:
- Quanto è matura l'adozione di CALM fuori dall'ambiente Morgan Stanley / FINOS?
- Come si gestisce la drift tra il modello CALM e l'infrastruttura reale nel tempo?
- Qual è il costo di onboarding dei team per imparare i pattern CALM?

### Connessioni con altri concetti:
- Si collega al concetto di **Platform Engineering** e Internal Developer Platforms (IDP)
- Complementare a strumenti come **Backstage** (Spotify) per il developer portal
- Si inserisce nel movimento **Security as Code / Policy as Code** (es. OPA, Kyverno)
- Il gap sviluppatore-sicurezza è lo stesso identificato nel **DORA report** come friction point chiave
