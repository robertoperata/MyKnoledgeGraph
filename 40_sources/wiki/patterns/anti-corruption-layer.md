---
title: Anti-Corruption Layer (ACL)
type: pattern
tags: [thread1-microservices, thread5-ddd]
sources:
  - "[[microsoft_anti-corruption-layer-pattern]]"
updated: 2026-05-08
related:
  - "[[concepts/bounded-context]]"
  - "[[concepts/legacy-modernization]]"
  - "[[concepts/hexagonal-architecture]]"
  - "[[concepts/information-hiding]]"
  - "[[patterns/strangler-fig]]"
---

# Anti-Corruption Layer (ACL)

## Problema che risolve

Quando un sistema moderno deve integrarsi con un sistema legacy (o con qualsiasi sistema esterno non controllabile dal team), è costretto ad adottare le strutture dati, i protocolli e i modelli concettuali dell'altro sistema. Questo inquina il design del sistema nuovo, che si trova a dover supportare API obsolete, schemi contorti o semantiche incompatibili che non avrebbe mai scelto autonomamente.

Il risultato è **corrup­zione del modello**: i concetti del sistema esterno si infiltrano nel dominio interno, deteriorando la chiarezza del design e accumulando technical debt.

> "Maintaining access between new and legacy systems can force the new system to adhere to at least some of the legacy system's APIs or other semantics. When these legacy features have quality issues, supporting them 'corrupts' what might otherwise be a cleanly designed modern application." — Azure Architecture Center

## Struttura

Inserire uno strato di façade o adapter tra i due sistemi. Lo strato contiene **tutta la logica di traduzione**:

```
Sistema A  ───────►  Anti-Corruption Layer  ───────►  Sistema B
(modello A)          (traduzione A ↔ B)              (modello B)
```

- Il Sistema A comunica con l'ACL usando **il proprio modello e linguaggio**.
- L'ACL traduce e parla con il Sistema B usando **il modello del Sistema B**.
- Nessuno dei due sistemi conosce o dipende dalle strutture interne dell'altro.

L'ACL può essere implementato come **componente interno** all'applicazione oppure come **servizio indipendente** deployato separatamente.

## Trade-off

| Pro | Contro |
|---|---|
| Il modello del sistema moderno rimane pulito | Latenza aggiuntiva su ogni chiamata cross-sistema |
| Il legacy può restare invariato durante la migrazione | Servizio aggiuntivo da gestire, monitorare, rilasciare |
| Riduce il technical debt nel nuovo sistema | Può diventare un collo di bottiglia se non scalato |
| Disaccoppia i cicli di vita dei due sistemi | Richiede sforzo per definire e mantenere la traduzione |

## Quando preferirlo ad alternative

**Usa ACL quando:**
- La migrazione avviene in più fasi e la coesistenza nuovo/legacy è temporanea ma necessaria
- I due sistemi hanno semantiche fondamentalmente diverse (termini diversi per gli stessi concetti, o stessi termini con significati diversi)
- Il sistema esterno ha scarsa qualità (API obsolete, schema convoluto)
- Il team non controlla il sistema esterno e non può cambiarne l'interfaccia

**Non usare ACL quando:**
- Le differenze semantiche tra i sistemi sono minime — il costo non è giustificato
- I sistemi condividono già un modello standardizzato (es. entrambi parlano OpenAPI ben progettato)

## Ciclo di vita

Decisione esplicita da prendere: l'ACL è **permanente** o **temporaneo**?

- **Permanente**: quando il sistema esterno non sarà mai sostituito e l'integrazione è strutturale
- **Temporaneo**: quando è un abilitatore della migrazione (Strangler Fig) destinato a essere ritirato al completamento

Documentare esplicitamente questa scelta per evitare che un ACL "temporaneo" diventi permanente per inerzia.

## Connessioni

- [[concepts/bounded-context]] — l'ACL è lo strumento tecnico che preserva l'integrità del bounded context nell'integrazione con contesti esterni a semantiche diverse
- [[concepts/legacy-modernization]] — l'ACL abilita la migrazione incrementale (Strangler Fig): il nuovo sistema può evolvere senza adottare il modello legacy
- [[concepts/hexagonal-architecture]] — l'ACL è concettualmente un outbound adapter esteso a livello inter-sistema; stessa filosofia di isolamento, scope diverso
- [[concepts/information-hiding]] — l'ACL è information hiding applicato ai confini inter-sistema: nessuno dei due sistemi vede le strutture interne dell'altro
- [[patterns/strangler-fig]] — pattern complementare: Strangler Fig definisce la strategia di migrazione, ACL abilita la coesistenza durante le fasi intermedie
