---
tags:
  - ai-agents
  - security
  - github-actions
  - agentic-workflows
  - prompt-injection
  - ci-cd
feature:
type: article
author: Landon Cox, Jiaxiao Zhou
source: https://github.blog/ai-and-ml/generative-ai/under-the-hood-security-architecture-of-github-agentic-workflows/
date: 2026-05-17
---

# Under the Hood: Security Architecture of GitHub Agentic Workflows

## Sunto

GitHub ha sviluppato un'architettura di sicurezza a più livelli per consentire agli agenti AI di operare in modo autonomo all'interno delle pipeline CI/CD senza compromettere la sicurezza dei segreti, la qualità del codice o la supervisione umana. Il modello di minaccia parte dall'assunzione che gli agenti siano fondamentalmente non deterministici: devono consumare input non fidati, ragionare sullo stato del repository e prendere decisioni a runtime, rendendo impossibile garantire comportamenti corretti con semplici politiche statiche.

L'architettura si articola in tre livelli sovrapposti. Il livello **Substrate** sfrutta le VM dei runner di GitHub Actions e container trusted per garantire isolamento dei componenti, mediazione delle operazioni privilegiate e confini di comunicazione applicati dal kernel. Il livello **Configuration** gestisce artefatti dichiarativi che controllano il caricamento dei componenti, i canali di comunicazione, l'assegnazione dei permessi e la distribuzione dei token. Il livello **Planning** crea workflow staged con scambi di dati espliciti, incluso un sottosistema di "safe outputs" per la gestione controllata delle scritture.

Il principio "zero-secret agent" è il cuore della sicurezza del sistema. Piuttosto che esporre variabili d'ambiente condivise all'agente (vettore classico di prompt injection con esfiltrazione di credenziali), GitHub isola l'agente in un container dedicato con egress controllato. I token LLM vengono instradati attraverso un proxy API isolato, mentre un MCP gateway in un container trusted separato gestisce l'autenticazione. L'agente vive in una `chroot` jail con mount selettivi che limitano la superficie scrivibile.

Tutte le scritture verso GitHub passano attraverso una pipeline di vetting in tre fasi: filtraggio delle operazioni consentite, limitazione del volume massimo di aggiornamenti per esecuzione, e analisi del contenuto che rimuove pattern indesiderati, URL e segreti. Questo approccio "stage and vet all writes" garantisce che nessuna modifica non revisionata raggiungi il repository, trattando l'esecuzione agentiva come un'estensione del modello CI/CD piuttosto che come un runtime separato.

La registrazione sistematica a tutti i confini di fiducia completa il quadro: il firewall registra l'attività di rete, il proxy API cattura i metadati delle richieste al modello, il gateway MCP registra le invocazioni degli strumenti, e la strumentazione dei container audita le azioni sensibili. Questo supporta la ricostruzione forense degli incidenti e il rilevamento di anomalie comportamentali.

## Immagini

![Architettura a tre livelli: Planning, Configuration e Substrate con i componenti di sicurezza](https://github.blog/wp-content/uploads/2026/03/figure1-layered-architecture.png)

![Isolamento dei container: agent container, firewall, MCP gateway e API proxy](https://github.blog/wp-content/uploads/2026/03/figure2-container-isolation.png)

![Pipeline di vetting delle scritture: dal Safe Outputs MCP alle analisi deterministiche](https://github.blog/wp-content/uploads/2026/03/figure3-write-staging-pipeline.png)
