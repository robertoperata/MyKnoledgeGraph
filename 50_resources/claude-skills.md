---
tags:
  - claude
  - tools
  - automation
type: reference
updated: 2026-05-03
---

# Claude Skills & Routines

Elenco delle skill (comandi slash locali) e delle routine remote (agenti cloud schedulati) disponibili in questo workspace.

---

## Routine remote

Le routine sono agenti Claude che girano nel cloud Anthropic su schedule fisso. Non hanno accesso alla macchina locale — operano su un checkout git remoto.

Gestione: https://claude.ai/code/routines

### Tech Newsletter — Daily Technical Digest

| Campo | Valore |
|-------|--------|
| ID | `trig_01SaaT21DtSsiQwPdyCTVbi8` |
| Schedule | Ogni giorno alle 7:00 UTC (9:00 Europe/Rome) |
| Repo | `github.com/robertoperata/MyKnoledgeGraph` |
| MCP | Gmail |
| Modello | claude-sonnet-4-6 |

**Cosa fa:** Ogni mattina legge le email delle ultime 24h da 6 newsletter tecniche, scarica gli articoli, produce un riassunto tecnico in italiano in formato Markdown e fa commit+push sul repo.

**Sorgenti monitorate:**

| Mittente | Cartella output | Logica |
|----------|----------------|--------|
| `vlad@vladmihalcea.com` | `articles/vlad-mihalcea/` | Solo email tecniche (Hibernate, JPA, DB) — salta commerciali |
| `thorben@thorben-janssen.com` | `articles/thorben-janssen/` | Solo email tecniche (Hibernate, JPA, DB) — salta commerciali |
| `info@deararchitects.xyz` | `articles/dear-architects/` | Segue tutti i link "Discover more" |
| `chris@chrisrichardson.net` | `articles/chris-richardson/` | Segue il link "keep reading" se presente |
| `architect-newsletter@mail.infoq.com` | `articles/infoq/` | Filtra per Java, architettura, Kubernetes, AI, persistence, Kafka; crea report giornaliero |
| `2minutestreaming@mail.beehiiv.com` | `articles/2-minute-streaming/` | Riassume il corpo dell'email + eventuale trascrizione podcast |

---

## Skill locali

Le skill sono comandi slash (`/nome`) che eseguono nel contesto locale della sessione Claude Code corrente.

### Workflow & automazione

| Skill | Trigger | Descrizione |
|-------|---------|-------------|
| `/schedule` | "schedule", "routine", "cron" | Crea, aggiorna, lista o esegue subito le routine remote. Gestisce conversione timezone Europe/Rome → UTC. |
| `/loop` | "loop", "ripeti ogni X" | Esegue un prompt o un altro slash command a intervallo ricorrente all'interno della sessione corrente. |
| `/update-config` | "from now on when X", "allow X", "set env var" | Modifica `settings.json` per configurare hook, permessi e variabili d'ambiente del harness Claude Code. |

### Knowledge base & contenuti

| Skill | Trigger | Descrizione |
|-------|---------|-------------|
| `/save-article` | "salva articolo `<url>`" | Legge un articolo da URL, crea un riassunto dettagliato in italiano e lo salva in `40_sources/articles/`. |
| `/course-summarizer` | URL di streaming (MPD, HLS, yt-dlp) | Scarica audio, trascrive con faster-whisper e produce un'elaborazione completa del corso in italiano. Salva in `40_sources/courses/`. |
| `/podcast-summarizer` | Link YouTube/Spotify, file audio, trascrizione grezza | Trascrive e riassume in modo strutturato un episodio podcast. Gestisce il flusso completo audio → trascrizione → riassunto. |
| `/oreilly-recordings` | "/oreilly-recordings" | Legge le mail Gmail con label "O'Reilly recordings", estrae metadati dei live event e aggiorna il CSV locale. |
| `/watch:watch` | URL o path video locale | Scarica con yt-dlp, estrae frame con ffmpeg, trascrive, risponde a domande sul contenuto del video. |

### Knowledge graph & DDD

| Skill | Trigger | Descrizione |
|-------|---------|-------------|
| `/ubiquitous-language` | "glossario", "ubiquitous language", "DDD" | Estrae un glossario DDD dalla conversazione corrente, segnala ambiguità e propone termini canonici. Salva in `UBIQUITOUS_LANGUAGE.md`. |
| `/grill-me` | "grillami", "stress-test il piano" | Intervista l'utente in modo serrato su un piano o design finché non si raggiunge una comprensione condivisa. |

### Repository & codice

| Skill | Trigger | Descrizione |
|-------|---------|-------------|
| `/sync-repos` | "sync repos", "aggiorna tutti i repo" | Per ogni progetto nel workspace esegue `git fetch --all --prune` e fast-forward se possibile. Segnala repo divergenti o sporchi. |
| `/init` | "init" | Inizializza un nuovo file `CLAUDE.md` con la documentazione del codebase corrente. |
| `/review` | "review" | Esegue una code review del pull request corrente. |
| `/security-review` | "security review" | Esegue una security review completa dei cambiamenti in staging sul branch corrente. |
| `/simplify` | "simplify" | Analizza il codice modificato per riuso, qualità ed efficienza, poi corregge i problemi trovati. |
| `/explain-code` | "how does this work?", "spiegami il codice" | Spiega il funzionamento del codice con diagrammi visivi e analogie. |
| `/claude-api` | Import `anthropic`, Anthropic SDK | Aiuta a costruire, debug e ottimizzare app che usano l'API Claude / Anthropic SDK. Include prompt caching. |

### Configurazione Claude Code

| Skill | Trigger | Descrizione |
|-------|---------|-------------|
| `/keybindings-help` | "keybinding", "rebind", "shortcut" | Configura le scorciatoie da tastiera in `~/.claude/keybindings.json`. |
| `/fewer-permission-prompts` | "meno prompt di permesso" | Scansiona i transcript e aggiunge una allowlist al progetto per ridurre le richieste di conferma. |
| `/statusline-setup` | (uso interno) | Configura la status line di Claude Code. |

### Newsletter

| Skill | Trigger | Descrizione |
|-------|---------|-------------|
| `/highnorth-newsletter-digest` | (uso specifico) | Digest della newsletter High North. |
