# Null Safety in Dart: Fewer Crashes, by Design

*A simple guide to null safety — non-nullable types, the `?` `!` `?.` `??` operators, and how Dart stops null crashes before they happen.*

---

The "null crash" — `NoSuchMethodError` on a null value — used to be one of the most common bugs in mobile apps. Dart's **null safety** makes most of them impossible. The compiler simply won't let you use a value that might be null without handling it. Let me show you how it works.

## The core idea: nothing is null unless you say so

In Dart, types are **non-nullable by default**. A `String` always holds a real string — never null.

If you want to allow null, you add a `?` to the type. `String?` can hold a string **or** null.

![Non-nullable String always has a value; nullable String? can be null](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/null-safety/fig-1.png)
*A String always holds a value. A String? can hold a value or null.*

```dart
String name = 'Oguz';   // never null
name = null;            // ERROR — String can't be null

String? nick = 'Oz';    // can be null
nick = null;            // OK
```

Because the compiler knows which values can be null, it forces you to handle the null case. That is where the operators come in.

## The four operators you must know

![The null operators — ?. safe access, ?? default, ! assert, ??= assign if null](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/null-safety/fig-2.png)
*Each operator handles null in a different way.*

**`?.` — safe access.** Call something only if the value is not null; otherwise the whole thing is null.

```dart
String? name;
print(name?.length); // null — does not crash
```

**`??` — default value.** Use a fallback when the value is null.

```dart
String? name;
print(name ?? 'Guest'); // 'Guest'
```

**`??=` — assign if null.** Set a value only if the variable is currently null.

```dart
String? name;
name ??= 'Guest'; // name is now 'Guest'
```

**`!` — the bang operator.** This says "I promise this is not null." Use it carefully — if you are wrong, **it crashes**.

```dart
String? name = getName();
print(name!.length); // crashes if name is null
```

## `?.` vs `!`: safe vs risky

This is the pair interviewers test. `?.` is safe — it never crashes; it just gives null. `!` is a promise — if the value is null, your app throws.

![?. is safe and returns null; ! crashes when the value is actually null](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/null-safety/fig-3.png)
*?. protects you. ! removes the protection — use it only when you're sure.*

Rule: prefer `?.` and `??`. Only use `!` when you are certain the value is not null and the compiler just can't see it.

## Where does `late` fit?

`late` is for a value that is non-nullable but not ready yet — you promise to set it before you read it. It avoids making the type nullable just because of timing.

```dart
late String token;      // not null, but set later
void init() => token = fetchToken();
```

(If you read `late` before setting it, you get a `LateInitializationError` — a topic on its own.)

## What "sound" null safety means

Dart's null safety is called **sound**. "Sound" means the compiler can fully trust it: if a type is non-nullable, the value is *guaranteed* to never be null at runtime. This is not just a warning — it is a promise the compiler enforces. That guarantee is what removes a whole class of bugs.

## How I would answer this in an interview

> "In Dart, types are non-nullable by default, so a `String` can never be null. If I want null, I use `String?`. The compiler then forces me to handle the null case. I use `?.` for safe access, which returns null instead of crashing, `??` to give a default value, and `??=` to assign only if null. The `!` operator asserts that a value is not null, but it crashes if I'm wrong, so I avoid it unless I'm sure. `late` is for a non-nullable value I set later. And Dart's null safety is sound, which means the compiler guarantees a non-nullable value is never null at runtime."

## Key points

- Types are **non-nullable by default**; add `?` to allow null (`String?`).
- **`?.`** safe access (returns null, never crashes).
- **`??`** default value when null. **`??=`** assign only if null.
- **`!`** asserts non-null — **crashes if wrong**, so use it rarely.
- **`late`** = non-nullable, set later.
- **Sound** null safety = the compiler *guarantees* non-nullable is never null.

---

*Thanks for reading. If this helped, follow for more simple Dart & Flutter guides.*
