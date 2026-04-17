---
tags:
  - architecture
  - api
  - developer-experience
  - agentic
  - testing
type: article
author: Leigh Griffin, Ray Carroll
source: https://www.infoq.com/articles/spec-driven-development/
date: 2026-01-12
---

# Spec Driven Development: When Architecture Becomes Executable

## Sunto

**Spec-Driven Development (SDD)** è un paradigma architetturale che inverte il tradizionale *source of truth* del software: invece di far emergere la verità dal codice, la verità è definita nelle **specifiche eseguibili**, e il codice ne diventa semplicemente la materializzazione. Gli autori inquadrano SDD come la quinta generazione di astrazione del software engineering, resa possibile dall'AI generativa che consente agli sviluppatori di esprimere *cosa* un sistema deve fare, delegando il *come* agli strumenti intelligenti.

Il modello SDD si articola in **cinque strati**:
1. **Specification Layer** — definizione autoritativa del comportamento del sistema
2. **Generation Layer** — trasformazione dell'intento dichiarativo in forma eseguibile
3. **Artifact Layer** — output concreti (codice generato, modelli, adattatori)
4. **Validation Layer** — applicazione continua dell'allineamento tra intento ed esecuzione
5. **Runtime Layer** — il sistema operativo vero e proprio

| Modello Classico | Modello SDD |
|---|---|
| Il codice definisce il comportamento | La specifica definisce il comportamento |
| L'architettura è consigliativa | L'architettura è eseguibile e applicabile |
| La deriva è scoperta dopo il fatto | La deriva è prevenuta pre-runtime |
| La validazione è retrospettiva | La validazione è continua |
| Il runtime è emergente | Il runtime è architettonicamente deterministico |

Un concetto centrale è il **drift detection**: la capacità architettonica di rilevare qualsiasi divergenza tra l'intento dichiarato e il comportamento osservato. La deriva non riguarda solo lo schema, ma anche payload non documentati, campi omessi nei refactor, messaggi senza versionamento coordinato, degradazione dei perimetri di sicurezza. Il rilevamento avviene tramite validazione dello schema, ispezione dei payload, contract testing e integrazione nelle CI pipeline.

Gli autori introducono il concetto di **SpecOps** (Specification Operations), che definisce cinque capacità fondamentali di un sistema spec-nativo: (1) authoring come superficie di engineering di prima classe, (2) validazione formale e type enforcement, (3) generazione deterministica — specifiche identiche producono sempre artefatti identici, (4) conformance continua a runtime, (5) evoluzione governata con controllo esplicito della compatibilità.

> "The unit of delivery is no longer a service or a codebase. The unit of delivery becomes the specification itself."

SDD non elimina il ruolo umano, ma lo **riloca a un piano di controllo superiore**: gli umani rimangono custodi della semantica di dominio, della tolleranza al rischio e dei vincoli etici/legali. I cambiamenti di schema breaking, le modifiche alle policy e i refactor proposti dall'AI richiedono approvazione umana esplicita. I trade-off sono reali: le specifiche ereditano complessità, debito tecnico e inerzia di compatibilità tipici del codice; la fiducia nel generatore diventa un problema di supply chain; la validazione runtime ha un costo computazionale; il cambio cognitivo richiesto agli engineer è significativo.

---

## Esempi pratici

**Specification Layer: definizione di un Order Management Service**

```yaml
service: Orders
api:
  POST /orders:
    request:
      Order:
        id: uuid
        quantity: int > 0
    responses:
      201: OrderAccepted
      400: ValidationError

policies:
  compatibility: backward-only
  security:
    auth: mTLS
```

**Artifact Layer: TypeScript generato dalla spec**

```typescript
export interface Order {
  id: string;
  quantity: number;
}

// Validator generato
if (order.quantity <= 0) {
  throw new ValidationError("quantity must be greater than zero");
}
```

---

## Link esterni

- [Rise of Worse is Better](https://www.dreamsongs.com/RiseOfWorseIsBetter.html) — riferimento storico sull'evoluzione delle astrazioni nel software engineering
- [Ambient Code Repository](https://github.com/ambient-code) — repository degli autori

---

## Immagini

- ![SDD 5 Layer Execution Model](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/spec-driven-development/en/resources/132figure-2-1767777705864.jpg) — modello a 5 strati di SDD
- ![Traditional Software Delivery Pipeline](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/spec-driven-development/en/resources/112figure-3-1767777705864.jpg) — pipeline tradizionale con perdita progressiva di informazione
- ![SDD Governed Software Delivery Pipeline](https://imgopt.infoq.com/fit-in/3000x4000/filters:quality(85)/filters:no_upscale()/articles/spec-driven-development/en/resources/78figure-4-1767777705864.jpg) — pipeline SDD con loop di controllo sulla specifica
