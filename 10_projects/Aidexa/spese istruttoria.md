# Spese di Istruttoria — Gestione Scontistica

## Contesto

L'introduzione delle **spese di istruttoria differenziate per fascia di importo** pone un problema: come gestire l'applicazione dei **codici sconto (promo code)** in combinazione con le logiche di calcolo di Actico, che determina autonomamente importo e durata ottimale del finanziamento.

---

## Le due soluzioni a confronto

### Soluzione A — Servizio Stateless (soluzione "pulita")
Un nuovo microservizio centralizzato e stateless dedicato al calcolo delle spese di istruttoria.

**Input**: `importo`, `durata`, `codice sconto`, `codice prodotto`  
**Output**: spese di istruttoria (con o senza sconto applicato)  
**Consumatori**: Actico, Onboarding, Back Office — chiunque debba calcolare l'istruttoria

**Pro**:
- Nessuna duplicazione di logica tra Actico e altri sistemi
- Actico diventa semplice consumatore del servizio
- Estendibile in futuro ad altri tipi di spesa

**Contro**:
- Impatto su Actico: le 3 LPI coinvolte (pricing, LPI 8, LPI 13, LPI 21) devono essere aggiornate
- Il codice sconto deve essere passato ad Actico nel contratto di chiamata (cambio contratti)
- Richiede tempo di sviluppo non banale, soprattutto se si vuole costruire qualcosa di solido e non una "macchia isolata"

---

### Soluzione B — Workaround (soluzione "zozza")
Onboarding calcola in autonomia la spesa di istruttoria corretta (con sconto) e la passa ad Actico come override, in continuità con la logica attuale.

**Pro**:
- Sviluppo più rapido
- Minimo impatto su Actico

**Contro / Problema critico**:
- Se il cliente inserisce un codice sconto e **poi cambia la durata**, Actico ricalcola l'importo sostenibile sulla base della nuova durata. L'importo può scendere, cadere in una fascia diversa → la base dell'istruttoria cambia → lo sconto applicato in precedenza è disallineato.
- Per gestire correttamente questo caso servirebbero: una doppia chiamata, la cancellazione dello sconto a ogni cambio di configurazione, e una logica UX complessa.
- Il loophole esiste ma dipende da quanto è frequente il caso: *inserire un codice sconto + cambiare la durata*.

---

## Comportamento attuale in produzione
Quando viene inserito un codice sconto, l'onboarding calcola l'istruttoria scontata sulla base della **durata selezionata** e la passa ad Actico. Il codice sconto salva una differenza percentuale rispetto all'istruttoria di prodotto, non un valore assoluto.

---

## Decisioni prese / accordi raggiunti

- La soluzione con Actico che calcola tutto in autonomia (senza questo servizio) **non è sul tavolo**: già esclusa.
- Il servizio stateless, se implementato, **non deve rimanere un componente isolato**: va inserito in una roadmap più ampia (iniziativa su DevOps).
- Vanno prodotti **sequence diagram** per i 3 flussi Actico impattati.

## Prossimi passi

| # | Azione | Owner |
|---|--------|-------|
| 1 | Scrivere e stimare entrambe le soluzioni (workaround vs. stateless service) | Ivano |
| 2 | Allineamento tecnico di dettaglio | Ivano + Guglielmo (30 min, settimana prossima) |
| 3 | Creare iniziativa su DevOps per evoluzione del servizio stateless | TBD |
| 4 | Produrre sequence diagram per LPI 8, 13, 21 di Actico | TBD |

## Criterio di scelta finale
> Se il workaround costa significativamente meno e copre le casistiche reali → si va sul workaround.  
> Se la differenza di effort è minima → si va sulla soluzione pulita.
