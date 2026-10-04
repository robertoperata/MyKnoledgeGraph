---
tags:
  - async-api
  - event-driven-architecture
  - api-governance
  - schema-registry
  - cloud-events
feature:
type: article
author: Ian Cooper
source: https://www.infoq.com/presentations/managing-async-apis/
date: 2026-10-04
---

# Managing Asynchronous APIs at Scale

## Sunto

Ian Cooper, architetto con oltre 30 anni di esperienza in contesti come Just Eat Takeaway e Reuters, presenta in questa sessione di QCon London 2026 un framework sistematico per gestire le API asincrone nelle architetture event-driven su larga scala. Il punto di partenza è la constatazione che le API asincrone tendono a essere trattate con minore rigore rispetto alle API sincrone, pur essendo altrettanto critiche per la coerenza e l'affidabilità dei sistemi distribuiti.

Il framework proposto si basa sulle "ABCs": **Address** (dove si trovano gli endpoint), **Binding** (il protocollo specifico utilizzato), e **Contract** (la struttura e i metadati dei messaggi). Questi tre pilastri guidano la governance delle API asincrone e si declinano in tre aree operative: Discovery (individuazione degli endpoint e comprensione dei contratti), Governance (coerenza degli schemi e loro evoluzione sicura), e Provisioning (deployment e gestione dell'infrastruttura).

La gestione degli standard è centrale nell'approccio di Cooper. **AsyncAPI** viene presentato come il corrispettivo di OpenAPI per i sistemi asincroni: uno standard di documentazione che definisce endpoint, canali, binding e contratti di messaggi. **CloudEvents** standardizza i metadati dei messaggi con attributi come ID, source, type e timestamp, abilitando l'interoperabilità tra sistemi eterogenei. I **Schema Registry** applicano regole di compatibilità e validazione degli schemi, prevenendo breaking changes non intenzionali.

Un caso concreto viene dall'implementazione di Just Eat Takeaway, che gestisce circa 603 specifiche AsyncAPI e 2.000 schemi di messaggistica distribuiti su tre regioni AWS. La pipeline di governance elabora file YAML, genera artifact JSON, esegue validazione degli schemi con verifica della compatibilità, effettua provisioning dell'infrastruttura tramite Pulumi e utilizza il catalogo Marmot per la visibilità degli asset. Strumenti come EventCatalog, Marmot e xRegistry sono citati come soluzioni per la discovery e la governance delle API.

La tesi centrale di Cooper è che le API asincrone meritano lo stesso livello di rigore architetturale delle API sincrone. Le specifiche devono essere la fonte di verità, la governance deve essere implementata attraverso l'automazione della pipeline (non attraverso la documentazione manuale), e i schema registry con modalità di compatibilità configurabili consentono un'evoluzione sicura nel tempo senza rompere i consumer esistenti.
