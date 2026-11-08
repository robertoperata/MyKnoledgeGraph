---
tags:
  - software-architecture
  - socio-technical
  - ddd
  - team-topologies
  - ai-agents
feature:
type: article
author: Luca Mezzalira
source: https://www.infoq.com/podcasts/educating-architect-conversation-luca-mezzalira/
date: 2026-11-08
---

# Educating an Architect: A Conversation with Luca Mezzalira

## Sunto

In questo podcast di InfoQ, Michael Stiefel intervista Luca Mezzalira sull'architettura software come disciplina socio-tecnica. Il messaggio centrale è che l'architettura non può essere separata dalla cultura organizzativa e dalle strutture di comunicazione: le scelte tecniche devono rispettare il contesto umano in cui vengono applicate.

Mezzalira illustra questo punto con un esempio vivido: un oratore che consigliava di "tornare al monolite" e che aveva ragione solo dopo aver conosciuto la struttura del team. Questo evidenzia come non esistano best practice universali in architettura — esistono scelte appropriate al contesto. Il compito dell'architetto è capire quel contesto prima di proporre soluzioni.

Un tema ricorrente è il passaggio dall'architetto-guardiano all'architetto-abilitatore. Un solo architetto per ogni team non scala. Mezzalira propone invece un modello basato su guardrail, fitness functions e un processo di "Architecture Advice Process" che permette ai team di prendere decisioni decentralizzate con autonomia controllata. La comunicazione non è più una soft skill opzionale ma una competenza core, soprattutto nell'era del prompting in linguaggio naturale.

L'AI trasforma il ruolo dell'architetto in due modi. Da un lato, con i modelli di subdomain DDD (core, supporting, generic), i team possono calibrare il livello di supervisione umana: nei domini core ad alta volatilità serve presenza umana stretta; nei domini generici (es. integrazione pagamenti standard) l'AI può operare con più autonomia. Dall'altro, Mezzalira sottolinea che la velocità di output AI può sopravanzare la qualità: "speed trumps perfection" è un trade-off reale che ogni team deve gestire deliberatamente.

La questione della crescita dei junior engineer è affrontata con attenzione: il rischio è che l'AI-generated code bypassi il processo di apprendimento. Mezzalira raccomanda pair programming, mob programming e code review condivisi come meccanismi per costruire intuizione anche quando l'AI produce la maggior parte del codice. Infine, sia Stiefel che Mezzalira concordano che gli agenti devono poter essere fermati in qualsiasi momento — il che richiede una governance esplicita, indipendentemente dalla presenza umana nel loop.

## Concetti chiave

- **Sistemi socio-tecnici** e Legge di Conway
- **Enablement teams** (Team Topologies)
- **Architecture Advice Process** con decisioni decentralizzate
- **Architectural fitness functions**
- **DDD subdomains** per calibrare human-in-the-loop oversight
- **Event Storming** e **Wardley Mapping**
- **Hexagonal architecture** (ports and adapters)
- **Resilience patterns**: retry, exponential backoff, graceful degradation
- **AI-DLC** e orchestrazione agentiva nel ciclo di sviluppo
