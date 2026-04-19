---
tags:
  - platform-engineering
  - dora-metrics
  - engineering-productivity
  - organizational-culture
  - developer-experience
feature:
type: article
author: Randy Shoup
source: https://www.infoq.com/presentations/platform-engineering-lessons/
date: 2026-04-19
---

# Lessons from the Rise and Fall of eBay Velocity

## Sunto

Randy Shoup racconta in questa presentazione QCon San Francisco 2025 la storia dell'iniziativa "Velocity" di eBay (2020–2025), un programma ambizioso che ha raddoppiato la produttività ingegneristica ma non ha salvato l'azienda dalla stagnazione. Il caso è un esempio potente di come l'eccellenza tecnica non possa compensare la mancanza di allineamento strategico e di cultura organizzativa sana.

I risultati tecnici raggiunti sono stati straordinari: la frequenza di deploy è aumentata di 10x (da 10 giorni a 1–2 giorni), il lead time per il cambio si è ridotto di 5x (da 10 giorni a 2 giorni), e il tasso di fallimento dei cambiamenti è migliorato del 3x. I team eBay sono passati dal 35° al 75° percentile nelle DORA metrics, passando da "medium performer" a "high performer". Sono stati introdotti deployment canary e feature flag su larga scala, cicli commit-to-deploy di un'ora, e la release mobile è passata da mensile a giornaliera grazie alla modernizzazione con SwiftUI e Jetpack Compose.

Nonostante questi successi, l'iniziativa non ha potuto contrastare i fallimenti strutturali dell'organizzazione. Il modello di pianificazione rimasto in stile waterfall — con cicli annuali di più mesi e release massive coordinate tra 50+ team — ha impedito l'adattabilità. La mentalità "feature factory" premia i milestones invece degli outcome per il cliente. Il progetto Managed Payments ha bruciato 1,5 miliardi di dollari in 3 anni con risultati deludenti. eBay ha anche accumulato debito tecnologico evitando cloud pubblico, open source e microservizi fino a tardi, sviluppando fork proprietari di OpenStack e Kubernetes che sono rimasti indietro rispetto al mainline.

La lezione più profonda di Shoup riguarda la cultura: eBay presentava una cultura "patologica" caratterizzata da paura del fallimento, empire-building, sindrome Not-Invented-Here, Faux Agile (sprint waterfall), e controllo top-down con piani immutabili di 12–18 mesi. Il ricercatore Nicole Forsgren (*Accelerate*) aveva già dimostrato che le culture generative predicono il successo organizzativo molto meglio dei soli DORA metrics. L'approccio futuro raccomandato da Shoup richiede tre livelli simultanei: supporto top-down per il backing strategico, engagement bottom-up dagli ingegneri, e connessione middle-out con i peer leader laterali — senza questi tre vettori, qualsiasi trasformazione tecnica rimane incompiuta.
