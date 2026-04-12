An **invariant** is a condition or rule that must always be true about your object's state, regardless of what operations are performed on it.

## What are invariants?

Invariants are **constraints** that define what constitutes a "valid" state for your object. They help you:

- Design thread-safe classes correctly
- Identify what needs to be protected by synchronization
- Ensure your object never enters an invalid state

## Examples of invariants:

### Simple Counter Example:

```java
public class Counter {
    private long value;
    
    // Invariant: value >= 0 (never negative)
    // Invariant: value <= Long.MAX_VALUE (no overflow)
}
```

### Bank Account Example:

```java
public class BankAccount {
    private double balance;
    private boolean isActive;
    
    // Invariants:
    // 1. balance >= 0 (no negative balance)
    // 2. if (!isActive) then balance == 0 (closed accounts have zero balance)
}
```

### Range Class Example:

```java
public class Range {
    private int min;
    private int max;
    
    // Invariant: min <= max (min is always less than or equal to max)
}
```

### Shopping Cart Example:

```java
public class ShoppingCart {
    private List<Item> items;
    private double totalPrice;
    
    // Invariant: totalPrice == sum of all item prices
    // Invariant: items.size() >= 0
}
```

## Why invariants matter for thread safety:

When multiple fields must maintain relationships (compound invariants), you need **atomic operations** to update them together:

```java
public class Range {
    @GuardedBy("this")
    private int min;
    @GuardedBy("this")
    private int max;
    
    // Invariant: min <= max
    
    public synchronized void setRange(int newMin, int newMax) {
        if (newMin > newMax) {
            throw new IllegalArgumentException();
        }
        // Must update both atomically to preserve invariant
        this.min = newMin;
        this.max = newMax;
    }
}
```

## The process:

1. **Identify** what conditions must always be true
2. **Document** them clearly
3. **Design synchronization** to ensure operations preserve all invariants
4. **Never allow** intermediate states that violate invariants to be visible to other threads

Invariants essentially define the "rules" your object must follow to remain in a valid, consistent state.