---
tags:
  - kotlin
  - spring
feature: 
type: live event
author: "[[Ken Kousen]]"
source: 
---
# Kotlin and Spring

##### Unit 
Rappresenta l'**assenza di un valore significativo** — è l'equivalente di `void` 
##### Nothing
Rappresenta un tipo che **non ha istanze** e indica che una funzione **non termina mai** — o perché lancia sempre un'eccezione, o perché è un loop infinito.

```kotlin
fun main(Array<String> args) {
}
```

La gerarchia è
Any (tipo Object ma non nullable)
Unit, String ecc ecc
Nothing (sottoclasse di tutto per cui sarà sempre qualcos'altro)

```kotlin
val // final
var // variable
var name:String? = "Priz" //name può essere nullo
println("Hello, ${name}") // variable interpolation
```

