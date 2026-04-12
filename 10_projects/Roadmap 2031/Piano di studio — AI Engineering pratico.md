_LangChain4j, Spring AI, pgvector, RAG e Python minimo per developer Java_

---

## Premessa: Spring AI vs LangChain4j

Esistono due librerie Java principali per integrare LLM in applicazioni Spring Boot:

||Spring AI|LangChain4j|
|---|---|---|
|Origine|Team Spring ufficiale|Community, ispirato a LangChain Python|
|Stile|Convention over configuration, autoconfiguration|API più espressiva, più vicina a LangChain Python|
|Integrazione Spring|Nativa e profonda|Buona, tramite starter|
|Casi d'uso forti|RAG, vector store, setup rapido|Tool calling, agent, memory conversazionale|
|Requisiti|Java 17+, Spring Boot 3.5+|Java 17+, Spring Boot 3.x|

**Strategia consigliata:** Spring AI come percorso principale per l'80% dei casi. LangChain4j da esplorare per i pattern avanzati (agent, tool calling). Le competenze si trasferiscono — i concetti RAG sono identici in entrambe.

---

## Python per AI Engineering — valore aggiunto senza dispersione

### La domanda giusta: quale Python?

Python è un ecosistema enorme. Il rischio è investire energie su cose che non portano valore al posizionamento Java enterprise. La chiave è capire esattamente dove Python aggiunge qualcosa che Java non può dare.

### Dove Python è irrinunciabile

**1. Valutazione dei modelli (LLM Evals)** RAGAS, DeepEval, LangSmith, Arize — tutti i framework seri per misurare la qualità di un sistema RAG sono Python. Non hanno equivalenti Java maturi. Per essere credibile sulla parte "valutazione", bisogna saper leggere e adattare codice Python per gli eval.

**2. Prototipazione rapida** Quando un cliente chiede "funzionerebbe questo approccio?", costruire un proof of concept in LangChain Python in 2 ore è molto più veloce che farlo in Java. Non per andare in produzione — per dimostrare fattibilità velocemente. Questa capacità vale molto in un ruolo consulenziale.

**3. Leggere la letteratura tecnica** I paper, i blog post di Anthropic/OpenAI/Google, i tutorial avanzati su RAG — tutto è scritto con esempi Python. Non saperlo leggere significa essere sempre in ritardo di 6 mesi rispetto a chi lo legge.

### Dove Python NON serve nel tuo caso

|Da evitare|Perché|
|---|---|
|Machine learning da zero (PyTorch, TensorFlow)|Anni di investimento, non coerente con il posizionamento|
|numpy/pandas avanzato, data science|Percorso da data scientist, diverso dall'AI engineer|
|Data engineering (Airflow, Spark)|Non coerente con il posizionamento Java enterprise|
|Training e fine-tuning di modelli|Richiede background matematico, anni di studio|

### La soglia minima utile — "Python da AI Engineer"

Diverso dal "Python da data scientist". Si raggiunge in 4-6 settimane partendo da zero (meno con esperienza pregressa).

**Cosa saper fare:**

```python
# 1. Gestire ambiente e dipendenze
python -m venv .venv
source .venv/bin/activate
pip install langchain openai ragas python-dotenv

# 2. Leggere e modificare script esistenti
# 3. Usare le librerie specifiche AI
# 4. Scrivere script semplici per test e eval
```

**Librerie da conoscere — e basta:**

|Libreria|Perché|
|---|---|
|`langchain`|Leggere tutorial e paper con esempi Python|
|`openai`|SDK ufficiale, per test rapidi e script di eval|
|`ragas`|Framework di valutazione RAG — la skill più rara|
|`python-dotenv`|Gestione variabili d'ambiente|
|`requests`|Chiamate HTTP a API esterne|
|`tiktoken`|Contare i token prima di fare chiamate LLM|

### Come integrarlo nel piano senza un percorso separato

Non è una fase aggiuntiva — si acquisisce in parallelo mentre si studia RAG.

- Quando si legge un tutorial RAG in Python per capire un concetto → si sta già imparando Python
- Quando si usa RAGAS per valutare il sistema Spring AI → si sta già usando Python produttivamente
- **2-3 ore a settimana durante i mesi 2-4** del piano di studio sono sufficienti per raggiungere la soglia utile

### Il valore aggiunto concreto sul mercato

Un AI Engineer Java che sa anche leggere e scrivere Python per eval e prototipazione è significativamente più raro di uno che sa solo Java.

Nella comunicazione con clienti e nei colloqui:

> _"Prototipo in Python per validare l'approccio, poi porto in produzione in Java"_

Dimostra che si capisce l'intero ecosistema AI, non solo la parte Java. È un differenziatore credibile che non richiede un investimento separato significativo.

### Esempio pratico: eval di un sistema RAG in Python

```python
from ragas import evaluate
from ragas.metrics import faithfulness, answer_relevancy, context_recall
from datasets import Dataset

# Dataset di test: domande, risposte del tuo RAG, contesti recuperati
data = {
    "question": ["Cos'è il pattern Saga?", "Quando usare CQRS?"],
    "answer": [risposta_rag_1, risposta_rag_2],          # output del tuo sistema Java
    "contexts": [contesti_recuperati_1, contesti_recuperati_2],  # chunk pgvector
    "ground_truth": ["risposta_attesa_1", "risposta_attesa_2"]
}

dataset = Dataset.from_dict(data)

# Calcola le metriche
results = evaluate(
    dataset,
    metrics=[faithfulness, answer_relevancy, context_recall]
)

print(results)
# faithfulness: 0.87, answer_relevancy: 0.91, context_recall: 0.79
```

Questo script si scrive in 30 minuti e ti dice immediatamente se il tuo sistema RAG Java funziona bene o no. È il caso d'uso Python più prezioso dell'intero percorso.

### Risorse Python mirate (non corsi generici)

- **"Python for Java Developers"** — qualsiasi tutorial di 2-3 ore che mostri le differenze sintattiche chiave
- **docs.ragas.io** — documentazione RAGAS con esempi pronti
- **python.langchain.com/docs** — per leggere i tutorial avanzati su RAG
- **cookbook.openai.com** — ricettario pratico con script Python per casi d'uso AI

---

## Fase 1 — Settimane 1-3: fondamenta concettuali

Prima del codice. Senza questi concetti il codice non ha senso.

### 1. Cosa sono gli embedding

Un testo viene trasformato in un array di numeri (es. 1536 float per i modelli OpenAI) che rappresenta il suo "significato" nello spazio vettoriale. Testi simili producono vettori vicini. Questo è il motore di tutto.

```
"Il cane corre nel parco"  → [0.12, -0.45, 0.78, ...]  (1536 numeri)
"Un labrador gioca all'aperto" → [0.11, -0.43, 0.76, ...]  (vicino al precedente)
"La banca è chiusa lunedì"  → [-0.34, 0.21, -0.55, ...]  (lontano)
```

### 2. Il flusso RAG completo — 5 fasi

```
┌─────────────────────────────────────────────────────────┐
│  INGESTION (offline, una volta)                         │
│                                                         │
│  Documento → Chunking → Embedding → pgvector            │
└─────────────────────────────────────────────────────────┘
                          ↕
┌─────────────────────────────────────────────────────────┐
│  RETRIEVAL + GENERATION (online, ad ogni query)         │
│                                                         │
│  Query → Embedding → Vector Search → Top-K chunk        │
│       → [chunk + query] → LLM → Risposta                │
└─────────────────────────────────────────────────────────┘
```

Le 5 fasi in dettaglio:

1. **Ingest** — carica documenti (PDF, MD, HTML, testo), dividili in chunk
2. **Embed** — converti ogni chunk in vettore tramite un embedding model
3. **Store** — salva i vettori in pgvector con i metadati del chunk originale
4. **Retrieve** — a fronte di una query, trova i K chunk più simili (vector search)
5. **Generate** — passa i chunk recuperati + la query originale all'LLM come contesto

### 3. Chunking strategy — spesso la variabile più importante

Come dividi il documento in pezzi impatta la qualità delle risposte più del modello LLM scelto.

|Strategia|Quando usarla|Pro / Contro|
|---|---|---|
|Fixed-size|Baseline, documenti omogenei|Semplice, può spezzare frasi|
|Sentence-based|Testi narrativi|Rispetta la grammatica|
|Semantic chunking|Documenti tecnici, manuali|Alta qualità, più costoso|
|Recursive|Documenti strutturati (MD, HTML)|Rispetta la struttura, consigliato|

**Regola pratica:** inizia con chunk da 512-1024 token con overlap del 10-20%. Poi misura e ottimizza.

### Risorse per questa fase

- Blog ufficiale LangChain (Python, ma i concetti sono identici)
- YouTube: "What is RAG?" — canale IBM Technology (ottimo per i concetti)
- Articolo: "Chunking strategies for RAG" — Pinecone Blog

---

## Fase 2 — Settimane 4-8: primo progetto funzionante

### Setup ambiente locale (settimana 4)

**Obiettivo: avere tutto funzionante in locale senza costi API**

**PostgreSQL + pgvector via Docker:**

```bash
docker run -it --rm --name pgvector \
  -p 5432:5432 \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=postgres \
  pgvector/pgvector:pg16
```

**Ollama — modelli LLM in locale:**

```bash
# Installa Ollama da ollama.ai
ollama pull llama3.2          # modello chat
ollama pull nomic-embed-text  # modello embedding
```

Con Ollama puoi sperimentare liberamente senza preoccuparti dei costi API. In produzione si switcha a OpenAI o Anthropic con una riga di configurazione.

---

### Stack principale: Spring AI + pgvector

**Dipendenze Maven (pom.xml):**

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.springframework.ai</groupId>
      <artifactId>spring-ai-bom</artifactId>
      <version>1.0.0</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>

<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>

  <!-- Spring AI con OpenAI (oppure Ollama starter per locale) -->
  <dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
  </dependency>

  <!-- pgvector store -->
  <dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-pgvector-store-spring-boot-starter</artifactId>
  </dependency>

  <!-- Driver PostgreSQL -->
  <dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
  </dependency>
</dependencies>
```

**Configurazione (application.yml):**

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/postgres
    username: postgres
    password: postgres

  ai:
    # Per Ollama locale:
    ollama:
      chat:
        model: llama3.2
      embedding:
        model: nomic-embed-text

    vectorstore:
      pgvector:
        index-type: HNSW
        distance-type: COSINE_DISTANCE
        dimensions: 768          # nomic-embed-text = 768, OpenAI text-embedding-3-small = 1536
        initialize-schema: true  # crea la tabella vector_store automaticamente
```

**Codice: ingestion dei documenti:**

```java
@Service
public class DocumentIngestionService {

    private final VectorStore vectorStore;

    public DocumentIngestionService(VectorStore vectorStore) {
        this.vectorStore = vectorStore;
    }

    public void ingestMarkdown(String filePath) {
        // Leggi il file
        Resource resource = new FileSystemResource(filePath);

        // Carica e chunka il documento
        DocumentReader reader = new TextReader(resource);
        List<Document> documents = reader.get();

        // Chunking con overlap
        TokenTextSplitter splitter = new TokenTextSplitter(512, 128, 5, 10000, true);
        List<Document> chunks = splitter.apply(documents);

        // Embedding + store (Spring AI fa tutto automaticamente)
        vectorStore.add(chunks);
    }
}
```

**Codice: RAG query:**

```java
@Service
public class RagService {

    private final ChatClient chatClient;
    private final VectorStore vectorStore;

    public RagService(ChatClient.Builder builder, VectorStore vectorStore) {
        this.chatClient = builder.build();
        this.vectorStore = vectorStore;
    }

    public String query(String userQuestion) {
        return chatClient.prompt()
            .advisors(new QuestionAnswerAdvisor(vectorStore,
                SearchRequest.defaults().withTopK(5)))
            .user(userQuestion)
            .call()
            .content();
    }
}
```

**Codice: controller REST:**

```java
@RestController
@RequestMapping("/api/rag")
public class RagController {

    private final RagService ragService;
    private final DocumentIngestionService ingestionService;

    // costruttore...

    @PostMapping("/ingest")
    public ResponseEntity<String> ingest(@RequestParam String filePath) {
        ingestionService.ingestMarkdown(filePath);
        return ResponseEntity.ok("Documento indicizzato con successo");
    }

    @GetMapping("/query")
    public ResponseEntity<String> query(@RequestParam String q) {
        return ResponseEntity.ok(ragService.query(q));
    }
}
```

---

### Primo progetto concreto da costruire

**"Chatbot sulla documentazione tecnica del tuo progetto"**

1. Prendi la documentazione del progetto su cui lavori (README, ADR, wiki)
2. Caricala nel sistema tramite l'endpoint `/ingest`
3. Fai domande tramite `/query`
4. Valuta le risposte — sai già se sono corrette

Questo è più efficace dei dataset di esempio: puoi valutare immediatamente la qualità perché conosci il contenuto.

**Consiglio pratico:** inizia con i file markdown di questa conversazione. Hai già del materiale strutturato su BPM, state machine, KNN, AI engineering — è un ottimo knowledge base di test.

---

## Fase 2b — Settimane 6-8: LangChain4j

### Perché impararlo dopo Spring AI, non invece

LangChain4j ha un'API più espressiva per pattern avanzati. Vale la pena conoscerlo per:

- `@AiService` — interfaccia dichiarativa molto pulita
- Tool calling — LLM che chiama funzioni Java
- Memory conversazionale — gestione della storia del dialogo
- Agent framework — LLM che prende decisioni autonome

### Dipendenza Maven:

```xml
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-open-ai-spring-boot-starter</artifactId>
    <version>1.0.0-beta2</version>
</dependency>
```

### Pattern @AiService — il concetto chiave:

```java
@AiService
interface DocumentAssistant {

    @SystemMessage("Sei un assistente tecnico. Rispondi solo in base ai documenti forniti.")
    @UserMessage("{{query}}")
    String answer(@V("query") String query);
}
```

Spring AI autoconfigura l'implementazione. Si inietta come qualsiasi `@Service`.

### Tool calling — esempio:

```java
@Component
class ProjectTools {

    @Tool("Recupera lo stato di un ticket Jira dato il suo ID")
    String getTicketStatus(String ticketId) {
        // chiamata alla tua API interna
        return jiraService.getStatus(ticketId);
    }
}
```

L'LLM può chiamare questo tool autonomamente quando necessario.

### Risorse LangChain4j:

- **docs.langchain4j.dev** — documentazione ufficiale, molto buona
- GitHub: `langchain4j/langchain4j-examples` — esempi pronti da clonare
- Tutorial: "Building a Booking ChatBot with Spring Boot and LangChain4j" (Medium, Giuseppe Trisciuoglio)

---

## Fase 3 — Mesi 3-5: qualità e produzione

### A. Valutazione delle risposte (LLM Evals)

La skill più rara e più richiesta. Come misuri se il tuo RAG funziona bene?

**Metriche fondamentali (framework RAGAS):**

|Metrica|Cosa misura|
|---|---|
|**Faithfulness**|La risposta è supportata dai documenti recuperati?|
|**Answer relevancy**|La risposta risponde davvero alla domanda?|
|**Context recall**|I chunk recuperati contengono l'informazione necessaria?|
|**Context precision**|I chunk recuperati sono pertinenti (pochi falsi positivi)?|

**Come implementarla praticamente:**

1. Crea un set di 20-50 domande + risposte attese (ground truth)
2. Esegui il RAG su quelle domande
3. Confronta le risposte con il ground truth
4. Misura le metriche sopra
5. Itera su chunking, retrieval e prompt

RAGAS è il framework di riferimento (Python), ma i concetti si applicano a qualsiasi implementazione.

---

### B. Ottimizzazione pgvector

**Indice HNSW — parametri chiave:**

```sql
CREATE INDEX ON documents
USING hnsw (embedding vector_cosine_ops)
WITH (m = 16, ef_construction = 64);

-- m: numero di connessioni per nodo (default 16, range 2-100)
-- ef_construction: dimensione della coda durante la costruzione (default 64)
-- Più alti = più precisi ma più lenti e più memoria
```

**Filtering per metadata:**

```java
// Spring AI supporta filtri sui metadati
SearchRequest.defaults()
    .withTopK(5)
    .withFilterExpression("categoria == 'architettura' && anno >= 2024");
```

---

### C. Hybrid Search e RRF (Reciprocal Rank Fusion)

Uno dei miglioramenti più impattanti su un sistema RAG reale. Combina vector search e full-text search in modo robusto, senza dover calibrare pesi arbitrari tra i due sistemi.

**Il problema del naive hybrid search**

Vector search e full-text search producono score incomparabili: uno è una distanza coseno (0.0–1.0), l'altro un punteggio BM25 che può valere 0.3 o 47.2 a seconda del corpus. Non si possono sommare direttamente.

**RRF — la soluzione**

Ignora gli score assoluti e usa solo la _posizione_ (rank) di ogni documento in ciascuna lista:

```
RRF_score(doc) = 1/(60 + rank_vector) + 1/(60 + rank_fulltext)
```

La costante `k=60` attenua la differenza tra le posizioni: premia la _consistenza_ tra i due sistemi più che il dominio di uno solo. Un documento che appare al 3° posto in entrambe le liste batte quasi sempre un documento che è primo in una sola.

**Strumenti necessari**

|Strumento|Ruolo|Note|
|---|---|---|
|PostgreSQL tsvector/tsquery|Full-text search|Già disponibile in pgvector|
|pgvector|Vector search|Già nel tuo stack|
|Spring AI `EnsembleRetriever`|Fusione RRF in Java|Dalla versione 1.0|
|SQL con FULL OUTER JOIN|RRF manuale|Massimo controllo|

**Implementazione SQL con RRF:**

```sql
WITH vector_results AS (
  SELECT id, content,
         ROW_NUMBER() OVER (ORDER BY embedding <=> query_vec) AS rank
  FROM documents ORDER BY embedding <=> query_vec LIMIT 20
),
text_results AS (
  SELECT id, content,
         ROW_NUMBER() OVER (ORDER BY ts_rank(
           to_tsvector('italian', content),
           plainto_tsquery('italian', :query)
         ) DESC) AS rank
  FROM documents
  WHERE to_tsvector('italian', content) @@
        plainto_tsquery('italian', :query)
  LIMIT 20
)
SELECT
  COALESCE(v.id, t.id) AS id,
  COALESCE(v.content, t.content) AS content,
  (1.0 / (60 + COALESCE(v.rank, 1000))) +
  (1.0 / (60 + COALESCE(t.rank, 1000))) AS rrf_score
FROM vector_results v
FULL OUTER JOIN text_results t ON v.id = t.id
ORDER BY rrf_score DESC
LIMIT 10;
```

Il `COALESCE(..., 1000)` penalizza i documenti che appaiono solo in una lista senza escluderli.

**Implementazione Spring AI:**

```java
@Bean
public QueryTransformer hybridRetriever(
        VectorStore vectorStore,
        JdbcTemplate jdbc) {

    var vectorRetriever = VectorStoreRetriever.builder()
        .vectorStore(vectorStore)
        .topK(20)
        .build();

    var textRetriever = new FullTextRetriever(jdbc); // custom

    return EnsembleRetriever.builder()
        .retrievers(List.of(vectorRetriever, textRetriever))
        .weights(List.of(0.5, 0.5))  // RRF gestisce la fusione
        .build();
}
```

**Quando RRF aiuta di più**

- Query brevi o con termini tecnici specifici (es. "pgvector HNSW") — il vector search da solo fatica perché i termini esatti contano
- Corpus con acronimi, nomi propri, codici — il full-text li trova esatti, il vector li trova per similarità semantica
- Domande vaghe o semantiche (es. "come funziona la ricerca vettoriale") — il vector search eccelle, il full-text aiuta poco ma non penalizza

**Tecnica avanzata: re-ranking dopo RRF**

Dopo RRF si può aggiungere un secondo passaggio con un cross-encoder — un modello che valuta la rilevanza di ogni chunk rispetto alla query in modo più preciso (ma più lento). Architettura completa:

```
Query
  → [Vector search top-20] ─┐
                             ├─ RRF fusion → top-10 → Cross-encoder → top-5 → LLM
  → [Full-text top-20]    ─┘
```

Il cross-encoder opera su un insieme ridotto (10-20 risultati) quindi la latenza aggiuntiva è accettabile.

**Strategia per diventare proficient**

Costruisci lo stesso sistema RAG in tre versioni progressive e misura con RAGAS su ciascuna:

1. Solo vector search — baseline
2. Hybrid search con RRF — confronta le metriche
3. RRF + re-ranking — confronta di nuovo

Vedere i numeri migliorare concretamente (context precision tipicamente +10-20%) è il modo più rapido per interiorizzare quando e perché RRF vale lo sforzo. Tempo stimato per questa progressione: 2-3 settimane nella Fase 3.

---

### D. Pattern RAG avanzati

**HyDE (Hypothetical Document Embeddings):**

1. Chiedi all'LLM di generare una risposta ipotetica alla query
2. Usa quella risposta come query per il vector search
3. I risultati sono spesso più pertinenti — la risposta ipotetica "assomiglia" semanticamente ai documenti reali più della query originale

**Chunking semantico:** Invece di dividere per dimensione fissa, identifica i confini semantici del testo. Più costoso ma produce chunk di qualità superiore.

**Parent-child chunking:** Indicizza chunk piccoli (per precision), ma recupera chunk più grandi (per contesto). Bilancia precision e recall.

---

### D. Osservabilità e costi

**Spring AI + OpenTelemetry:**

```yaml
management:
  tracing:
    enabled: true
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
```

Spring AI emette metriche native per:

- Numero di token per richiesta (input + output)
- Latenza delle chiamate LLM
- Numero di documenti recuperati
- Cache hit rate (se abilitata)

**Stima costi OpenAI (ordini di grandezza):**

- Embedding: ~$0.00002 per 1K token (quasi gratuito)
- GPT-4o: ~$0.005 per 1K token input, $0.015 output
- Una query RAG tipica: 500-2000 token totali → $0.01-0.03 a query

Impostare un budget mensile su OpenAI e monitorare con le metriche è fondamentale prima di andare in produzione.

---

## Il progetto finale — portfolio concreto

### Cosa costruire entro 5 mesi

**"Knowledge base assistant per sistemi enterprise"**

Funzionalità:

- Ingestione di documenti (PDF, Markdown, HTML)
- Chunking configurabile
- Ricerca semantica + filtri per metadati
- Chat con memoria della conversazione
- Dashboard semplice con statistiche (chunk indicizzati, query/giorno, costi)
- Deploy con Docker Compose (app + postgres + pgvector)

**Stack:**

- Spring Boot 3.x
- Spring AI 1.0+
- PostgreSQL + pgvector (HNSW)
- Ollama per sviluppo locale, OpenAI per produzione
- Docker Compose per il deploy

**Perché questo progetto:**

- Dimostra l'intero pipeline RAG end-to-end
- È production-ready, non un toy example
- È applicabile immediatamente a qualsiasi azienda con documentazione interna
- Rappresenta esattamente il profilo "AI integration su stack Java enterprise"

---

## Tabella riassuntiva — sequenza temporale

|Settimana|Obiettivo|Output|
|---|---|---|
|1-3|Concetti: embedding, RAG, chunking|Comprensione solida senza codice|
|4|Setup: Docker, pgvector, Ollama|Ambiente funzionante in locale|
|5-6|Spring AI: primo RAG funzionante|Endpoint /ingest e /query che funzionano|
|7-8|LangChain4j: pattern @AiService, @Tool|Secondo progetto con tool calling|
|7-8 _(parallelo)_|Python base: sintassi, env, openai SDK|Saper leggere e modificare script Python|
|9-10|Evals con RAGAS — baseline vector search|Metriche di qualità misurabili|
|10-12|Hybrid search + RRF — confronto metriche|Miglioramento context precision misurabile|
|12-14|Re-ranking dopo RRF, pattern avanzati|Sistema RAG ottimizzato end-to-end|
|14-20|Produzione: osservabilità, costi, deploy|Progetto portfolio completo|

---

## Risorse di riferimento

### Documentazione ufficiale

- **docs.spring.io/spring-ai** — Spring AI reference
- **docs.langchain4j.dev** — LangChain4j reference
- **github.com/pgvector/pgvector** — pgvector README e esempi SQL
- **docs.ragas.io** — framework di valutazione RAG in Python

### Tutorial pratici

- "Build a RAG-Powered Chat App with Spring AI and PGVector" — sohamkamani.com
- "Building a Booking ChatBot with Spring Boot and LangChain4j" — Medium, Giuseppe Trisciuoglio
- "PostgreSQL + PgVector + Spring AI: Your First Production-Ready RAG Pipeline" — Medium, 2025

### GitHub da clonare e studiare

- `langchain4j/langchain4j-examples` — esempi ufficiali LangChain4j
- `ThomasVitale/langchain4j-spring-boot` — integrazione avanzata LangChain4j + Spring
- `spring-projects/spring-ai` — sorgenti Spring AI con test di integrazione
- `explodinggradients/ragas` — framework eval RAG, ottimo per capire le metriche

### Canali YouTube

- "SpringDeveloper" — canale ufficiale Spring, molti video su Spring AI
- "JFocus conference" — talks tecnici su AI in Java
- "IBM Technology" — spiegazioni concettuali su RAG e LLM

### Python specifico per AI Engineering

- **cookbook.openai.com** — ricettario pratico con script Python per casi d'uso AI
- **python.langchain.com/docs** — per leggere tutorial avanzati su RAG
- Qualsiasi tutorial "Python for Java Developers" di 2-3 ore per le differenze sintattiche

### Nota sui costi durante lo sviluppo

Usa **Ollama** con modelli locali (Llama3, Mistral, nomic-embed-text) per tutto lo sviluppo e la sperimentazione — zero costi. Passa a OpenAI/Anthropic solo per la validazione finale e la produzione.

---

_Documento creato il 29 marzo 2026 — aggiornato con sezione Python per AI Engineering e Hybrid Search / RRF_