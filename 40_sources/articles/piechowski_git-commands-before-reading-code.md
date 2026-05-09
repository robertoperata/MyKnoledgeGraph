---
tags:
  - devops
  - developer-experience
type: article
author: Ally Piechowski
source: https://piechowski.io/post/git-commands-before-reading-code
date: 2026-04-08
---

# The Git Commands I Run Before Reading Any Code

## Sunto

L'articolo presenta un approccio diagnostico per comprendere la salute di un codebase prima ancora di leggere una riga di codice: eseguire cinque comandi git che analizzano la storia dei commit. L'autrice sostiene che la storia dei commit rivela pattern di sviluppo, dinamiche del team e aree problematiche in modo più rapido e affidabile di qualsiasi ispezione statica del codice.

Il primo comando identifica i file che cambiano più frequentemente nell'ultimo anno (*file churn*). L'autrice cita uno studio Microsoft Research del 2005 che dimostrò come le metriche basate sul churn predicessero i difetti in modo più affidabile della sola complessità ciclomatica. Un file con **alto churn e alto tasso di bug rappresenta il singolo rischio maggiore** in un progetto.

Il secondo comando (`git shortlog`) rivela chi ha costruito il sistema e in che proporzione. Se una sola persona rappresenta il 60% o più dei commit, il progetto è esposto a un elevato *bus factor*. È importante verificare anche se i top contributor sono ancora attivi nei mesi recenti, per capire se la conoscenza è ancora disponibile.

Il terzo comando filtra i commit con messaggi contenenti termini come "fix", "bug" o "broken" e identifica i file più ricorrenti in questi contesti. Incrociando questo dato con il churn si ottiene una mappa precisa dei punti di maggiore rischio del sistema.

Il quarto comando mostra il numero di commit per mese, rivelando la *velocity* del team nel tempo. Un trend calante può indicare perdita di momentum; picchi isolati suggeriscono rilasci in batch piuttosto che deployment continuo, il che è spesso un segnale di problemi nel processo.

Il quinto e ultimo comando conta revert, hotfix e rollback nell'ultimo anno. Una frequenza elevata di queste operazioni indica problemi profondi: test inaffidabili, pipeline di deploy fragili, o una cultura di sviluppo che preferisce correggere in produzione piuttosto che prevenire. L'insieme dei cinque comandi richiede pochi minuti e fornisce un contesto strategico prezioso prima di immergersi nel codice.

---

## Esempi pratici

### File con maggior churn (ultimi 12 mesi) - cosa è cambiato più spesso

```bash
git log --format=format: --name-only --since="1 year ago" | sort | uniq -c | sort -nr | head -20
```

### Distribuzione dei contributor per numero di commit

```bash
git shortlog -sn --no-merges
```

### File più presenti in commit di bug-fix

```bash
git log -i -E --grep="fix|bug|broken" --name-only --format='' | sort | uniq -c | sort -nr | head -20
```

### Velocity mensile dei commit

```bash
git log --format='%ad' --date=format:'%Y-%m' | sort | uniq -c
```

### Frequenza di revert e hotfix nell'ultimo anno

```bash
git log --oneline --since="1 year ago" | grep -iE 'revert|hotfix|emergency|rollback'
```

---

## Link esterni

- [Your Code as a Crime Scene](https://pragprog.com/titles/atcrime/your-code-as-a-crime-scene/) — libro di Adam Tornhill citato dall'autrice, che tratta l'analisi forense del codice tramite la storia git
- Microsoft Research (2005) — studio citato sull'efficacia delle metriche di churn per predire i difetti (non è fornito un link diretto nell'articolo)

---

## Immagini

Nessuna immagine presente
