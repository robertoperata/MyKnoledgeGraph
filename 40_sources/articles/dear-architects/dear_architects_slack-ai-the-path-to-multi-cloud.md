---
tags:
  - multi-cloud
  - llm-infrastructure
  - mlops
  - circuit-breaker
  - intelligent-routing
  - enterprise-ai
feature:
type: article
author: Shaurya Kethireddy
source: https://slack.engineering/slack-ai-the-path-to-multi-cloud/
date: 2026-08-02
---

# Slack AI: The Path to Multi-Cloud

## Sunto

L'articolo documenta l'evoluzione dell'infrastruttura di Slack per servire Large Language Model a scala enterprise, attraverso quattro fasi architetturali distinte nell'arco di tre anni. Il punto di partenza è stato AWS SageMaker, scelto per le sue garanzie di sicurezza, la conformità FedRamp e il controllo sui modelli. Tuttavia emergono rapidamente limitazioni strutturali: tempi di inizializzazione elevati che limitano la velocità di scaling, scarsa disponibilità di GPU enterprise (A100, H100), e costi elevati per il mantenimento di risorse idle necessarie a rispettare gli SLA di picco.

La seconda fase (metà 2024) vede la migrazione ad **Amazon Bedrock** dopo che quest'ultimo ha ottenuto la conformità FedRamp Moderate. La transizione porta tre vantaggi immediati: semplificazione operativa tramite servizi gestiti, accesso più rapido ai nuovi modelli, ed efficienza infrastrutturale tramite Provisioned Throughput e opzioni On-Demand. La migrazione stessa viene eseguita con "zero incidenti" attraverso quattro passi: approvazioni compliance, load test per determinare le Model Unit necessarie, A/B testing per verificare la parità qualitativa, e shift graduale del traffico con rollback immediato. Permangono però inefficienze: l'over-provisioning per i picchi globali crea capacità idle notturna, e i contratti plurimensili limitano la flessibilità di migrazione verso modelli superiori.

La terza fase risolve il problema delle risorse idle spostando i workload asincroni e a burst verso l'infrastruttura **On-Demand**, mantenendo le feature latency-sensitive su Provisioned Throughput. Un pattern di spillover instrada automaticamente le richieste in eccesso verso endpoint on-demand. Viene introdotta anche una gerarchia di modelli per il fallback automatico in caso di degradazione del provider principale. Questa fase evidenzia due gap strategici residui: vulnerabilità agli outage a livello di provider e limitazione nell'accesso a modelli disponibili esclusivamente su specifici cloud.

La quarta fase (inizio 2026) vede l'espansione verso **Google Cloud Platform Vertex AI**, concretizzando una strategia multi-cloud. I quattro driver strategici sono: ridondanza infrastrutturale (eliminando il provider singolo come single point of failure), ottimizzazione modello-per-feature (con miglioramenti qualitativi ~10% per task di ragionamento complesso e riduzione della latenza ~67% per workload ad alta velocità), accesso all'innovazione indipendente dal provider, e orchestrazione dinamica dei workload basata su telemetria in tempo reale. Il cuore della soluzione è uno **strato di routing intelligente** con tre componenti: selezione del modello guidata da metriche di qualità interne, regole sperimentali e A/B testing per validare rapidamente nuovi LLM post-compliance, e circuit breaker automatici con monitoraggio dell'health degli endpoint (TTFT, error rate, percentili di latenza).

Le riflessioni conclusive dell'articolo sintetizzano le lezioni apprese: lo scaling sicuro richiede allineamento cross-funzionale tra Legal, Risk, Compliance e Security; lo strato di astrazione è più strategico del modello specifico scelto; l'architettura richiede evoluzione continua dato il ritmo dei managed service; la ridondanza multi-provider previene interruzioni di piattaforma che i failover single-provider non possono evitare; e la degradazione delle performance deve essere trattata come un "soft failure" — un servizio operativo ma lento è effettivamente rotto.
