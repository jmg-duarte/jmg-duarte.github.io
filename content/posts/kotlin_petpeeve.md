---
title: "My Kotlin Pet Peeve"
date: 2019-08-05T19:32:28+01:00
draft: true
---

**Disclaimer**: I'm not more than a Kotlin user, this post expresses my opinion and should not be taken as any kind of attack.

In the beggining of summer I got accepted for a summer internship in [Caixa Mágica](https://www.caixamagica.pt/en),
when I arrived there I had no clue what I would be doing,
in retrospective this seems a really stupid idea but things worked out for the best,
I was put on an Android project,
building a library to interface with the portuguese Citizen Card. 

I've done Android before, during my Erasmus, I cannot say I loved it, 
however I didn't hated it either.
One of the reasons for such was [Kotlin](https://kotlinlang.org/), 
it made code so much simpler, 
and as stupid as it sounds felt like it had a better "keystroke mileage", 
allowing me to get more done in less code.

*Doing Android with Java now seems like an herculean task.*

However, as great as Kotlin is, it is not without its flaws (at least in my eyes).

What hurts me everytime I use it is nothing more than the range operator, let me explain why.

```kotlin
fun main() {
    for (i in 0..9) {
        println(i)
    }
}
```

```kotlin
fun main() {
    for (i in 0 until 9) {
        println(i)
    }
}
```

```rust
for i in 0..=9 {
    println!("{}", i);
}
```

```rust
for i in 0..9 {
    println!("{}", i);
}
```