---
tags:
  - ai-agents
  - software-development
  - llm-engineering
  - programmazione
  - automazione
  - influxdb
feature:
type: article
author: Paul Dix
source: https://pauldix.com/the-end-of-programming
date: 2026-08-30
---

# The End of Programming

## Sunto

Paul Dix, fondatore e CTO di InfluxData e creatore di InfluxDB, argomenta che la scrittura manuale di codice e la revisione umana riga per riga si stanno avvicinando all'obsolescenza. La tesi non è la fine della creazione di software, bensì la fine di come la programmazione è stata tradizionalmente praticata: gli agenti AI produrranno la maggior parte del software funzionale, con gli esseri umani che revisionano solo gli output finali piuttosto che le singole righe di codice. Il punto di inflessione si raggiunge quando il software prodotto dall'AI supera quello scritto dagli umani in termini di volume totale — e secondo Dix siamo già lì.

A sostegno della tesi, Dix cita dati concreti: un'impennata esponenziale nei commit di codice dal 2025 visibile nell'analisi del down di GitHub del 17 agosto, e il caso studio di Bun 1.4. In quest'ultimo, Jarred Sumner ha riscritto oltre 1 milione di righe da Zig a Rust in 11 giorni, dirigendo agenti pre-release di Fable 5 che lavoravano in parallelo, generando 6.778 commit a un costo di circa $165.000 in API. La cosa più significativa, sottolinea Dix, non è la velocità: "l'AI ha scritto 1M di righe di codice e le ha raffinate per mesi per produrre software affidabile in esecuzione su milioni di macchine di sviluppatori."

Dix racconta anche la propria esperienza diretta con due progetti significativi. Il primo riguarda l'integrazione Iceberg per InfluxDB (abilitazione dell'API REST Iceberg, integrazione del compactor per la creazione di manifest e Parquet, implementazione S3 API): implementazione funzionante in 14 ore, verificata con client DuckDB e PyIceberg. Il secondo è un sistema di edge data replication con nodi InfluxDB satellite, pipeline di compressione/filtraggio, API di accesso al catalogo e observability via metriche e system table: sistema end-to-end funzionante in 28 ore, deployato in infrastruttura AWS di test. Entrambi i progetti operavano entro i limiti di allocazione settimanale di Fable — con accesso illimitato ai token, i cicli di miglioramento continuo 24/7 sarebbero economicamente impraticabili "a centinaia di migliaia di spesa mensile".

L'articolo delinea anche una traiettoria temporale dei modelli: Opus 4.5 e GPT-5.2 (fine 2025) hanno già causato preoccupazione significativa ai dirigenti; Fable 5 rappresenta una soglia di capacità maggiore; Dix prevede modelli di capacità equivalente a costi inferiori entro fine anno o inizio 2027, con intelligenza frontier "abbastanza economica" per uso standard entro 2-3 anni e velocità di generazione di token migliorate di 10-100x. La realtà organizzativa, però, è che la maggior parte delle aziende continuerà con le pratiche tradizionali per "inerzia organizzativa", mentre i principali creatori di software adotteranno i paradigmi di sviluppo diretto dall'AI.
