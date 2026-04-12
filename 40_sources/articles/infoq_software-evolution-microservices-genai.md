---
tags:
  - architecture
  - microservices
  - legacy-modernization
  - genai
  - evolutionary-architecture
feature:
type: article
author: InfoQ
source: https://www.infoq.com/podcasts/software-evolution-microservices/
---

# Software Evolution: Microservizi, Legacy Modernization e GenAI

Podcast tra **Michael Stiefel** (host) e **Chris Richardson** — creatore di microservices.io, autore di *Microservices Patterns* e fondatore di Eventuate. Il filo conduttore è l'evoluzione dell'architettura software: come i sistemi cambiano nel tempo, perché è difficile farlo, e quale ruolo può avere l'AI.

## Il ruolo dell'architetto

L'architettura emerge da decisioni che nessun altro prende: quelle che non entrano in un use case. Non si può scrivere "rendi questo sistema scalabile" come requisito funzionale. Stiefel sintetizza: tutto ciò che non può essere scritto come requisito funzionale — scalabilità, sicurezza, performance, maintainability — è responsabilità dell'architetto. Se non c'è un responsabile, non viene fatto.

> "I can't define it, but I know it when I see it." — come la pornografia secondo un giudice della Corte Suprema, così è l'architettura.

Richardson aggiunge che oggi, con il fast flow come imperativo, anche **testabilità, maintainability e deployability** sono diventate qualità architetturali di primo livello — non solo proprietà emergenti.

## Perché i sistemi legacy devono evolversi

> "Le aziende hanno già scritto tutto il software di cui hanno bisogno. Devono solo continuare a farlo evolvere."

Le ragioni principali che guidano la modernizzazione:

- **Obsolescenza tecnologica**: sistemi COBOL del 1960 che girano su emulatori cloud, hardware custom non più disponibile
- **Cambiamento dei requisiti non funzionali**: dal batch notturno al real-time 24/7 (esempio: un broker che non può più chiudere le operazioni di notte)
- **Esaurimento del capitale umano**: i developer che conoscono i sistemi storici vanno in pensione

## Dal monolite ai microservizi: il percorso reale

Migrare significa "scolpire pezzi dal monolite e trasformarli in servizi". In pratica:

- Si identifica un modulo che può diventare servizio
- Lo si **disintrica** dal resto del monolite — operazione che può richiedere anni
- Ogni servizio porta la **propria tecnologia stack**: permette di modernizzare incrementalmente con una cost-benefit analysis, invece di un big bang upgrade

## Il problema dei dati è il problema più difficile

La maggior parte delle conversazioni con i clienti finisce sempre sullo stesso punto: *"E i dati?"*

Un modulo non è solo codice — è anche una slice del database. Quando estrai un servizio, fai refactoring di **codice e schema contemporaneamente**, su database aziendali spesso enormi e poco comprensibili.

**Tecnica pratica — repliche read-only transitorie:**
Lasciare le colonne nel database originale come repliche read-only mentre il servizio estratto diventa il nuovo owner, replicando i cambiamenti indietro verso il monolite. Il problema inevitabile è l'**eventual consistency**: il monolite che legge le repliche potrebbe non avere la verità assoluta.

Questa situazione è però temporanea — quando anche il secondo modulo sarà estratto come servizio, le repliche spariranno e si tornerà a comunicare via API. Le "hack" crescono e poi si contraggono man mano che la migrazione avanza.

## Reporting come caso speciale

Dividere i database crea un problema per i report, che richiedono una vista globale. Le opzioni in ordine crescente di allineamento ai principi microservizi:

1. **ETL** diretto dai database dei singoli servizi — funziona ma viola il disaccoppiamento (accesso diretto allo schema altrui è anti-pattern)
2. **Event streaming** — i servizi pubblicano eventi, il data warehouse si aggiorna sottoscrivendosi
3. **Data mesh** — i servizi espongono data product, il reporting avviene tramite quella infrastruttura

## Il rewrite totale è un anti-pattern

Richardson è netto: il "big bang rewrite" è quasi sempre sbagliato.

- Non viene consegnato valore fino al termine (possibilmente anni)
- Le decisioni tecniche non vengono validate finché non si va in produzione
- Più si costruisce su decisioni non validate, più si rischia di costruire il prodotto sbagliato

La filosofia alternativa è il **fast flow**: deploy continui in produzione, feedback rapido dall'ambiente e dagli utenti, molte piccole decisioni invece di poche grandi.

> "Until something is deployed into production, in the hands of users, you do not have any true validation that it is working correctly."

## GenAI per comprendere codebase complesse

Richardson ha usato **Claude Code** per generare ~70 pagine di documentazione di un'architettura su una codebase di ~10.000 righe. Il risultato era utile in parte (sequence diagrams, system operations cross-service) ma con un problema critico: **l'AI aveva inventato nomi di eventi e funzionalità inesistenti**. Solo dopo avergli esplicitamente chiesto di verificare ogni elemento nel codice, ha ammesso le allucinazioni.

La conclusione: l'AI può aiutare a costruire una comprensione di una codebase complessa, ma richiede supervisione attenta. Un workshop di "architettura visibile" con Lego e fili in una conference room resta insostituibile per creare comprensione condivisa e rendere la complessità visibile al management.

## L'AI come architetto: scetticismo costruttivo

I problemi fondamentali che rendono l'AI inadeguata come architetto:

- **Ambiguità dei requisiti**: i requisiti sono intrinsecamente ambigui; l'AI è debole nel gestire l'ambiguità e nel fare le domande giuste per risolverla
- **Non-determinismo**: gli LLM non producono lo stesso output dati gli stessi input — problematico in contesti safety-critical
- **Nessun modello del mondo reale**: gli LLM sono "predittori del prossimo token che non sanno niente e non ragionano" — producono risposte plausibili senza vera comprensione

> "LLMs to me are very strange compared to tooling that we normally use where you know what it's going to do."

## Dove l'AI crea valore oggi

Il caso d'uso più importante e concreto è la **produttività dei developer**. Scrivere codice è un dominio in cui il risultato è testabile — se funziona, funziona. L'architettura invece richiede di validare decisioni per un futuro incerto, non testabile a priori.

## Resilienza come principio architetturale

Il principio unificante di tutto il podcast: **costruire sistemi facili da cambiare**. Testabilità, maintainability e deployability sono la risposta strutturale a un futuro imprevedibile.

I microservizi non servono solo a consegnare software più veloce oggi, ma a garantire che l'architettura possa **evolversi domani** in risposta a cambiamenti tecnologici e di business che oggi non possiamo anticipare. Fare molte piccole decisioni invece di poche grandi riduce il rischio di costruire il prodotto sbagliato nel modo sbagliato.
