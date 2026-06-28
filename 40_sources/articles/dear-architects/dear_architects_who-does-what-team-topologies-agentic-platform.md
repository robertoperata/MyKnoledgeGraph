---
tags:
  - team-topologies
  - platform-engineering
  - agentic-ai
  - software-architecture
  - organizational-design
feature:
type: article
author: Olivier Wulveryck
source: https://blog.owulveryck.info/2026/06/24/who-does-what-team-topologies-for-the-agentic-platform.html
date: 2026-06-28
---

# Who Does What? Team Topologies for the Agentic Platform

## Sunto

L'articolo affronta un problema emergente nell'era dell'AI agentiva: man mano che gli agenti AI comprimono la complessità cognitiva sul singolo individuo in archi di tempo brevissimi, le organizzazioni devono ripensare la struttura dei team per distribuire sia il carico di anticipazione (prevedere i fallimenti dell'agente) sia il throughput decisionale. L'autore propone un'estensione del framework Team Topologies applicata specificamente alle piattaforme agentive, definendo chi fa cosa e come i team interagiscono in questo nuovo paradigma.

Il modello distingue quattro tipologie di team adattate al contesto agentivo. I **Stream-Aligned Team** diventano composti da esperti di dominio piuttosto che ingegneri software: guidano l'orchestratore AI e forniscono contesto dinamico (specifiche, guardrail, conoscenza di dominio), delegando alla piattaforma tutte le operazioni. I **Platform Team** offrono quattro pilastri come self-service: contesto globale (system prompt, pattern condivisi), guardrail deterministici (sicurezza, affidabilità, coerenza del brand), tooling agentivo (MCP server, CI/CD, skill condivise) e motore di esecuzione (infrastruttura di inferenza). La loro maturità si misura con lo "self-service index": zero interventi manuali nei deployment.

Gli **Enabling Team** ricoprono un ruolo permanente piuttosto che transitorio, poiché i team di business non acquisiranno mai autonomamente le competenze di ingegneria software. Fanno da ponte tra l'intento di business e il rigore ingegneristico, gestendo il provisioning degli ambienti e identificando cosa non può essere automatizzato. La loro missione è rendersi superflui attraverso il successo: "l'enabling team scompare perché ha avuto successo, non perché ha fallito". I **Complicated Subsystem Team** si occupano di infrastruttura AI profonda (red-teaming, RAG avanzato, fine-tuning, valutazioni custom) e collaborano con il platform team anziché direttamente con i team di prodotto.

Un elemento chiave dell'articolo è la "regola dei tre" applicata alla governance: quando uno stesso guardrail (es. scrubbing PII) compare indipendentemente in tre team distinti, diventa candidato alla sistematizzazione nella piattaforma. Il platform team astrae il guardrail, lo rende configurabile e lo espone globalmente come servizio. Questo processo crea però il "Paradosso del Collo di Bottiglia": il successo genera sovraccarico sul platform owner, che deve automatizzare il rilevamento dei candidati alla sistematizzazione e il tracking del portfolio anziché fare arbitraggio manuale.

L'articolo descrive l'evoluzione verso l'autonomia su due assi paralleli: nella fase iniziale i team scoprono come pacchettizzare il contesto e l'enabling team è embedded; nella fase di maturazione i team padroneggeggiano i fondamentali e la piattaforma espande la copertura dei guardrail; nella fase di piena autonomia l'interazione X-as-a-Service domina e l'enabling team diventa consulenza opzionale. Il principio di governance sottostante è chiaro: "la facilità di andare in produzione deve essere bilanciata dalla facilità di supervisione", prevenendo la "Shadow IT industrializzata" tramite deployment centralizzato e visibilità sistemica.
