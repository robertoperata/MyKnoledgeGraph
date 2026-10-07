---
tags:
  - llm-architecture
  - software-architecture
  - ai-engineering
  - functional-decomposition
  - microservices
feature:
type: article
author: Chris Richardson
source: https://microservices.io
date: 2026-10-07
---

# Architecting with LLMs: use plain old code when you can, LLMs when you must

## Sunto

L'articolo di Chris Richardson affronta un problema fondamentale nello sviluppo di sistemi basati su LLM: l'anti-pattern del "Golden Hammer", ovvero la tendenza a usare gli LLM per ogni problema indipendentemente dal fatto che siano lo strumento più adatto. L'autore propone una regola pratica che deriva dalla sua esperienza con il "Coding agent sandwich": usare codice tradizionale (Plain Old Code, POC) quando è possibile specificare come calcolare la risposta, e usare un LLM solo quando non è possibile farlo.

Il POC è la scelta giusta quando le regole sono note e la correttezza può essere definita con precisione. Offre ripetibilità (stesso input, stesso output), facilità di debug (i fallimenti sono riproducibili) e prestazioni superiori in termini di costo e velocità rispetto a un'invocazione LLM. Al contrario, un LLM è appropriato quando l'input è ambiguo o non strutturato, quando è richiesta la comprensione del linguaggio naturale, o quando produrre la risposta richiede giudizio e interpretazione — ad esempio: interpretare una richiesta di supporto operativo, tradurre l'intenzione dell'utente in una transazione enterprise, o diagnosticare un problema di produzione sconosciuto.

L'approccio di design raccomandato inizia con la decomposizione funzionale ricorsiva: si identificano le responsabilità necessarie per risolvere il problema (inclusa l'orchestrazione), poi si decide per ciascuna se implementarla con POC o LLM. Applicare il "POC-or-LLM decision" separatamente per ogni responsabilità, invece di usare un LLM per tutto, è il modo per evitare sprechi e inefficienze. Quando si assegna una responsabilità a un LLM, è importante mantenerla strettamente definita: un'invocazione che gestisce troppe responsabilità è meno affidabile e più costosa.

Per l'orchestrazione valgono le stesse regole: se i passi e i rami del flusso di controllo possono essere definiti in anticipo, usare POC orchestration; se non possono, usare LLM-based orchestration. Un agente (LLM + harness + tools) è la scelta quando l'LLM deve decidere ogni azione successiva dopo aver visto il risultato della precedente — il harness (come Claude Code) è POC che esegue il loop. Agli LLM devono essere forniti strumenti POC per le operazioni deterministiche (eseguire test, interrogare database, invocare API).

Infine, l'autore osserva che applicare la regola POC-or-LLM tende a collocare gli LLM ai confini del sistema, dove ricevono input non strutturati dal mondo reale che devono essere interpretati e convertiti in stato strutturato su cui il POC può operare. La decisione POC-or-LLM non è definitiva: può cambiare con la crescita della comprensione del problema, andando sia nella direzione POC→LLM (quando le regole diventano troppo complesse) che LLM→POC (quando si riesce a specificare il calcolo).

## Immagini

![Albero di decomposizione funzionale del workflow implement plan con POC (grigio) e LLM (giallo)](https://microservices.io/i/llms/functional-decomposition-tree.svg)

![Schema di decomposizione funzionale gerarchica delle responsabilità](https://microservices.io/i/llms/functional-decomposition.svg)

![Diagramma decisionale POC-or-LLM per ciascuna responsabilità](https://microservices.io/i/llms/poc-or-llm-decision.svg)

![LLM con strumenti POC deterministici - lo strato inferiore del Coding agent sandwich](https://microservices.io/i/llms/llm-deterministic-tools.svg)

![LLM ai confini del sistema: interpreta input non strutturati, il POC gestisce il resto](https://microservices.io/i/llms/llm-at-the-boundary.svg)
