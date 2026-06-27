---
tags:
  - llm
  - ai-agents
  - multimodal
  - on-device-ai
  - model-architecture
feature:
type: article
author: Sergio De Simone
source: https://www.infoq.com/news/2026/06/google-gemma4-12b-local-coding/
date: 2026-06-27
---

# Gemma 4 12B Enables On-Device, Multimodal Agentic Workflows with an Encoder-Free Architecture

## Sunto

Google ha rilasciato Gemma 4 12B, un modello da 12 miliardi di parametri che introduce un'architettura innovativa "encoder-free" per il processing multimodale. A differenza dei modelli tradizionali che utilizzano encoder separati per visione e audio accoppiati con un decoder testuale, Gemma 4 12B elabora tutti i tipi di dati attraverso un singolo transformer decoder-only, eliminando la latenza e le inefficienze di memoria dei pipeline multi-encoder.

Per il processing visivo, il modello utilizza un embedder da 35 milioni di parametri che proietta patch di pixel grezzi 48×48 direttamente nello spazio hidden del LLM tramite una singola moltiplicazione matriciale. Le informazioni spaziali vengono codificate tramite coordinate X-Y con lookup factorizzato, permettendo al modello di comprendere la posizione degli elementi nell'immagine senza un encoder visivo separato. Per l'audio, viene applicata una proiezione lineare diretta di frame audio a 16 kHz (40ms/640 campioni) nello spazio di input del LLM, eliminando completamente gli encoder audio tradizionali.

L'architettura a pesi unificati è particolarmente significativa: usando gli stessi pesi per tutte le modalità, il fine-tuning tramite adapter come LoRA diventa più efficiente perché le adattazioni si propagano uniformemente attraverso tutte le capacità modali. Questo contrasta con l'approccio tradizionale dove il fine-tuning di un encoder specializzato richiede allineamento con il decoder sottostante.

Le capacità operative di Gemma 4 12B includono la generazione ed esecuzione di codice da linguaggio naturale, il processing autonomo di dati, la creazione di visualizzazioni e la costruzione di pagine web — tutto eseguibile localmente su hardware consumer. Il modello è accessibile tramite Google AI Edge Gallery, LiteRT-LM, Hugging Face, Ollama e LM Studio.

La ricezione nella comunità dei developer ha evidenziato due aspetti come particolarmente innovativi: l'efficienza dell'architettura encoder-free che riduce i requisiti di memoria runtime, e il supporto audio nativo che apre scenari di workflow agentici multimodali on-device precedentemente impossibili senza cloud. Questo posiziona Gemma 4 12B come un modello orientato al deployment edge per applicazioni che richiedono bassa latenza e privacy dei dati.
