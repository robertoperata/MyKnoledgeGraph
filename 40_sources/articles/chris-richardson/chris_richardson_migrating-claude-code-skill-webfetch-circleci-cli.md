---
tags:
  - claude-code
  - ai-agent-tools
  - circleci
  - cli-design
  - microservices
feature:
type: article
author: Chris Richardson
source: https://microservices.io
date: 2026-07-29
---

# Migrating a Claude Code Skill from WebFetch to the New CircleCI CLI: Lessons Learned About Tool and API Design

## Sunto

Chris Richardson descrive come ha migrato uno dei suoi skill di Claude Code — `debugging-ci-failures` — dall'uso diretto dell'API REST di CircleCI tramite il tool `WebFetch` al nuovo CircleCI CLI. Lo skill istruisce Claude Code su come monitorare e diagnosticare una build CI fallita in cinque fasi: attendere il completamento della build, identificare il job fallito, recuperare l'output del passo fallito, scaricare gli artefatti di test e leggere i file JUnit `TEST-*.xml` per trovare la causa radice.

Il vecchio approccio basato su WebFetch presentava diverse criticità: Claude Code doveva conoscere i dettagli dell'API REST, i repository privati richiedevano un API token da esporre all'agente, e il tool WebFetch mantiene in cache le risposte per circa 15 minuti. Quest'ultimo problema obbligava lo skill a usare un parametro di cache-busting artigianale (`&_ts=1`) su ogni richiesta — un hack inelegante e fragile.

La migrazione al nuovo CircleCI CLI ha eliminato tutti questi problemi. Il CLI gestisce l'autenticazione in modo trasparente, nasconde i dettagli dell'API REST e offre comandi dedicati per ogni fase del workflow. La migrazione ha richiesto un singolo prompt di una riga, ma l'aspetto più interessante non è la meccanica della migrazione bensì ciò che Claude Code ha scoperto esplorando autonomamente l'albero dei comandi del CLI tramite `--help`.

L'articolo si chiude con tre lezioni di design generalizzabili: preferire i CLI alle API grezze quando si costruiscono skill per agenti AI; progettare i CLI in modo che siano esplorabili (l'output di `--help` è un'interfaccia di prima classe, non solo una comodità per gli umani); e progettare i CLI per un output token-efficiente, come dimostrato dal flag `--condensed` di CircleCI che filtra server-side le righe rumorose prima di restituire l'output a un LLM.

Un risultato inatteso della migrazione è stato la scoperta di `circleci testresult list <job-id>`, un comando che permette a Claude Code di identificare i test falliti senza scaricare alcun artefatto. Lo skill ora prova questo approccio rapido prima di ricorrere al download dei file `TEST-*.xml`, riducendo ulteriormente il numero di token consumati e la latenza della diagnosi.

## Immagini

![From WebFetch to CircleCI CLI](https://microservices.io/i/genai/idea-to-code/from-webfetch-to-circleci-cli.png)

## Codice

Il vecchio approccio richiedeva un parametro di cache-busting su ogni richiesta WebFetch all'API CircleCI:

```
WebFetch: https://circleci.com/api/v1.1/project/github/<org>/<repo>?limit=1&branch=<branch>&_ts=1
```

Il nuovo CLI sostituisce ogni step con un comando dedicato. Tabella di mapping completa:

| Passo dello skill | Vecchio (WebFetch + hack `_ts`) | Nuovo (CLI `circleci`) |
|---|---|---|
| Attendere la build | Poll su `/build/<n>?_ts=N` in loop | `circleci run watch` — bloccante, exit code = esito |
| Trovare l'ultima run | WebFetch endpoint progetto | `circleci run list --branch <branch>` |
| Stato di run/workflow/job | WebFetch + parsing JSON | `circleci run get <run-id> --json` |
| Trovare il passo fallito | WebFetch `?include=steps` + scan | `circleci job output list <job-id> --json --jq '.steps[] \| select(.exit_code != 0)'` |
| Output del passo fallito | WebFetch `/output/<idx>/0` | `circleci job output get <job-id> --step-num N` |
| Lista artefatti | WebFetch `/artifacts` | `circleci artifact <job-id> --json` |
| Download artefatti | WebFetch su ogni URL | `circleci artifact <job-id> --output test-reports` |

Comando per identificare i test falliti senza scaricare artefatti:

```bash
circleci testresult list <job-id>
```

Il flag `--condensed` per output token-efficiente filtrato server-side:

```bash
circleci job output get <job-id> --step-num N --condensed
```
