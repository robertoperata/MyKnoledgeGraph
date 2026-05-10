---
tags:
  - small-language-models
  - reasoning-strategies
  - agent-orchestration
  - chain-of-thought
  - ai-engineering
feature:
type: article
author: Nacho Martinez
source: https://blogs.oracle.com/developers/16-ways-to-make-a-small-language-model-think-bigger
date: 2026-05-10
---

# 16 Ways to Make a Small Language Model Think Bigger

## Sunto

I modelli linguistici di piccole dimensioni (SLM) possono raggiungere prestazioni paragonabili a quelle dei modelli enormi se avvolti nelle giuste strategie di ragionamento. L'articolo di Nacho Martinez, Data Scientist Advocate di Oracle, dimostra come un modello da 270M di parametri, con il framework corretto, possa eguagliare o superare modelli ben più grandi su benchmark complessi. Il punto centrale è che gran parte dei progressi nell'IA moderna deriva dall'orchestrazione — prompting strutturato, ricerca, flusso di controllo — e non solo dall'aumento delle dimensioni del modello.

L'articolo classifica le 16 strategie in quattro famiglie. Le **strategie sequenziali** (Chain of Thought, Decomposed Prompting, Least-to-Most) elaborano il problema in passi lineari e sono le più veloci. Le **strategie a ramificazione** (Tree of Thoughts, Self-Consistency, MCTS) esplorano più percorsi di ragionamento in parallelo, offrendo maggiore accuratezza a costo di latenza più alta — ToT è 5-8x più lento di CoT. Le **strategie riflessive** (Self-Reflection, Adversarial Debate, Refinement Loop) adottano cicli bozza-critica-revisione, ideali per contenuti tecnici di alta qualità. Le **strategie cross-dominio e meta** (Analogy-Based Reasoning, Socratic Questioning, ReAct, MetaReasoningAgent) permettono trasferimento di conoscenza e integrazione con strumenti esterni.

Il framework implementa un `ReasoningInterceptor` che funziona come client drop-in per Ollama: basta aggiungere un tag `+Strategy` al nome del modello (es. `gemma3:270m+cot`) per attivare la strategia desiderata, senza modificare il codice applicativo. Sono supportati oltre 55 alias che mappano alle 16 classi agente. È possibile usarlo come libreria Python, come proxy di rete su localhost:8080, o integrato in pipeline LangChain.

I benchmark condotti su 4.200 valutazioni con `qwen3.5:9b` mostrano che Chain of Thought ottiene la migliore media (88,7% su GSM8K, MMLU e ARC-Challenge). Tree of Thoughts e Self-Consistency raggiungono 76,7% su GSM8K. ReAct eccelle nei task che richiedono accesso a strumenti come ricerca web o calcolatrici. La scelta della strategia dipende dal tipo di problema: CoT per task sequenziali, ToT per puzzle con soluzioni multiple, Self-Reflection per scrittura tecnica, ReAct per query fattuali che necessitano di dati aggiornati.

L'approccio MetaReasoningAgent (Auto Router) classifica automaticamente l'input in una delle undici categorie e instrada verso la strategia ottimale con una sola invocazione LLM, eliminando la necessità di selezionare manualmente la strategia. Il codice e i benchmark sono open-source nel repository Oracle AI Developer Hub.

## Codice

Installazione del framework tramite pip o uv:

```bash
pip install agent-reasoning
# oppure
uv add agent-reasoning
```

Esempio di invocazione con la strategia Chain of Thought tramite tag nel nome modello:

```python
# Invece di usare il modello direttamente:
# model = "gemma3:270m"
# Si aggiunge il tag +cot per attivare Chain of Thought:
model = "gemma3:270m+cot"

# Oppure +meta per il routing automatico:
model = "gemma3:270m+meta"

# Il ReasoningInterceptor intercetta la chiamata, rimuove il tag,
# e instrada al corrispondente agente di ragionamento.
```

Esempio di configurazione del router automatico con routing per categoria:

```python
# MetaReasoningAgent classifica l'input e seleziona automaticamente
# tra le 16 strategie disponibili. Supporta alias come:
# "cot", "chain_of_thought", "CoT" → CotAgent
# "tot", "tree_of_thoughts" → TreeOfThoughtsAgent
# "react" → ReActAgent (con accesso a strumenti esterni)
```
