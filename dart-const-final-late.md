# const vs final vs late in Dart: When Each One Matters

*A simple guide to Dart's three keywords — const, final, and late — and the interview questions behind them.*

---

If you write Dart or Flutter, you use `const`, `final`, and `late` every day. In an interview, "what's the difference?" sounds easy — but it is a trap. The real answer is not about how you write them. It is about **when the value is set**, and **what the compiler already knows**.

Here is the simple way to think about it.

## The short version

- **`final`** — set **once, at runtime**. The value can be calculated while the app runs.
- **`const`** — a **compile-time** value. It must be known before the app starts. It never changes, deeply.
- **`late`** — a non-nullable variable that you set **later**. You must set it before you first read it.

## When is the value set?

This is the main idea. `const` is decided at **compile time** — before the app runs. `final` and `late` are decided at **runtime** — while the app runs.

![When is the value decided — compile-time vs runtime](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/const-final-late/fig-1.png)
*const is set before the program runs. final and late happen while it runs.*

```dart
final now = DateTime.now();   // OK — calculated at runtime
const pi = 3.14;              // OK — known at compile time
const bad = DateTime.now();   // ERROR — now() is only known at runtime
```

`const bad` fails because `DateTime.now()` is only known when the app runs. `const` needs a value the compiler can decide early.

## Which one should you use?

![Which one should you reach for — decision tree](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/const-final-late/fig-2.png)
*Two questions lead you to the right keyword every time.*

- Known at compile time and never changes → **`const`**.
- A runtime value that you set once → **`final`**.
- Non-nullable, but you can't set it yet (or it's expensive) → **`late`**.

## The `const` superpower: canonicalization

This is the part interviewers like. When two `const` values are the same, Dart keeps **one shared object** and reuses it. This is called *canonicalization*.

```dart
identical(const Point(0, 0), const Point(0, 0)); // true — same object
identical(Point(0, 0), Point(0, 0));             // false — two objects
```

![const canonicalization — one shared instance vs two](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/const-final-late/fig-3.png)
*const values become one shared object. A normal constructor makes a new object every time.*

This is **why `const` helps Flutter performance**. A `const Text('Hello')` widget is not rebuilt — Flutter reuses the same object and skips the work. Adding `const` to the fixed parts of your widget tree is one of the easiest ways to make it faster.

## `late`: useful, but be careful

`late` lets you declare a non-nullable field without a value at first:

```dart
late String token;
void init() { token = fetchToken(); }
// If you read token before init() runs → LateInitializationError (the app crashes)
```

The risk is clear: read it too early and the app crashes. But `late` also has a very useful mode — **lazy loading**:

```dart
late final config = loadConfig(); // runs loadConfig() once, on first read, then remembers it
```

`loadConfig()` does not run until something reads `config`, and it runs only once. This is great for values that are expensive and maybe never used.

## Three common mistakes

1. **`final` locks the variable, not the content.**
   ```dart
   final list = [1, 2];
   list.add(3);   // OK — the list can still change
   list = [4];    // ERROR — you can't point final to a new list
   ```
   Do you want the content to be fixed too? Use `const [1, 2]`.

2. **`const` goes all the way down.** Everything inside must also be const. `const [DateTime.now()]` does not compile.

3. **`const` fields must be `static`.** In a class, a normal field can be `final` but not `const`. A `const` field must be `static const`.

## The cheat sheet

![const vs final vs late cheat sheet](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/const-final-late/fig-4.png)
*Keep this table in mind before the interview.*

## How I would answer this in an interview

> "`final` and `const` both mean you can't reassign the variable. The main difference is *when* the value is set. `final` is set once at runtime, so the value can be calculated at runtime, like `DateTime.now()`. `const` is set at compile time, so the value must be known early. Dart also shares identical const values as one object, which is why const widgets help Flutter performance. `late` is different — it lets me declare a non-nullable variable and set it later. If I read it too early, I get a `LateInitializationError`. I also use `late final` for lazy loading when a value is expensive."

## Key points

- Think **compile time vs runtime**, not just "can't change."
- `const` → known early, never changes, one shared object, free Flutter performance.
- `final` → a runtime value, set once.
- `late` → set later; good for lazy loading, but it crashes if you read it too early.

---

*Thanks for reading. If this helped, follow for more simple Dart & Flutter guides.*
