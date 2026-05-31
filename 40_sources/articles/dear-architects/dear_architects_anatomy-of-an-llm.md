---
tags:
  - llm
  - transformers
  - attention-mechanism
  - embeddings
  - machine-learning
feature:
type: article
author: Roy van Rijn
source: https://www.royvanrijn.com/anatomy-of-an-llm/
date: 2026-05-31
---

# The Anatomy of an LLM

## Sunto

L'articolo di Roy van Rijn è una guida visuale e interattiva completa che percorre, passo per passo, ogni operazione matriciale che avviene all'interno di un LLM durante l'inferenza. Il percorso completo è: testo → token → vettori → blocchi transformer → logits → campionamento → output. Ogni capitolo affronta un componente specifico con visualizzazioni interattive e guide alle dimensioni dei tensori, rendendo tangibili concetti che spesso restano astratti.

La **tokenizzazione** (Cap. 1) converte il testo in ID di token: un token può essere una parola intera, parte di una parola, punteggiatura o un numero. Gli **embedding vettoriali** (Cap. 2) mappano ciascun ID token su un vettore appreso tramite una lookup table: "Un embedding è la rappresentazione iniziale del token, non il suo significato finale." Le dimensioni tipiche vanno da 768 a 3072+. Le **funzioni di attivazione** non lineari (Cap. 3) permettono alle reti di modellare pattern complessi, mentre le **reti feed-forward** (Cap. 4) trasformano l'informazione posizione per posizione, a differenza dell'attenzione che opera tra posizioni diverse.

Il meccanismo di **attenzione** (Cap. 8-10) è il cuore dell'architettura: il framework Q/K/V distingue le query (cosa il token cerca), le chiavi (contenuto disponibile) e i valori (informazione da trasportare). La **multi-head attention** esegue più pattern di attenzione in parallelo, catturando relazioni diverse. Le **RoPE** (Rotary Positional Embeddings) iniettano la consapevolezza posizionale ruotando i vettori query e key, mantenendo le proprietà di relatività posizionale durante l'inferenza.

Il **blocco transformer** completo (Cap. 11) combina normalizzazione, grouped-query attention, connessioni residuali e reti feed-forward. I **logits e il campionamento** (Cap. 5) mostrano come il modello produca distribuzioni di probabilità sul vocabolario e come strategie diverse (greedy, temperatura, top-k, nucleus sampling) determinino la selezione del token successivo. Il **KV cache** (Cap. 14) durante la generazione evita il ricalcolo dell'attenzione sui token precedenti, riducendo drasticamente il compute dell'inferenza.

Le **fasi di addestramento** distinguono tra pre-training (ottimizzazione della predizione del token successivo su corpus massivi, che insegna le capacità generali) e post-training (instruction tuning e preference optimization che plasmano il comportamento in assistente). La **quantizzazione** (Cap. 15) riduce i requisiti di memoria rappresentando i pesi con meno bit: "La quantizzazione è approssimazione controllata. Riduce la memoria e spesso migliora l'inferenza pratica." L'articolo è una risorsa fondamentale per chiunque voglia capire cosa succede davvero tra prompt e risposta, a livello di algebra lineare.
