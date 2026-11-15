---
tags:
  - testing
  - ai-coding
  - test-driven-development
  - specification-by-example
  - software-quality
feature:
type: article
author: Matteo Vaccari
source: https://martinfowler.com/articles/tests-as-data.html
date: 2026-11-15
---

# Tests as Data: Ensuring AI-Generated Code Does What We Want

## Sunto

In questo articolo pubblicato su martinfowler.com, Matteo Vaccari (Technical Principal at Thoughtworks Italia) affronta un problema emergente nell'era degli agenti AI per il coding: quando un agente scrive sia il codice di produzione che i test, chi garantisce che il comportamento desiderato sia correttamente catturato? La risposta proposta è spostare il contratto tra umano e AI dai test-come-codice ai **test-come-dati**.

Il problema fondamentale è che gli agenti AI tendono a riscrivere i test insieme al codice durante il refactoring, accoppiando i test all'implementazione piuttosto che al comportamento. I test diventano così una descrizione di come funziona il codice (spesso sbagliato), non di come dovrebbe funzionare. Se i test sono dati — file YAML, CSV, tabelle Markdown — separati dall'harness di esecuzione, è molto più difficile per un agente modificarli inconsapevolmente o per convenienza.

La distinzione chiave è tra **customer tests** (che verificano che il sistema faccia quello che il business si aspetta) e **developer tests** (che verificano l'implementazione interna). I customer tests dovrebbero essere scritti da umani o almeno approvati esplicitamente, perché rappresentano il contratto con il cliente. Mantenerli come dati li rende stabili e leggibili da non-sviluppatori.

L'articolo propone tecniche concrete: tabelle per input scalari/output, scenari strutturati per casi più complessi (con sezioni GIVEN/WHEN/THEN come dati), linguaggi di test specifici del dominio con parser e printer. Per applicazioni UI, una rappresentazione testuale semplificata della pagina (ad esempio un rendering ASCII della struttura) può sostituire i test HTML fragili. La tradizione su cui si basa include FIT, Specification by Example, i tabular tests di Go, e l'approval testing.

Un pattern importante è la protezione dell'harness di test: Vaccari propone una tabella esplicita che descrive quali file dovrebbero cambiare per ogni tipo di task (nuova funzionalità, refactoring, miglioramento copertura, miglioramento DSL). Se l'AI modifica file che non dovrebbe toccare per quel task, è un segnale di allarme. Questo approccio trasparente rende il comportamento atteso dell'agente verificabile.

## Codice

I test HTTP standard in Java mostrano l'eccessivo boilerplate che oscura l'intenzione del test:

```java
@Test
void defaultGreeting() throws IOException, InterruptedException {
    HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(baseUrl + "/hello"))
            .GET()
            .build();

    HttpResponse<String> response =
            client.send(request, HttpResponse.BodyHandlers.ofString());

    assertEquals(200, response.statusCode());
    assertEquals("Hello, world!", response.body());
}
```

Lo stesso test come dati YAML: solo il nome, l'input e il risultato atteso — senza codice boilerplate:

```yaml
suite: greetings
tests:
  - name: "default greeting"
    request:
      resource: "/hello"
      name: null
    greeting: "Hello, world!"
  - name: "personalised greeting"
    request:
      resource: "/hello"
      name: "Joe"
    greeting: "Hello, Joe!"
  - name: "name must not be entirely whitespace"
    request:
      resource: "/hello"
      name: " "
    error: "Name must not be entirely whitespace"
```

Tabella di accesso all'admin dashboard — coprire molti ruoli e risorse in forma compatta senza scenario Gherkin per ciascuno:

```
| admin_dashboard_access_control                                             |
| role    | session_state | resource          | expected_result | logged |
|----------------------------------------------------------------------------|
| trader  | valid         | admin dashboard   | denied          | yes    |
| trader  | valid         | admin API         | forbidden       | yes    |
| admin   | valid         | admin dashboard   | allowed         | no     |
| admin   | valid         | admin API         | allowed         | no     |
| viewer  | valid         | admin dashboard   | denied          | yes    |
| support | valid         | admin dashboard   | denied          | yes    |
| guest   | none          | admin dashboard   | login required  | yes    |
| trader  | expired       | admin dashboard   | login required  | yes    |
| trader  | tampered      | admin API         | unauthorized    | yes    |
| trader  | valid         | admin direct URL  | denied          | yes    |
```

Test per un interprete BASIC come dati YAML — un programma per input, una lista di output attesi:

```yaml
tests:
  - name: "LEN function basic tests"
    program: |
      10 PRINT LEN("HELLO WORLD")
      20 PRINT LEN("")
      30 PRINT LEN("A")
      40 END
    expected:
      - "11\n"
      - "0\n"
      - "1\n"
```

Scenario strutturato per una UI todo-list con sezioni GIVEN/WHEN/THEN come dati:

```
## Unchecking a completed todo reactivates it
GIVEN model:
    [x] buy milk
    [ ] walk the dog
WHEN user:
    click toggle of "buy milk"
THEN page:
    > ""
    [ ] buy milk
    [ ] walk the dog
    -- **2** items left
THEN model:
    [ ] buy milk
    [ ] walk the dog
```

Tabella che definisce i cambiamenti attesi per ogni tipo di task — strumento per proteggere l'harness dall'AI:

```
| Quando                             | Codice produzione? | Dati test? | Test runner? |
| Nuova funzionalità                 | Sì                 | Sì         | No           |
| Refactoring codice produzione      | Sì                 | No         | No           |
| Miglioramento copertura            | No                 | Sì         | No           |
| Miglioramento DSL di test          | No                 | Sì         | Sì           |
```
