---
title: Legacy Modernization
type: concept
tags: [thread1-microservices]
sources:
  - "[[infoq_software-evolution-microservices-genai]]"
updated: 2026-04-17
related:
  - "[[concepts/independent-deployability]]"
  - "[[concepts/bounded-context]]"
  - "[[patterns/event-driven]]"
  - "[[patterns/cqrs-read-model]]"
---

# Legacy Modernization

## Definizione

Il processo di evoluzione incrementale di sistemi software esistenti per adattarli a nuovi requisiti tecnologici, di business o non funzionali. Il principio guida è il **fast flow**: deploy continui, feedback rapido, molte piccole decisioni invece di poche grandi.

> "Le aziende hanno già scritto tutto il software di cui hanno bisogno. Devono solo continuare a farlo evolvere."
> — Chris Richardson

## Perché i sistemi legacy devono evolversi

1. **Obsolescenza tecnologica** — hardware custom non più disponibile, sistemi COBOL su emulatori cloud
2. **Cambiamento dei requisiti non funzionali** — dal batch notturno al real-time 24/7
3. **Esaurimento del capitale umano** — i developer che conoscono i sistemi storici vanno in pensione

## Anti-pattern: Big Bang Rewrite

Il rewrite totale è quasi sempre sbagliato:
- Non viene consegnato valore fino al termine (possibilmente anni)
- Le decisioni tecniche non sono validate finché non si va in produzione
- Più si costruisce su decisioni non validate, più si rischia di costruire il prodotto sbagliato

> "Until something is deployed into production, in the hands of users, you do not have any true validation that it is working correctly."

## Pattern: Strangler Fig (estrazione incrementale)

"Scolpire pezzi dal monolite e trasformarli in servizi":
1. Identificare un modulo che può diventare servizio
2. Disintricarlo dal resto — può richiedere mesi/anni
3. Ogni servizio porta la propria tecnologia stack: modernizzazione incrementale con cost-benefit analysis

## Il problema dei dati

Estrarre un servizio significa fare refactoring di codice **e** schema contemporaneamente. Tecnica:

**Repliche read-only transitorie:**
- Le colonne rimangono nel DB originale come repliche read-only
- Il servizio estratto diventa il nuovo owner, replica i cambiamenti indietro
- Problema inevitabile: eventual consistency per il monolite che legge le repliche
- Situazione temporanea: quando anche il secondo modulo sarà estratto, le repliche spariscono

## Reporting in sistemi decomposed

| Approccio | Note |
|---|---|
| ETL diretto dai DB dei servizi | Funziona ma viola il disaccoppiamento (accesso diretto allo schema altrui è anti-pattern) |
| Event streaming | I servizi pubblicano eventi, il data warehouse si aggiorna sottoscrivendosi |
| Data mesh | I servizi espongono data product, il reporting avviene tramite quella infrastruttura |

## Qualità architetturali nel legacy moderno

Con il fast flow come imperativo, **testabilità, maintainability e deployability** sono diventate qualità architetturali di primo livello — responsabilità esplicita dell'architetto, non proprietà emergenti.

## AI per comprendere codebase complesse

L'AI (es. Claude Code) può generare documentazione e sequence diagram di codebase complesse, ma richiede supervisione: tende ad allucinare nomi di eventi e funzionalità inesistenti. Usare con verifica esplicita ("verifica ogni elemento nel codice").

## Connessioni

- [[concepts/independent-deployability]] — obiettivo finale della modernizzazione incrementale
- [[concepts/bounded-context]] — guida la decomposizione del monolite in servizi
- [[patterns/event-driven]] — event streaming come alternativa a ETL per il reporting
- [[patterns/cqrs-read-model]] — pattern per gestire la proiezione dei dati dopo la decomposizione
- [[patterns/anti-corruption-layer]] — abilitatore chiave dello Strangler Fig: permette al nuovo sistema di coesistere col legacy senza adottarne il modello durante le fasi intermedie di migrazione
- [[patterns/modular-monolith]] — step intermedio naturale nello spettro prima di estrarre servizi: i moduli già ben delimitati rendono l'estrazione con Strangler Fig un'operazione chirurgica invece di un rewrite
