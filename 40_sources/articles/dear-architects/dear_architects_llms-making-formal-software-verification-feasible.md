---
tags:
  - formal-verification
  - software-security
  - llm-research
  - bug-finding
  - program-correctness
feature:
type: article
author: Keane Lucas et al. (Anthropic)
source: https://www.anthropic.com/research/formal-verification
date: 2026-10-25
---

# LLMs are Making Formal Software Verification Feasible at a Much Larger Scale

## Sunto

Anthropic, in collaborazione con il team di Theorem, ha utilizzato Claude per applicare la **verifica formale del software a scala industriale**, dimostrando che ciò che richiedeva anni di lavoro da parte di esperti può ora essere automatizzato con modelli linguistici. L'esperimento principale ha dimostrato la proprietà "no-crash" (assenza di crash) per 239 entry point distribuiti su 123 librerie open source, lavorando a livello di assembly RISC-V a partire da fuzz test esistenti — non da specifiche formali scritte manualmente.

Un risultato particolarmente significativo è la capacità di Claude di **trovare e correggere bug** che avevano eluso il fuzzing per anni. Un esempio concreto: un counter underflow in un decompressore che si manifestava solo su input rari intorno ai 20 MB — un caso tipicamente impossibile da raggiungere con fuzzing statistico ma rilevabile attraverso ragionamento formale. Altri casi di studio includono: la verifica di zlib rispetto all'RFC 1950, un interprete Python in Rust controllato contro il comportamento di CPython (identità al 99,95% su 1,7 milioni di programmi casuali), e un lettore PDF dove numerosi security bug sono stati identificati in una settimana.

I costi rimangono elevati: alcune proof richiedono centinaia di milioni di token. Tuttavia, il ricercatori evidenziano due fattori di ottimizzazione: i modelli più recenti richiedono meno token per lo stesso task, e le **proof riusabili** riducono il consumo del 17% attraverso la condivisione di lemmi tra entry point diversi. Il parallel scaling delle prove su workspaces condivisi è uno degli approcci tecnici più promettenti per ridurre i costi.

Le limitazioni sono importanti da comprendere: un programma formalmente verificato non è necessariamente privo di bug. La validità di una proof dipende dall'accuratezza della specifica e del modello della macchina. L'approccio attuale copre binari RISC-V statici single-thread su musl libc — non il deployment tipico su x86/ARM con glibc in ambienti multi-thread. Qualsiasi assunzione errata nella specifica invalida la garanzia fornita dalla proof.

La conclusione degli autori è ambiziosa ma motivata dai dati: **la verifica formale diventerà routine come il fuzzing** per la maggior parte del software in produzione. Se questa previsione è corretta, le implicazioni per la sicurezza del software sono profonde — in particolare per librerie critiche, componenti di sicurezza e firmware, dove il costo di un bug è molto superiore al costo della verifica.

## Immagini

![Immagine hero della ricerca sulla verifica formale](https://www-cdn.anthropic.com/images/4zrzovbb/website/62ef4c44cbeeaca254eadaaaf982d1d32a7e9819-2000x1125.jpg)

![Grafico del budget token per la verifica](https://www-cdn.anthropic.com/images/4zrzovbb/website/02207cbc6f46e272fe9a0fa7929cb8ec29578e74-1920x980.png)

![Grafico animato delle proof su workspace condiviso](https://www-cdn.anthropic.com/images/4zrzovbb/website/5078468fa7e09774b1ab6e86984ec5691d3d11fa-1080x712.gif)

![Grafico del riuso delle proof](https://www-cdn.anthropic.com/images/4zrzovbb/website/8e295614d599d548fbd2ff97ed3cc3e62f501b2b-1920x1248.png)
