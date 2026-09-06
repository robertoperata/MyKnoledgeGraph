---
tags:
  - archunit
  - architecture-testing
  - java
  - netflix
  - gradle
feature:
type: article
author: Dear Architects
source: https://netflixtechblog.com/scaling-archunit-with-nebula-archrules-b4642c464c5a
date: 2026-09-06
---

# Scaling ArchUnit with Nebula ArchRules

## Sunto

Il Netflix Tech Blog descrive come il team **JVM Ecosystem** di Netflix abbia risolto il problema di applicare regole architetturali in modo consistente attraverso decine di migliaia di repository Java, nel contesto di una strategia **polyrepo** — una delle più grandi al mondo per dimensioni. La sfida principale è che in un contesto polyrepo di questa scala, le regole architetturali tendono a restare confinanti in pagine wiki che nessuno legge, invece di essere enforcement automatico nel processo di build.

La soluzione adottata combina due strumenti complementari: **ArchUnit**, un framework Java per testare le regole architetturali tramite unit test (verifica dipendenze, layer separation, naming conventions, e pattern strutturali del codice), e **Nebula**, la suite di Gradle plugin sviluppata internamente da Netflix per standardizzare il processo di build, mantenere le dipendenze aggiornate e pubblicare artefatti in modo affidabile attraverso l'intero ecosistema Java. L'integrazione dei due ha dato vita a **Nebula ArchRules**: un meccanismo per distribuire e applicare regole ArchUnit condivise a livello di piattaforma, senza che ogni singolo team debba definirle o mantenerle autonomamente.

Il vantaggio architetturale fondamentale di questo approccio è la centralizzazione della governance: le regole architetturali vengono definite una volta dal team di piattaforma, distribuite come parte della toolchain Gradle, e applicate automaticamente in ogni repository che adotta Nebula. Questo garantisce che le decisioni architetturali critiche — separazione dei layer, dipendenze proibite, pattern di naming — siano enforcement obbligatorio piuttosto che linee guida opzionali.

Il pattern descritto è particolarmente rilevante per organizzazioni che operano con molti team autonomi: la governance architetturale distribuita via tooling è molto più efficace della governance via documentazione. ArchUnit consente di esprimere le regole come codice Java testabile, con messaggi di errore chiari e diagnostica precisa in caso di violazione. Nebula garantisce la propagazione di queste regole attraverso tutta la flotta di repository senza friction per i team consumer, che ereditano automaticamente gli standard di piattaforma tramite il plugin Gradle.
