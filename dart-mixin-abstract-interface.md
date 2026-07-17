# Mixin vs Abstract Class vs Interface in Dart: Which One and When

*A simple guide to Dart's three ways of sharing and requiring behavior — `extends`, `implements`, and `with`.*

---

"What's the difference between a mixin, an abstract class, and an interface?" is a classic Dart interview question. They overlap, so it's easy to get confused. The trick is to think about **what each one is for**: sharing code, forcing a contract, or adding a reusable ability.

## The three tools at a glance

![extends inherits code, implements forces a contract, with adds an ability](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/mixin-abstract-interface/fig-1.png)
*extends: inherit code (one parent). implements: a contract (write it all). with: add a reusable ability.*

- **`extends`** (abstract class) → inherit code from **one** parent. "**is-a**".
- **`implements`** (interface) → promise to provide methods, write them **all** yourself. "**can-do** contract".
- **`with`** (mixin) → add a reusable **ability** to a class. "**has-ability**".

## Abstract class: a base you extend

An abstract class **can't be created directly**. It's a base that holds some shared code and leaves some methods for the child to fill in. You use `extends`, and a class can extend **only one** parent.

```dart
abstract class Animal {
  void breathe() => print('breathing'); // shared code
  void makeSound();                      // child must fill this in
}

class Dog extends Animal {
  @override
  void makeSound() => print('Woof');
}
```

Use it when classes share a real "is-a" relationship and some common code. A `Dog` **is an** `Animal`.

## Interface: a contract with no shared code

In Dart, **every class is also an interface.** When you `implements` a class, you promise to provide **all** of its methods — but you inherit **no** code. You write everything yourself.

```dart
class Logger {
  void log(String msg) => print(msg);
}

class SilentLogger implements Logger {
  @override
  void log(String msg) {} // must write it — nothing is inherited
}
```

You can `implements` **many** interfaces at once. Use it when you only want to guarantee a shape (a contract), not share code.

## Mixin: a reusable ability

A **mixin** is a block of behavior you can add to many classes with `with`. It's not a parent and not a contract — it's shared code you "mix in". A class can use **many** mixins.

![A class combines several mixins with the `with` keyword](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/mixin-abstract-interface/fig-2.png)
*A mixin adds ready-made behavior. One class can mix in several.*

```dart
mixin Swimmer {
  void swim() => print('swimming');
}

mixin Flyer {
  void fly() => print('flying');
}

class Duck extends Animal with Swimmer, Flyer {
  @override
  void makeSound() => print('Quack');
}

// Duck can now breathe(), makeSound(), swim(), and fly()
```

The power: `Swimmer` and `Flyer` can be added to **any** class, even unrelated ones. A `Duck` and a `Fish` can both be `Swimmer`, without sharing a parent. A mixin has **no constructor** — that's the main limit.

## When to use which

![Decision table — extends for is-a, implements for a contract, with for shared ability](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/mixin-abstract-interface/fig-3.png)
*Pick by intent: is-a, a contract, or a shared ability.*

- Share code **and** a type, one parent, "is-a" → **`extends`** an abstract class.
- Just guarantee a shape, no shared code, can be many → **`implements`** (interface).
- Reuse an ability across unrelated classes, can be many → **`with`** (mixin).

Quick memory aid: **is-a → extends. can-do → implements. has-ability → with.**

## How I would answer this in an interview

> "An abstract class is a base you `extends` to inherit shared code, and you can only extend one — it's an 'is-a' relationship. An interface in Dart is any class you `implements`: you must provide all its methods yourself and you inherit no code, but you can implement many. A mixin is reusable behavior you add with `with` — it's not a parent or a contract, just shared code you can mix into many unrelated classes, and it has no constructor. So I use extends for is-a with shared code, implements for a pure contract, and a mixin to reuse an ability across different classes."

## Key points

- **`extends`** (abstract class) → inherit code, **one** parent, "is-a".
- **`implements`** (interface) → provide **all** methods yourself, no inherited code, **many** allowed.
- **`with`** (mixin) → reusable ability added to **many** classes; **no constructor**.
- Memory aid: **is-a → extends · can-do → implements · has-ability → with.**

---

*Thanks for reading. If this helped, follow for more simple Dart & Flutter guides.*
