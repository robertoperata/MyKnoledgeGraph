---
tags:
  - microservices
  - architecture
  - ddd
  - platform-engineering
  - developer-experience
type: article
author: Vaughn Vernon (host), Christian Deger (guest)
source: https://open.spotify.com/episode/0wJKoSuMloYxqHCeGaOJ5S
source_youtube: https://www.youtube.com/watch?v=tMoX7d9nBjw
date: 2021-10-14
---

# Big Ball of Mud to Microservices — Our Journey

**Podcast:** Add Dot — S01E03  
**Host:** Vaughn Vernon  
**Guest:** Christian Deger — Head of Platform & Architecture, RIO The Logistics Flow (ex Chief Architect Autoscout24)  
**Durata:** 1h 02m  
**Lingua originale:** inglese

---

## Sintesi

Christian Deger racconta in prima persona la trasformazione architetturale di **Autoscout24** (2013–2019): da un monolite .NET su Windows in datacenter a una architettura a microservizi su AWS in Scala, con team DevOps autonomi organizzati secondo il modello "you build it, you run it". Non si trattò di un cambio incrementale ma di un big bang controllato: linguaggio, cloud provider, deployment model e struttura organizzativa cambiarono in un unico progetto multi-anno.

Il cuore della conversazione è il **come** — come si scala la conoscenza in un'organizzazione che cresce, perché Conway's Law torna sempre a fare la differenza, come i **Bounded Context del DDD** (più del termine "microservizio") diventano l'unità organizzativa fondamentale anche per la strategia multi-account AWS, e come il ruolo di Chief Architect non è scrivere codice ma fissare principi, vision e guardrail lasciando ai team la libertà nelle decisioni micro-architetturali.

Nella seconda parte, Christian parla del suo lavoro attuale in RIO: costruire una **Minimal Viable Platform** raccogliendo capabilities dai team product, evitando di gonfiare la piattaforma senza valore reale. Tema trasversale: il rapporto tra monolite e microservizi — entrambi validi se usati nel contesto giusto, con menzione del meetup "Majestic Modular Monoliths" come contrappeso alla retorica pro-microservizi.

---

## Punti chiave

- **Big bang multi-dimensionale ad Autoscout24**: nel 2013-2014 il management decise di cambiare tutto insieme — da Windows/.NET a Linux/JVM (Scala), da datacenter a AWS, da monolite a microservizi, da dev+ops separati a team DevOps "you build it, you run it". In retrospettiva era molto da fare insieme, ma le dipendenze tra i cambiamenti rendevano difficile separarli (es. nessuno voleva gestire Windows su AWS).

- **Strangler Fig come pattern dominante**: ~90% dei servizi estratti dal monolite usarono lo strangler fig pattern. La "definition of done" era: il servizio è in produzione su AWS **e** il codice corrispondente è stato rimosso dal monolite C#. Il challenger più difficile: i PSql trigger Oracle sul ciclo di vita delle inserzioni (processo non documentato, nascosto nel codice database).

- **Scaling della conoscenza: seeding vs boot camp**: inizialmente crescita per seeding (da 1 a 2 team, poi a 4, con metà delle persone esperte che passano al nuovo team). Funziona bene fino a un certo punto, poi il rapporto novizi/esperti diventa insostenibile. Christian preferisce il modello **boot camp**: i nuovi entrano in un team già formato, vengono "assimilati", poi si crea un nuovo team con alcuni dei formati. Più lento ma più efficace.

- **Conway's Law — onnipresente**: "ogni intervista alla fine torna a Conway's Law". La struttura dei team determina l'architettura. La crescita del progetto rese necessario passare da un approccio tacito (i 4 esperti insegnano agli altri naturalmente) a **principi architetturali espliciti** scritti e condivisi, per mantenere l'allineamento con l'aumentare dei team.

- **Scala: vantaggio competitivo e rischio**: scelta controversa ma deliberata — attrarre persone che vogliono lavorare con linguaggi funzionali (anziché tornare a Java). Rischio reale: deriva verso l'astrazione pura (stile Haskell/Cats) invece di spedire codice. Soluzione: linee guida interne ("Scala styleguide"), enfasi sull'idiomatic Scala di "livello medio". Nel nuovo ruolo (RIO) Christian ha scelto Kotlin come compromesso: meno potente di Scala ma meno rischi di "mayhem funzionale".

- **Bounded Context come unità organizzativa reale**: in RIO, Christian ha abbandonato il termine "microservizio" in favore di **Bounded Context**. Ogni Bounded Context ha il suo **AWS account separato** — il confine di account diventa il confine del contesto. Regola fondamentale: i BC non comunicano via rete/queue a livello infrastrutturale direttamente tra loro, solo tramite **API REST o eventi Kafka**. Questo elimina il debate "quanto deve essere micro?" e mette DDD al centro.

- **Context map come primo step**: quando Christian è arrivato in RIO trovò microservizi creati senza pensare al dominio (nomi generici, confini arbitrari). Il primo lavoro fu **reverse engineering della context map** per capire come il dominio era effettivamente organizzato, identificare incongruenze e riposizionare i servizi.

- **Minimal Viable Platform**: in RIO, la piattaforma non deve crescere perché c'è un team che lavora su di essa — deve crescere solo se porta valore. Approccio: **harvesting** — raccogliere capabilities che i team product hanno costruito per sé stessi, trasformarle in offerte di piattaforma proprie, riducendo il cognitive load dei team. Due flavour: Tech Platform (AWS, delivery, monitoring, cost) e Business Platform (capabilities di dominio condivise).

- **Il ruolo del Chief Architect**: tre livelli di architettura — **domain** (context map, DDD), **macro** (standard di interfaccia, sicurezza, AWS guardrail), **micro** (lasciata ai team). Il CA fissa principi e vision, non handhold i team. Esempio di principio: "AWS First, higher-level services preferred" — Fargate sopra EC2, serverless dove possibile, no Kubernetes self-managed (perché ECS Fargate è già a un livello superiore). Tech radar costruito con i lead engineer dei team.

- **Monolite modulare come alternativa valida**: il meetup "Majestic Modular Monoliths" (speaker Axel Fontaine) è citato come contrappunto importante. I microservizi sono la risposta giusta solo se il contesto lo richiede. Un altro meetup mostrava come una startup avesse deliberatamente iniziato con un monolite modulare (conoscendo il dominio ancora bene, volendo ridurre la complessità infrastrutturale) per poi migrare quando il dominio era chiaro e la scalabilità lo richiedeva.

---

## Citazioni notevoli

> "We packed a lot of changes into that project. In hindsight it was a lot of things we did at the same time. There might have been simpler approaches, but all of those changes depended on each other."  
> — Christian Deger (sulla migrazione Autoscout24)

> "The moment you're talking about microservices, the debate is always 'how micro?' Switching to Bounded Context as the unit of granularity removes that debate entirely."  
> — Christian Deger

> "The truth lies in the code, not in the answer. I navigate to repositories to find things out without asking people."  
> — Christian Deger (sulla filosofia del Chief Architect hands-on)

> "Setting up the frame, setting up the vision — and then watching beautiful things happen."  
> — Vaughn Vernon (sulla sintesi del ruolo di Chief Architect)

> "Every interview — it always just boils back down to Conway's Law. People, conversation, communication, collaboration is what goes into architecture."  
> — Vaughn Vernon

---

## Timeline Autoscout24 (ricostruita)

| Anno | Evento |
|---|---|
| 2013 | Workshop con Vaughn Vernon; articolo Martin Fowler sui microservizi pubblicato; decisione di migrare tutto insieme |
| 2014 | Start progetto con 1 team di 8 persone; primo servizio in produzione (strangler); articolo Fowler come "blueprint" della discussione interna; fondazione del Munich Microservices Meetup |
| 2014-2015 | Crescita da 1 a 2 team (seeding); poi a 4 team; introduzione principi architetturali espliciti |
| ~2016+ | Scala styleguide per prevenire deriva funzionale; integrazione progressiva delle operations nei team |
| ~2018-2019 | Christian lascia Autoscout24 (il lavoro sulla Oracle DB per le inserzioni non ancora terminato) |
| Post-2019 | Christian passa a RIO The Logistics Flow come Head of Platform; sceglie Kotlin over Scala |

---

## Riferimenti e risorse

- **Munich Microservices Meetup** — fondato da Christian Deger nel 2014, 61 meetup totali al momento dell'episodio
- **Axel Fontaine** — speaker del meetup "Majestic Modular Monoliths" (caso d'uso pro-monolite modulare)
- **Philip Gabba** — collega di Christian ad Autoscout24, poi product owner nella tech platform di RIO
- **Martin Fowler** — articolo "Microservices" (2014) citato come blueprint della discussione interna ad Autoscout24
- **Vaughn Vernon** — host del podcast, autore di libri DDD (incluso "Strategic Monoliths and Microservices", menzionato nel finale)
- **Stephaniekov** — citato da Christian come fonte della classificazione domain/macro/micro architecture
- **RIO The Logistics Flow** — azienda attuale di Christian (al momento dell'episodio)
- **Autoscout24** — marketplace auto europeo, caso di studio principale dell'episodio
- **Tech Radar** (Thoughtworks) — adottato da Christian in RIO come strumento di governance tech stack condivisa con i lead engineer
