---
tags:
  - genai
  - coding-agent
  - software-architecture
  - llm
  - development-workflow
  - tdd
feature:
type: article
author: Chris Richardson
source: https://microservices.io/post/architecture/2026/05/19/genai-development-platform-part-4-coding-agent-sandwich-pattern.html
date: 2026-05-28
---

# GenAI-based development platform - part 4: The coding agent sandwich pattern

## Sunto

Il "coding agent sandwich" è un pattern architetturale per sistemi basati su GenAI che consiste nel racchiudere le invocazioni LLM tra due strati di codice deterministico tradizionale (POC — Plain Old Code). Il pattern nasce dall'osservazione pratica che delegare l'intera orchestrazione di un workflow complesso a un agente LLM produce risultati inaffidabili e costosi: il modello tende a tralasciare passaggi, a fare supposizioni errate e a sprecare token su operazioni che potrebbe risolvere in modo deterministico un normale ciclo `while`.

Il caso d'uso concreto è il workflow `implement plan`, l'ultimo passo del sistema open-source `idea-to-code` di Richardson. Il workflow riceve in input un piano composto da più task e li converte in commit Git, gestendo il ciclo completo: attesa del build CI, risoluzione dei fallimenti, risposta ai commenti della pull request e implementazione iterativa dei task usando TDD. Questo ciclo è troppo lungo e con troppe variabili per essere lasciato interamente a un agente.

Il **top slice** (fetta superiore) è un orchestratore in Python puro che dirige il workflow in modo deterministico: decide quando invocare il build fixer, quando recuperare il task successivo, quando fare push. Questo strato garantisce chiarezza, efficienza e debuggabilità — nessuna speranza che l'agente "faccia la cosa giusta". Il **filling** (ripieno) è composto da invocazioni narrowly scoped dell'agente di coding per i compiti intrinsecamente ambigui: generare messaggi di commit, analizzare e correggere fallimenti CI, rispondere a commenti di revisione, implementare i singoli task del piano. Il **bottom slice** (fetta inferiore) sono gli strumenti deterministici che ancorano l'agente alla realtà: `git`, `gh`, `uv`, `gradlew`, `gitleaks`, il CodeScene MCP server.

Il principio guida è la decomposizione funzionale: ogni responsabilità usa la tecnologia più adatta. Il codice deterministico gestisce la logica di controllo del flusso; gli LLM intervengono solo dove la determinizzabilità si rompe, ovvero dove è richiesta comprensione del linguaggio naturale, ragionamento su codice sconosciuto o generazione creativa. Il pattern è ricorsivo: anche i tool usati dall'agente possono internamente utilizzare LLM per le loro parti ambigue.

La regola pratica è diretta: prima di invocare un LLM, chiedersi se un normale `if/else` o un `subprocess.run` basta. Se sì, usare quelli. L'LLM entra in scena solo quando la determinizzabilità effettivamente si rompe, e in quel caso va dotato di strumenti deterministici per restare ancorato alla realtà. Questo approccio riduce i costi, aumenta la prevedibilità e rende il sistema molto più facile da debuggare e mantenere.

## Immagini

![The coding agent sandwich pattern — overview](https://microservices.io/i/genai/idea-to-code/coding-agent-sandwich-overview.png)
![Implement plan workflow — implementazione completa](https://microservices.io/i/genai/idea-to-code/idea-to-code-workflow-implementation.png)
![Coding agent sandwich — dettaglio strato 1](https://microservices.io/i/genai/idea-to-code/idea-to-code-coding-agent-sandwich-01.png)
![Coding agent sandwich — dettaglio strato 2](https://microservices.io/i/genai/idea-to-code/idea-to-code-coding-agent-sandwich-02.png)
![Coding agent sandwich — dettaglio strato 3](https://microservices.io/i/genai/idea-to-code/idea-to-code-coding-agent-sandwich-03.png)

## Codice

Anti-pattern: delegare l'intero workflow a un singolo prompt, affidandosi alla non-deterministica esecuzione dell'agente.

```bash
claude -p "Implement the following plan: ${plan_file} using the following workflow:

commitChanges()
pushCommits()
while True
    1. If there's a running CI build, wait for it to complete.
    2. If the CI build failed, fix it.
    3. Respond to any comments on the pull request.
    4. Get the next task from the plan.
    5. If there are no more tasks, break the loop.
    6. Implement the task.
    7. Mark the task as complete in the plan.
    8. Commit the changes to Git.
    9. Push the commit to the remote repository.
"
```

Il pattern corretto: il top slice Python orchestra deterministicamente il loop, invocando l'agente solo per i passi ambigui.

```python
def execute(self):
    self._loop_steps.commit_recovery.commit_if_needed()
    if self._git_repo.has_unpushed_commits():
        self._push_and_ensure_pr()
    while True:
        if self._loop_steps.build_fixer.check_and_fix_ci():
        ...
```
