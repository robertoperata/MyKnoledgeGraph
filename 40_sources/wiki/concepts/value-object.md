---
title: Value Object
type: concept
tags: [thread5-ddd]
sources:
  - "[[Domain-Driven Design Aggregates Domain Events and Value Objects]]"
updated: 2026-04-09
related:
  - "[[concepts/aggregate]]"
  - "[[concepts/bounded-context]]"
---

# Value Object

## Definizione

Un Value Object è un oggetto di dominio definito dai suoi **attributi**, non da un'identità. Due value object con gli stessi attributi sono uguali. I value object sono **immutabili**: se si deve cambiare un valore, si crea un nuovo value object.

## Come funziona

Caratteristiche:
- **Nessuna identità**: l'uguaglianza è per valore, non per riferimento
- **Immutabilità**: non ha setter, lo stato non cambia dopo la costruzione
- **Self-validation**: contiene la logica di validazione dei propri valori
- **Side-effect free**: i metodi non modificano lo stato, ma restituiscono nuovi value objects

```java
// Value object: Money
public record Money(BigDecimal amount, Currency currency) {
    public Money {
        Objects.requireNonNull(amount);
        Objects.requireNonNull(currency);
        if (amount.compareTo(BigDecimal.ZERO) < 0)
            throw new IllegalArgumentException("Amount cannot be negative");
    }
    
    public Money add(Money other) {
        if (!this.currency.equals(other.currency))
            throw new IllegalArgumentException("Cannot add different currencies");
        return new Money(this.amount.add(other.amount), this.currency);
    }
}
```

## "Power through simplicity"

Il sottotitolo del corso DDD (Vaughn Vernon) per i value objects è "modelling concepts as Value Objects: power through simplicity". La semplicità è la forza: un value object che porta la propria logica di validazione e le proprie operazioni è molto più espressivo di un semplice `BigDecimal` o `String`.

## Connessione con Java

I **Java Records** (introdotti in Java 16 come feature stabile) sono la rappresentazione naturale dei value objects in Java moderno: immutabili per default, uguaglianza basata sui componenti, compatti.

## Connessioni

- [[concepts/aggregate]] — gli aggregates contengono value objects
- [[concepts/bounded-context]] — gli stessi concetti reali possono diventare value objects diversi in bounded context diversi (es. "Address" nel contesto della spedizione vs nel contesto della fatturazione)
