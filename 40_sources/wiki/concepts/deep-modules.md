---
title: Deep Modules
type: concept
tags:
  - thread4-ai
  - thread1-microservices
sources:
  - "[[youtube_ai-coding-workflow-matt-pocock]]"
updated: 2026-05-09
related:
  - "[[concepts/information-hiding]]"
  - "[[concepts/agentic-patterns]]"
  - "[[patterns/testing-pyramid]]"
  - "[[concepts/context-management]]"
---

# Deep Modules

## Definizione

Concetto di John Ousterhout (*A Philosophy of Software Design*): un **deep module** ha un'interfaccia piccola e semplice ma una grande quantità di funzionalità interna. Un **shallow module** ha un'interfaccia che non è molto più semplice della sua implementazione.

Il rapporto qualità/costo di un modulo:
- **Costo** = complessità dell'interfaccia (ciò che gli altri devono imparare per usarlo)
- **Beneficio** = funzionalità nascosta (ciò che l'utente del modulo non deve gestire)

## Shallow vs Deep

| Caratteristica | Shallow Module | Deep Module |
|---|---|---|
| File | Molti file piccoli, poche responsabilità | Pochi moduli con molta logica interna |
| Interfaccia | Molte funzioni/metodi, spesso pass-through | Poche funzioni con contratto chiaro |
| Dipendenze | Grafo intricato tra file | Frontiera netta tra interno ed esterno |
| Testing | Confini di test non chiari | Modulo = unità naturale di test |

## Perché i deep modules sono critici con l'AI

### Navigabilità nella smart zone

L'AI deve tracciare il grafo delle dipendenze tra file per capire cosa fa il sistema. Con shallow modules il grafo è enorme e la smart zone (~100k token) si esaurisce prima di ottenere una comprensione completa. Con deep modules, l'AI vede l'interfaccia e non ha bisogno di tracciare il grafo interno.

### Feedback loop efficaci

I test si wrappano attorno al confine del modulo e catturano molto comportamento con pochi test. Con shallow modules non è chiaro dove tracciare i confini — si finisce con unit test granulari che testano l'implementazione, non il comportamento.

> "Il soffitto dei tuoi feedback loop è il soffitto dell'AI. Se la tua codebase non ha feedback loop, non otterrai mai output decenti dall'AI." — Matt Pocock

### Gray box mental model

Con i deep modules, il developer può trattare i moduli come **gray boxes**: conosce il comportamento e il contratto dell'interfaccia, non necessariamente ogni dettaglio implementativo. Questo permette di delegare l'implementazione interna all'AI mantenendo la padronanza dell'architettura.

> "With AI you're working harder than ever before, but knowing your codebase less well. Deep modules are my way to retain a sense of the codebase while preserving my sanity." — Matt Pocock

## L'AI produce naturalmente shallow modules

Senza guida esplicita, i coding agent tendono a produrre architetture shallow: molti file piccoli con una responsabilità ciascuno, molte dipendenze tra file. Superficialmente corretto (rispetta SRP), ma difficile da navigare per l'AI nelle sessioni successive e per il developer.

La skill `improve codebase architecture` analizza il repo e propone candidati per approfondire i moduli, trovando cluster di file correlati che potrebbero essere testati come unità con un'interfaccia unica.

## Connessione con Information Hiding

Parnas (information hiding, 1972) e Ousterhout (deep modules) convergono sullo stesso principio — nascondere la complessità implementativa dietro interfacce stabili — con motivazioni diverse:

- **Information hiding**: nascondere ciò che cambia spesso per garantire backwards compatibility
- **Deep modules**: minimizzare il costo cognitivo dell'interfaccia per ridurre la cognitive load (umana e AI)

## Connessioni

- [[concepts/information-hiding]] — stesso principio, motivazione diversa: Parnas nasconde per cambiamento, Ousterhout per cognitive load
- [[concepts/agentic-patterns]] — prerequisito per delegare l'implementazione interna all'AI mantenendo padronanza dell'architettura
- [[patterns/testing-pyramid]] — il confine del modulo è il confine naturale del test di component/integration
- [[concepts/context-management]] — smart zone limitata: deep modules consentono all'AI di capire il sistema con meno token
