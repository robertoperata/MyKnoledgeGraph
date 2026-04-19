---
tags:
  - ai-agents
  - incident-response
  - llm
  - model-context-protocol
  - data-engineering
feature:
type: article
author: Yarden Wolf
source: https://www.wix.engineering/post/when-ai-becomes-your-on-call-teammate-inside-wix-s-airbot-that-saves-675-engineering-hours-a-month
date: 2026-04-19
---

# When AI Becomes Your On-Call Teammate: Inside Wix's AIRbot

## Sunto

Wix gestisce circa 3.500 pipeline Apache Airflow che processano oltre 4 miliardi di transazioni HTTP al giorno, servendo 250 milioni di utenti. Anche al 99,9% di affidabilità, questa scala garantisce fallimenti quotidiani. Il ciclo tradizionale di incident response — ricezione dell'alert, indagine manuale su più interfacce, analisi dei log, sintesi della root cause — consumava circa 45 minuti per incidente e richiedeva risorse ingegneristiche significative. AIRbot nasce come soluzione per automatizzare questo ciclo.

L'architettura di AIRbot è costruita su tre livelli. Il **connectivity layer** utilizza Slack Socket Mode con WebSocket outbound invece di webhook HTTP, eliminando la necessità di aprire porte nel firewall aziendale — una scelta di sicurezza importante che mantiene il bot invisibile a internet pubblico pur mantenendo piena interattività. È costruito su Slack Bolt Python e wrappato in FastAPI. L'**intelligence layer** utilizza Model Context Protocol (MCP) per la visibilità sull'infrastruttura: un MCP personalizzato per i log Airflow (accesso diretto a S3 via IAM roles), integrazioni con GitHub, Trino, Spark, OpenMetadata per i metadati degli schema, e un sistema di tag per il routing verso i team proprietari. Il **reasoning engine** implementa un'architettura Chain of Thought via LangChain con tre catene sequenziali: Classification → Analysis → Solution, con selezione dinamica del modello (GPT-4o Mini per classificazione, Claude 3.5 Opus per analisi complesse con grandi context window).

Due scenari operativi illustrano il valore pratico del sistema. Nel primo, quando una query Trino fallisce per una colonna mancante, AIRbot recupera il codice SQL da GitHub, confronta lo schema attuale da OpenMetadata, identifica il mismatch e genera automaticamente una pull request correttiva. Nel secondo, quando una pipeline è in timeout per dati upstream mancanti, AIRbot interroga le API interne per identificare la tabella bloccata, risolve il team proprietario tramite il sistema di tag, e instrada l'alert direttamente al team responsabile invece del team downstream — eliminando una fonte classica di confusion e perdita di tempo.

I risultati misurati in 30 giorni sono notevoli: 180 PR candidate generate, 28 mergeate direttamente senza modifiche umane (15% di full automation), risparmio di almeno 15 minuti per incidente, e un totale di 675 ore ingegneristiche risparmiate al mese — equivalente a circa 4 ingegneri FTE. Il costo medio per interazione AI è di circa $0,30, con ROI immediato rispetto al costo del tempo ingegneristico.

La scelta di usare Pydantic per strutturare gli output LLM è un pattern chiave: invece di affidare il downstream parsing a testo libero non deterministico, i modelli vengono vincolati a produrre JSON strettamente tipizzato. Questo "bridging" tra non-determinismo LLM e requisiti deterministici dei sistemi di produzione è una delle lezioni più trasferibili dell'architettura AIRbot.

## Codice

Il sistema implementa tre catene sequenziali di reasoning con selezione dinamica del modello:

```python
# Classification chain (modello economico per alto volume)
classification_chain = LLMChain(
    llm=ChatOpenAI(model="gpt-4o-mini"),
    prompt=classification_prompt,
    output_parser=PydanticOutputParser(pydantic_object=ClassificationResult)
)

# Analysis chain (modello potente per contesti grandi)
analysis_chain = LLMChain(
    llm=ChatAnthropic(model="claude-3-5-opus"),
    prompt=analysis_prompt,
    output_parser=PydanticOutputParser(pydantic_object=AnalysisResult)
)

# Solution chain
solution_chain = LLMChain(
    llm=ChatAnthropic(model="claude-3-5-opus"),
    prompt=solution_prompt,
    output_parser=PydanticOutputParser(pydantic_object=SolutionResult)
)
```

Slack Socket Mode elimina la necessità di esporre endpoint pubblici:

```python
from slack_bolt.adapter.socket_mode import SocketModeHandler
from slack_bolt import App

app = App(token=os.environ["SLACK_BOT_TOKEN"])

# Connessione outbound via WebSocket — nessuna porta in ingresso necessaria
handler = SocketModeHandler(app, os.environ["SLACK_APP_TOKEN"])
handler.start()
```
