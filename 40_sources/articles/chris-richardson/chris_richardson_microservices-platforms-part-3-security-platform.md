---
tags:
  - microservices
  - team-topologies
  - platform-engineering
  - security
  - oauth2
  - authentication
  - authorization
feature:
type: article
author: Chris Richardson
source: https://microservices.io/post/architecture/2026/01/30/qconsf-microservices-platforms-part-3.html
date: 2026-05-03
---

# Microservices Platforms - part 3: Security Platform

## Sunto

Questo articolo è il terzo di una serie basata sul talk tenuto da Chris Richardson al QCon San Francisco 2025, intitolato "Microservices Platforms: When Team Topologies Meets Microservices Patterns". Esamina il secondo layer della piattaforma: la **Security Platform**, il cui scopo è semplificare lo sviluppo di microservizi sicuri.

Il problema di fondo è che la sicurezza nei sistemi a microservizi è complessa e pervasiva: autenticazione delle identità degli utenti, autorizzazione inter-servizio, gestione dei segreti, comunicazioni cifrate via TLS, audit logging. Se ogni team applicativo deve implementare e mantenere questi meccanismi autonomamente, il risultato è inevitabilmente inconsistenza, errori e vulnerabilità.

La **Security Platform** astrae queste responsabilità, fornendo ai team applicativi capacità di sicurezza pronte all'uso. Tipicamente include:

- Un **Identity Provider** centralizzato (es. Keycloak, Auth0) che gestisce autenticazione e rilascio di token, spesso tramite OAuth 2.0 / OIDC
- Meccanismi di **service-to-service authentication** che garantiscono che solo i servizi autorizzati possano comunicare tra loro (es. mTLS tramite service mesh, JWT con firma condivisa)
- Integrazione con un sistema di **gestione dei segreti** (es. HashiCorp Vault) per distribuire credenziali in modo sicuro senza che appaiano in chiaro nei repository o nelle variabili d'ambiente
- **Policy di autorizzazione** centralizzate, in modo che le regole di accesso siano consistenti tra tutti i servizi

Seguendo il modello delle **Team Topologies**, il Platform Team che gestisce la Security Platform offre queste capacità come servizi interni consumabili dai team applicativi. Un team che crea un nuovo servizio non deve preoccuparsi di come configurare TLS, come validare i token JWT o come connettersi all'identity provider: ci pensa la piattaforma. Questo abbassa significativamente la barriera per sviluppare servizi sicuri by default, riducendo al contempo la superficie di attacco generata da implementazioni eterogenee.
