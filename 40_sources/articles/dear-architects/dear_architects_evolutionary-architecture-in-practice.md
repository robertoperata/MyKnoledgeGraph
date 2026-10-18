---
tags:
  - evolutionary-architecture
  - fitness-functions
  - architectural-patterns
  - ci-cd
  - software-quality
feature:
type: article
author: InfoQ
source: https://www.infoq.com/minibooks/evolutionary-architecture
date: 2026-10-18
---

# Evolutionary Architecture in Practice

## Sunto

Questo eMag di InfoQ (ottobre 2026), curato da Luca Mezzalira, raccoglie sei articoli scritti da partecipanti dell'InfoQ Certified Architect Program. Il filo conduttore è che l'architettura evolutiva non consiste semplicemente nel "rendere il cambiamento più facile", ma nel mantenerlo **sicuro, chiaro e deliberato** di fronte al movimento tecnologico e organizzativo costante. Il modello proposto è quello di un sistema di feedback continuo: le fitness function segnalano quando il sistema devia, e la comprensione (comprehension) conferma che le persone capiscono ancora cosa stanno modificando.

Il primo articolo introduce le **Agentic Fitness Functions**: estendono le fitness function tradizionali con un layer di giudizio AI che verifica la deriva semantica e il rischio architetturale in CI/CD. L'idea è che i check automatici non debbano limitarsi a metriche strutturali (dipendenze, copertura dei test), ma possano valutare se il comportamento del sistema rimane coerente con le intenzioni architetturali originali.

Il secondo articolo tratta la **comprensione come caratteristica architetturale**: la comprensione umana del sistema diventa una proprietà da misurare e preservare, tanto quanto la scalabilità o la resilienza. Nel contesto del codice generato da AI, questo è particolarmente urgente: il codice sintetico può erodere la comprensione collettiva del team, rendendo più difficile il ragionamento su cambiamenti futuri.

Tre articoli approfondiscono l'applicazione pratica: evoluzione di una piattaforma B2C verso B2B (gestione di schema changes e boundary refactoring), architetture multi-cloud e multi-region con fitness function per bilanciare autonomia regionale e requisiti centrali di sicurezza e compliance, e infine la trasformazione di un prototipo AI-assisted in un sistema distribuito senza riscrittura completa (pattern "vibed prototype").

L'ultimo articolo usa la **Inverse Conway Maneuver** per federare la ownership del sistema: strutturare i team secondo l'architettura desiderata, combinando zero-trust security, platform engineering e micro-frontend per supportare l'evoluzione autonoma dei bounded context.
