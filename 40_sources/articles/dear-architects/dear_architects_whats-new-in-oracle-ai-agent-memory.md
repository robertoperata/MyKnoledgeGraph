---
tags:
  - ai-agents
  - memory-management
  - oracle-database
  - vector-search
  - llm
feature:
type: article
author: Daniela Pavlenco
source: https://blogs.oracle.com/developers/whats-new-in-oracle-ai-agent-memory-custom-extraction-hybrid-search-and-more-control
date: 2026-07-12
---

# What's New in Oracle AI Agent Memory: Custom Extraction, Hybrid Search, and More Control

## Sunto

L'articolo presenta le nuove funzionalità di Oracle AI Agent Memory 26.6, un layer di memoria basato su database che aiuta le applicazioni a gestire messaggi, record, memorie durevoli, riassunti e contesto pronto per i prompt usando Oracle AI Database. Il sistema supporta sia la memoria a breve termine che a lungo termine per workflow di agenti multi-turn, affrontando uno dei problemi più critici nello sviluppo di agenti AI: la gestione efficace del contesto tra sessioni diverse.

La versione 26.6 introduce **workflow di estrazione più veloci** separando l'elaborazione della memoria in due percorsi distinti: il percorso in primo piano (foreground) in cui l'utente riceve la risposta successiva dell'agente, e il percorso di consolidamento (background) in cui i fatti durevoli vengono estratti e indicizzati in modo asincrono. Questa separazione riduce la latenza percepita pur mantenendo la coerenza dei dati. Una delle aggiunte più significative è la **ricerca ibrida** che combina il richiamo semantico vettoriale con la corrispondenza testuale esatta: questo risolve le limitazioni del solo retrieval vettoriale per identificatori strutturati come codici di ordine, numeri di fattura e SKU che richiedono corrispondenza precisa, non semantica.

Le **istruzioni di estrazione personalizzate** permettono di guidare quali informazioni diventano memorie durevoli rispetto a dettagli temporanei, rendendo la formazione della memoria selettiva anziché comprensiva: vengono preservati i fatti specifici del dominio escludendo saluti e credenziali. Le **Context Card** compattano conversazioni lunghe in contesto pronto per il prompt, combinando riassunti, argomenti di recupero, memorie rilevanti e messaggi recenti con la possibilità di specificare conteggi minimi per tipi specifici di memoria. Il sistema distingue chiaramente tra memoria a breve termine (mantenere le conversazioni focalizzate tramite riassunti e context card) e a lungo termine (preservare informazioni cross-sessione come preferenze, policy e identificatori).

I benchmark dimostrano l'efficacia del sistema: 94.4% su LongMemEval per retrieval e reasoning multi-sessione, 68.5% su BEAM 100K per storie di 100K token, 65.6% su BEAM 1M e 50.3% su BEAM 10M, mostrando una degradazione controllata anche su contesti enormi. Dal punto di vista della sicurezza, l'articolo raccomanda connessioni database cifrate, di escludere segreti da prompt e metadati, di implementare autenticazione utente prima delle operazioni di memoria e di non usare il testo derivato dalla memoria per autorizzare azioni sensibili.

## Codice

Creazione di un thread con gestione della memoria e TTL configurabile:

```python
thread = memory.create_thread(
    thread_id=THREAD_ID,
    user_id=USER_ID,
    agent_id=AGENT_ID,
    metadata={"tenant_id": TENANT_ID, "case_id": CASE_ID, "status": "open"},
)

await thread.add_memory_async(
    "The customer prefers SMS for support updates.",
    memory_type="preference",
    metadata=CASE_METADATA,
    ttl_days=180,
    ttl_anchor=TimeToLiveAnchor.CREATED_AT,
)
```

Configurazione della ricerca ibrida che combina retrieval vettoriale e full-text:

```python
store = OracleDBMemoryStore(
    embedder=db_embedder,
    pool=db_pool,
    search_strategy=SearchStrategy.HYBRID,
    search_index_sync=SearchIndexSyncMode.ON_COMMIT,
)
```

Ricerca con filtri sui metadati per delimitare il perimetro di retrieval per tenant e stato:

```python
extracted_hits = await thread.search_async(
    "damaged ORDER-7421 and preferred update channel",
    exact_user_match=True,
    metadata_filter={
        "tenant_id": TENANT_ID,
        "tags": {"$array_contains": "damaged-delivery"},
        "review": {"status": "approved"},
    },
)
```

Assemblaggio di una Context Card con requisiti minimi per tipo di memoria:

```python
context_card = await thread.get_context_card_async(
    fallback_message_count=6,
    max_relevant_results=8,
    min_relevant_results_by_type={
        "preference": 1,
        "guideline": 1,
    },
)
```

Aggiornamento di una memoria esistente (operazione non-append-only):

```python
updated_id = await thread.update_memory_async(
    CONTACT_PREFERENCE_ID,
    content="The customer prefers email for support updates.",
    metadata=replacement_metadata,
    ttl_days=180,
)
```
