# 23785 — Small Corporate: Logica di Implementazione

## Stato Attuale (AS-IS)

```
Partner inserisce promo code manualmente
        ↓
Pagina partner standard (NSA = Non Small Corporate)
        ↓
Scelta prodotto (es. XGEXI)
        ↓
Onboarding parte con il promo code sbagliato
```

**Problema:** Il partner usa il promo code NSA anche quando l'azienda dovrebbe rientrare nel circuito Small Corporate. L'errore avviene perché il partner usa la pagina standard anziché quella Small Corporate.

---

## Nuova Logica (TO-BE)

La modifica introduce una **rilevazione automatica** del promo code corretto, basata su due condizioni verificate in sequenza:

### Condizione 1 — Ricavi dell'azienda

```
Ricavi azienda > 20 milioni?
    ├── SÌ → potenzialmente Small Corporate → vai a Condizione 2
    └── NO → promo code NSA standard (nessun cambio)
```

### Condizione 2 — Convenzione del Partner

```
Il Partner ID ha una convenzione Small Corporate?
    ├── SÌ → sostituisci automaticamente con promo code Small Corporate
    └── NO → promo code NSA standard (nessun cambio)
```

> **Nota critica:** la verifica avviene sul **Partner ID** (identificativo univoco), NON sul Partner Name. Questo suggerisce che uno stesso partner potrebbe avere nomi simili ma ID diversi, e solo alcuni ID hanno la convenzione Small Corporate attiva.

---

## Punto di Intervento nel Flusso

La sostituzione automatica del promo code avviene **al momento della selezione del prodotto**, prima che l'onboarding parta formalmente:

```
Utente arriva alla schermata "Scelta Prodotto"
        ↓
Sistema mostra XGEXI (compatibile con il partner name)
        ↓
[NUOVO] Sistema valuta: ricavi > 20M AND partner ID ha convenzione SC?
        ↓
Sostituisce silenziosamente il promo code con quello Small Corporate
        ↓
Utente seleziona il prodotto (es. XG...)
        ↓
Sistema notifica: "Onboarding avviato con promo code NSA Small Corporate"
        ↓
Comunicazione ad Actico: onboarding Small Corporate iniziato ✓
```

---

## Trasparenza per i vari attori

| Attore | Vede il cambio? |
|--------|----------------|
| Cliente finale | No — completamente trasparente |
| Partner | No — non deve fare nulla di diverso |
| Actico (sistema ricevente) | Sì — riceve la notifica corretta del tipo di onboarding |
| Sistema interno | Sì — logga il promo code effettivamente usato |

---

## Elementi Tecnici da Chiarire

Dalla riunione **mancano dettagli** su:

1. **Dove vive la logica** — lato frontend (al momento della selezione prodotto) o backend (API call)?
2. **Come viene letta la soglia dei 20M** — da anagrafica azienda già caricata, o da input utente?
3. **Come è mappata la convenzione Small Corporate per Partner ID** — tabella dedicata? campo su anagrafica partner?
4. **Feature flag** — la nuova logica va protetta da un feature flag (da richiedere a Francesco), quindi in fase iniziale sarà disattivata e attivabile progressivamente

---

## Timeline

| Milestone | Data |
|-----------|------|
| Pre-produzione | Fine aprile |
| Produzione | 4-5 maggio 2026 |

---

## Partecipanti alla riunione

- Ennio Donatone
- Guglielmo Pelino
- Francesco Mirenda
- Ivano Gentile
- Roberto Perata
