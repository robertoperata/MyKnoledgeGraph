---
tags:
  - context-engineering
  - llm
  - langchain4j
  - redis
  - ai-architecture
feature:
type: article
author: Ricardo Ferreira
source: https://www.infoq.com/presentations/context-engineering-redis-llm-architecture/
date: 2026-09-06
---

# Beyond Prompting: Context Engineering for Production-Grade AI

## Sunto

Ricardo Ferreira, Principal Developer Advocate at Redis, affronta in questa presentazione (registrata a QCon AI Boston 2026) un cambio di paradigma fondamentale nello sviluppo di applicazioni AI: passare dal semplice prompt engineering al **context engineering**, ovvero progettare in modo architetturale il contesto che un sistema LLM effettivamente utilizza durante l'esecuzione. Il punto di partenza è la costruzione di "My Jarvis", un'Alexa skill alimentata da LLM, che ha permesso di identificare cinque problemi chiave legati al contesto: *Context Poisoning* (informazioni irrilevanti che distorcono le risposte), *Context Distraction* (eccesso di dati che impedisce risposte focalizzate), *Context Confusion* (dati miscellanei e incoerenti), *Context Rot* (degrado del contesto nel tempo) e *Context Clash* (informazioni conflittuali su versioni e timing).

Una delle soluzioni architetturali più rilevanti è l'implementazione di un sistema di memoria duale con Redis Agent Memory Server: una **memoria a breve termine** (Short-term Memory) basata su TTL, limitata alla sessione conversazionale, e una **memoria a lungo termine** (Long-Term Memory, LTM) con indicizzazione vettoriale HNSW, in cui i ricordi rilevanti vengono promossi automaticamente dagli LLM tramite embedding semantici. Questo approccio consente di recuperare informazioni tra conversazioni distanti nel tempo, superando il limite del contesto finito delle finestre di prompt.

Il processo completo di context engineering implementato comprende diversi componenti in sequenza: un **Context Injector** per la formattazione strutturata dell'input, un **QueryTransformer** per la compressione e la risoluzione dei pronomi nelle domande (es. "Dove abita lui?" → "Dove abita John Doe?"), un **QueryRouter** per lo smistamento semantico tra knowledge base generica e memorie specifiche dell'utente, un **ContentRetriever** per la ricerca vettoriale, un **ContentAggregator** con reranking via API Cohere (soglia di rilevanza ≥ 80%), e un **SemanticCache** che implementa il pattern cache-aside basato sulla similarità semantica delle query — capace di riconoscere parafrasi equivalenti e ridurre le chiamate all'API OpenAI del 75% o più.

La gestione dei costi è uno degli aspetti più pratici della presentazione. Il principale problema identificato è la crescita esponenziale del consumo di token quando la cronologia della conversazione viene accumulata indefinitamente. La soluzione adottata è `TokenWindowChatMemory` (built-in in LangChain4j), che limita la finestra di contesto ai token massimi del modello e taglia automaticamente i messaggi più vecchi. Una soluzione più sofisticata è la **strategia di summarizzazione**: invece di eliminare il contesto, un LLM estrae fatti, preferenze e affermazioni chiave prima del trimming, mantenendo la rilevanza storica a costo costante nel tempo.

Lo stack tecnico adottato combina **LangChain4j** (implementazione Java di LangChain), **AWS Lambda** (con vincolo di timeout a 8 secondi), **Redis con HNSW indexing** come vector database, **OpenAI GPT** come modello, **Cohere API** per il reranking e il **Redis Agent Memory Server** open-source. Il risultato pratico per un caso d'uso domestico (3 utenti, 10 query/giorno ciascuno): $4,20/mese solo con OpenAI, meno di $1/utente/mese aggiungendo Cohere, contro una crescita esponenziale insostenibile nella versione non ottimizzata.

## Codice

Il sistema implementa la risoluzione dei pronomi tramite `QueryTransformer`:

```java
// LangChain4j QueryTransformer - risolve pronomi e riferimenti impliciti
QueryTransformer queryTransformer = new CompressingQueryTransformer(chatLanguageModel);

// Esempio di trasformazione
// Input: "Dove abita lui?" (con John Doe menzionato in precedenza)
// Output: "Dove abita John Doe?"
```

Implementazione del semantic routing tra due retrievers:

```java
// Router semantico basato su chiamata LLM intermedia
QueryRouter router = new LanguageModelQueryRouter(
    chatLanguageModel,
    Map.of(
        userMemoryRetriever, "Memorie specifiche dell'utente",
        knowledgeBaseRetriever, "Conoscenza generale del dominio"
    )
);
```

Configurazione del semantic cache con soglia di similarità:

```java
// Cache semantica con pattern cache-aside
SemanticCache semanticCache = RedisSemanticCache.builder()
    .embeddingModel(embeddingModel)
    .embeddingStore(redisEmbeddingStore)
    .maxResults(1)
    .minScore(0.85)  // soglia calibrata sperimentalmente
    .build();
```

Configurazione della memoria con finestra a token limitati:

```java
// Token window memory - evita crescita esponenziale del contesto
ChatMemory chatMemory = TokenWindowChatMemory.builder()
    .maxTokens(4096, new OpenAiTokenCountEstimator(GPT_3_5_TURBO))
    .build();
```

Pipeline completa di context engineering prima dell'invocazione LLM:

```java
// Pipeline pre-flight: Context Injection → Transform → Route → Retrieve → Rerank → Cache
RetrievalAugmentor retrievalAugmentor = DefaultRetrievalAugmentor.builder()
    .queryTransformer(queryTransformer)     // compressione pronomi
    .queryRouter(router)                    // routing semantico
    .contentAggregator(rerankingAggregator) // Cohere reranking ≥80%
    .build();
```
