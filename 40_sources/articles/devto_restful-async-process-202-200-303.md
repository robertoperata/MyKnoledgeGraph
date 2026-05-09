---
tags:
  - api
  - architecture
  - microservices
type: article
author: Vinodh Thiagarajan
source: https://dev.to/vinodhthiagarajan1309/dial-202-200-303-for-restful-asynchronous-process-3mph
date: 2020-03-03
---

# Dial 202–200–303 for RESTful Asynchronous Process

## Sunto

L'articolo descrive un pattern pratico per gestire **processi long-running avviati via REST API**, usando la sequenza di HTTP status code 202 → 200 (polling) → 303. Il caso d'uso è semplice e concreto: conversione di un file MP4 in MP3 tramite upload — un'operazione che può richiedere decine di secondi o minuti, incompatibile con una risposta sincrona HTTP.

Il pattern si articola in quattro passi. Il client carica il file su cloud storage (S3, Dropbox, Google Drive) ottenendo un webhook o URL. Chiama poi il servizio REST con una POST, che risponde **202 Accepted** (non 200!) indicando che il processo è stato accettato ma non ancora completato. L'header `Location` nel 202 contiene l'URL del *job resource* che il client può interrogare via polling.

Il client effettua GET periodiche sul job resource, ricevendo **200 OK** con un body JSON che descrive lo stato corrente (percentuale completamento, tempo stimato, frequenza di polling consigliata, eventuale errore). Questo stesso endpoint restituisce 200 sia in caso di progresso normale sia in caso di errore — la distinzione avviene nel body, non nello status code.

Quando il processo termina con successo, la GET sul job resource restituisce **303 See Other**, redirigendo il client alla risorsa finale (es. la pagina di download) o a una schermata di completamento. Il 303 funge da segnale semantico di "processo completato — vai qui per il risultato".

---

## Il Pattern: sequenza HTTP completa

```
Client                          Server
  │                               │
  │── POST /converter/mp4tomp3 ──>│
  │<── 202 Accepted               │
  │    Location: /converter/mp4tomp3/48578
  │                               │
  │── GET /converter/mp4tomp3/48578 ──>│  (polling ogni N secondi)
  │<── 200 OK                     │
  │    { "percentage_completed": "18%",
  │      "estimated_time_to_complete": "75 seconds",
  │      "poll_every": "5 seconds",
  │      "error": "" }
  │                               │
  │── GET /converter/mp4tomp3/48578 ──>│  (processo completato)
  │<── 303 See Other              │
  │    Location: /downloads/result/48578
  │                               │
  │── GET /downloads/result/48578 ──>│
  │<── 200 OK (pagina di download)│
```

---

## Esempi pratici

### Step 2 — POST che avvia il processo

```http
POST https://converteranything.com/converter/mp4tomp3

HTTP/1.1 202 Accepted
Location: https://converteranything.com/converter/mp4tomp3/48578
```

`48578` può essere un job ID generato dal sistema o un OS process ID. La risorsa `/mp4tomp3` è un *controller resource* — una convenzione non strettamente RESTful ma pragmatica.

### Step 3 — GET di polling (processo in corso)

```http
GET https://converteranything.com/converter/mp4tomp3/48578

HTTP/1.1 200 OK
{
  "percentage_completed": "18%",
  "estimated_time_to_complete": "75 seconds",
  "poll_every": "5 seconds",
  "error": ""
}
```

### Step 3 — GET di polling (errore)

```http
GET https://converteranything.com/converter/mp4tomp3/48578

HTTP/1.1 200 OK
{
  "percentage_completed": "",
  "estimated_time_to_complete": "",
  "poll_every": "",
  "error": "Unable to access the file"
}
```

**Nota**: sia il progresso normale che l'errore restituiscono 200. La distinzione avviene nel campo `error` del body.

### Step 4 — 303 a completamento

```http
GET https://converteranything.com/converter/mp4tomp3/48578

HTTP/1.1 303 See Other
Location: https://converteranything.com/downloads/result/48578
```

Il 303 (non il 302!) è semanticamente corretto: indica che la risposta alla richiesta originale si trova altrove, e il client deve effettuare una GET su quel nuovo URL.

---

## Semantica degli status code

| Status | Significato in questo pattern |
|---|---|
| **202 Accepted** | Richiesta ricevuta e job avviato; il processing è in corso ma non ancora completato |
| **200 OK** (polling) | Stato attuale del job — può contenere progresso o errore nel body |
| **303 See Other** | Job completato con successo; vai al risultato tramite la `Location` header |

**Perché 202 e non 200 all'avvio?** Il 200 implicherebbe che la richiesta è stata completata. Il 202 comunica esplicitamente che l'elaborazione è in corso — semantica più corretta per processi asincroni.

**Perché 303 e non 302?** Il 303 indica specificamente "usa GET per recuperare la risposta", indipendentemente dal metodo originale della richiesta. Il 302 ha semantica meno precisa.

---

## Link esterni

- Amazon RESTful API Design (API University-3) — citato come riferimento di design per REST API
- Amazon RESTful Web APIs: Services for a Changing World — libro citato come background

---

## Immagini

- ![Progress indicator UI](https://res.cloudinary.com/practicaldev/image/fetch/s--QIuQAOOt--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/dbvmq3hhlibz5kpj5b9h.png) — Screenshot di esempio di un progress indicator che visualizza i dati del polling (percentuale, tempo stimato)
