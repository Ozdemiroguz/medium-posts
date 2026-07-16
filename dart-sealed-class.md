# Sealed Classes in Dart: Safe, Complete `switch` by Design

*A simple guide to sealed classes — what they are, why Dart 3 added them, and how they make your `switch` statements safe.*

---

Dart 3 added a small keyword with a big effect: `sealed`. It solves a problem every developer has hit — you add a new case to your app, forget to handle it somewhere, and find the bug much later. Sealed classes make the compiler catch this for you.

## What is a sealed class?

A `sealed` class is a class that:

- **cannot be created directly** (it is abstract), and
- **can only be extended inside the same file**.

That second rule is the key. Because all the subtypes live in one file, the compiler knows the **complete list** of them. There is no way to add a surprise subtype from somewhere else.

```dart
sealed class Shape {}

class Circle extends Shape {
  final double radius;
  Circle(this.radius);
}

class Square extends Shape {
  final double side;
  Square(this.side);
}
```

Now `Shape` has exactly two subtypes: `Circle` and `Square`. The compiler knows this for sure.

![A sealed class with a known, closed set of subtypes](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/sealed/fig-1.png)
*A sealed class knows all of its subtypes, because they must live in the same file.*

## The main benefit: exhaustive `switch`

Because the compiler knows every subtype, it can check that your `switch` handles **all of them**. This is called an *exhaustive* switch.

```dart
double area(Shape shape) => switch (shape) {
  Circle(radius: var r) => 3.14 * r * r,
  Square(side: var s)   => s * s,
};
```

No `default` branch needed. And here is the useful part: if you later add a third shape, this `switch` **stops compiling** until you handle it.

```dart
class Triangle extends Shape { /* ... */ }
// Now area() gives a compile error: "The switch is missing Triangle."
```

The compiler becomes your checklist. You cannot forget a case, because the code will not build.

![Adding a new subtype breaks every non-exhaustive switch at compile time](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/sealed/fig-2.png)
*Add a new subtype, and the compiler points you to every switch you must update.*

## sealed vs abstract: what's the difference?

They look similar — both can't be created directly. The difference is about **who can extend them** and whether the compiler can check exhaustiveness.

![sealed vs abstract — closed set vs open set](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/sealed/fig-3.png)
*abstract is open for anyone to extend. sealed is a closed, known set.*

- **`abstract`** — an open set. Anyone, in any file, can extend it. The compiler does **not** know all subtypes, so a `switch` still needs a `default`.
- **`sealed`** — a closed set. Only the same file can extend it. The compiler knows all subtypes, so `switch` is exhaustive and needs no `default`.

Rule of thumb: use `sealed` when you have a **fixed, known set** of types and you want the compiler to check every case. Use `abstract` when you want an **open** type that others can extend freely (like a base class in a library).

## A very common use: result types

Sealed classes are perfect for modeling "one of a few states". A network result is a great example:

```dart
sealed class Result<T> {}
class Success<T> extends Result<T> { final T data; Success(this.data); }
class Failure<T> extends Result<T> { final String error; Failure(this.error); }

Widget build(Result<User> result) => switch (result) {
  Success(data: var user) => UserView(user),
  Failure(error: var msg) => ErrorView(msg),
};
```

If you add a `Loading` state later, every `switch` on `Result` will tell you to handle it. No missed states, no silent bugs.

## How I would answer this in an interview

> "A sealed class is an abstract class that can only be extended in the same file. Because of that, the compiler knows the full set of subtypes. The main benefit is exhaustive switch — the compiler checks that I handle every subtype, and I don't need a default case. If I add a new subtype later, the code stops compiling until I handle it everywhere. The difference from abstract is that abstract is open — anyone can extend it — so the compiler can't check all cases. I use sealed for a fixed set of states, like a Result type with success and failure."

## Key points

- `sealed` = abstract + can only be extended in the same file.
- The compiler knows the **full list** of subtypes.
- This gives **exhaustive `switch`** — no `default` needed.
- Add a subtype later → the compiler shows you every place to fix.
- `abstract` = open set. `sealed` = closed, known set.
- Great for result and state types.

---

*Thanks for reading. If this helped, follow for more simple Dart & Flutter guides.*
