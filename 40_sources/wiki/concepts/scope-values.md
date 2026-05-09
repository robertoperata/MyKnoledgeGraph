---
title: Scope Values
type: concept
tags: [thread2-java]
sources:
  - "[[youtube_java-25-lts-features-jchampions]]"
updated: 2026-05-08
related:
  - "[[concepts/virtual-threads]]"
  - "[[concepts/structured-concurrency]]"
  - "[[concepts/java-concurrency]]"
---

# Scope Values (Java 25)

## Definizione

I `ScopeValue` sono una sostituzione moderna di `ThreadLocal` per condividere valori **immutabili** all'interno di un contesto di esecuzione delimitato. Diventati finali in Java 25 dopo quattro versioni di preview.

## Problema con ThreadLocal

`ThreadLocal` ha tre problemi strutturali:

1. **Non si propaga tra thread diversi**: se si vuole passare un valore (es. credenziali di autenticazione) a task eseguiti su altri thread, bisogna copiarlo manualmente
2. **Memory leak**: il valore rimane nel thread fino a rimozione esplicita; i thread pool riusano i thread, quindi i valori "vecchi" persistono
3. **Alto consumo di memoria**: ogni thread mantiene una mappa di tutti i ThreadLocal attivi

## Come funzionano i Scope Values

```java
// Dichiarazione (equivalente a ThreadLocal)
static final ScopeValue<Credentials> AUTH = ScopeValue.newInstance();

// Binding: definisce il valore per il blocco
ScopeValue.where(AUTH, myCredentials).run(() -> {
    // AUTH.get() disponibile qui e in tutti i metodi chiamati da qui
    fetchData();      // può accedere a AUTH.get()
    callService();    // idem
});
// Fuori dal blocco: AUTH.get() lancia NoSuchElementException
```

**Caratteristiche chiave:**
- **Immutabili**: il valore non può essere modificato dopo il binding
- **Scoped**: disponibili solo all'interno del blocco `run()`/`call()`
- **Propagazione automatica con Structured Concurrency**: con `StructuredTaskScope` il valore si propaga ai subtask (quando Structured Concurrency sarà finale in Java 29)

## Confronto con ThreadLocal

| Aspetto | ThreadLocal | ScopeValue |
|---|---|---|
| **Mutabilità** | Mutable (`set()` disponibile) | Immutabile (bind-only) |
| **Scope** | Thread lifetime (rimozione manuale) | Delimitato al blocco |
| **Propagazione thread** | No (copia manuale) | Automatica con StructuredTaskScope |
| **Memory** | Persiste nel thread pool | Garbage collected fuori dallo scope |
| **Memory leak** | Rischio alto | Nessun rischio |
| **Uso ideale** | Stato mutabile per-thread | Contesto immutabile (auth, trace ID, config) |

## Caso d'uso tipico: contesto di autenticazione

```java
static final ScopeValue<String> USER_ID = ScopeValue.newInstance();

// Nel request handler
ScopeValue.where(USER_ID, request.getUserId()).run(() -> {
    service.process();  // service può leggere USER_ID.get()
    audit.log();        // audit può leggere USER_ID.get()
});

// In un metodo chiamato nello scope
class AuditService {
    void log() {
        String userId = USER_ID.get(); // disponibile automaticamente
        auditLog.write(userId + ": action performed");
    }
}
```

## Nota: integrazione con Structured Concurrency

Quando `StructuredTaskScope` (ancora preview in Java 25) sarà finale, i `ScopeValue` si propagheranno automaticamente ai subtask strutturati senza nessun codice aggiuntivo. Per ora la propagazione automatica non è disponibile nei subtask paralleli.

## Connessioni

- [[concepts/virtual-threads]] — i ScopeValue sono thread-agnostic: funzionano sia con platform thread che con virtual thread
- [[concepts/structured-concurrency]] — la propagazione automatica di ScopeValue ai subtask dipende da StructuredTaskScope (Java 29)
- [[concepts/java-concurrency]] — sostituzione di ThreadLocal nel modello di concorrenza moderno Java
