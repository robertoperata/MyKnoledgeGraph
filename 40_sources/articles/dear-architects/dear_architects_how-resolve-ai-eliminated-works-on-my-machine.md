---
tags:
  - reproducible-environments
  - developer-tooling
  - nix
  - devops
  - package-management
feature:
type: article
author: Steve Swoyer
source: https://flox.dev/case-studies/how-resolve-ai-eliminated-works-on-my-machine/
date: 2026-06-14
---

# How Resolve AI Eliminated Works-on-My-Machine

## Sunto

Resolve AI sviluppa sistemi multi-agente autonomi per l'investigazione e la risoluzione di incidenti in produzione, servendo clienti come Coinbase, DoorDash e Salesforce. Prima di adottare Flox, il team affrontava il classico problema della crescita in ambito startup: ogni sviluppatore manteneva la propria configurazione locale, con approcci eterogenei basati su documentazione, script, package manager e conoscenza tacita. Con l'espansione del team, i fallimenti del tipo "funziona sulla mia macchina" divennero endemici: codice che non compilava o non girava su altri sistemi, impossibilità di gestire più versioni di pacchetti in contemporanea.

Resolve AI ha valutato tre categorie di soluzioni: dev container, shell di sviluppo cloud e soluzioni basate su Nix. La scelta è ricaduta su Flox, un package manager e sistema di build riproducibile costruito su Nix open-source. Il punto di forza di Flox rispetto alle alternative è la combinazione di forte riproducibilità con un flusso di lavoro locale normale: a differenza dei container, Flox non isola ermeticamente l'ambiente, ma offre ambienti identici su macOS, Linux, x86 e ARM senza virtualizzazione, con pacchetti nativi della piattaforma.

L'adozione è avvenuta in fasi: prima con gli sviluppatori più interessati per costruire fiducia organica, poi con un'integrazione più ampia. La curva di apprendimento risulta bassa grazie a una configurazione in TOML leggibile e comandi intuitivi (`flox init`, `flox install`, `flox search`, `flox activate`). Dopo pochi mesi, Flox è diventato il riferimento condiviso per la maggior parte del team, con versioni standardizzate dei pacchetti e onboarding semplificato.

Il risultato più significativo è che i problemi "works on my machine" sono scomparsi. I team possono ora distinguere con certezza tra bug effettivi nel codice e artefatti ambientali. Questo è particolarmente critico per Resolve AI: i suoi agenti AI necessitano di dipendenze e input fissi in tutti gli ambienti per rilevare anomalie genuine del sistema anziché rumore derivante da variazioni arbitrarie nell'ambiente.

Il caso Resolve AI illustra un principio architetturale importante: la riproducibilità deterministica dell'ambiente di sviluppo non è solo una comodità operativa, ma un requisito fondamentale per sistemi AI affidabili. Quando la fondazione ambientale è stabile e condivisa, i team possono concentrarsi sulla qualità del codice invece di debugging infrastrutturale, e gli agenti AI possono operare con confidence maggiore.

## Codice

I comandi principali per gestire un ambiente Flox:

```bash
# Inizializzare un nuovo ambiente Flox nella directory corrente
flox init

# Cercare e installare pacchetti
flox search <package-name>
flox install <package-name>

# Attivare l'ambiente (anche automaticamente con direnv)
flox activate
```
