# `==` vs `identical` in Dart: How Equality Really Works

*Two objects look the same — but are they? A simple guide to Dart equality, the hashCode rule, and a bug that quietly breaks your Sets.*

---

Dart gives you two ways to ask "are these the same?" — and mixing them up causes many small, hidden bugs. One asks about **identity** (the same object in memory). The other asks about **equality** (the same *value*). Interviewers like this topic because a wrong answer breaks `Set`, `Map`, and widget rebuilds.

## The two questions

- **`identical(a, b)`** — "Are these the **same object** in memory?" (reference)
- **`a == b`** — "Do these **mean the same thing**?" (value — and you decide what that means)

![identical vs == — reference identity vs value equality](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/equality/fig-1.png)
*identical asks about the object. == asks about the meaning.*

By default, `==` is the same as `identical` — Dart's `Object.==` only checks if it is the same object. Things change only when you **override** it.

```dart
class Point {
  final int x, y;
  const Point(this.x, this.y);
}

final a = Point(1, 2);
final b = Point(1, 2);

a == b;              // false — different objects, no override
identical(a, b);     // false — clearly two different objects
```

Two points with the same numbers are "not equal" — because we never told Dart what equal means for `Point`.

## Overriding `==` — and the hashCode rule

To get value equality, override `==`. But there is one rule you **must** follow:

> If `a == b`, then `a.hashCode` **must** equal `b.hashCode`.

```dart
class Point {
  final int x, y;
  const Point(this.x, this.y);

  @override
  bool operator ==(Object other) =>
      other is Point && other.x == x && other.y == y;

  @override
  int get hashCode => Object.hash(x, y);
}
```

If you break this rule, you get a bug that is very hard to find.

![The == and hashCode rule, and how breaking it corrupts a Set](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/equality/fig-2.png)
*Override == but forget hashCode, and your Set stops removing duplicates.*

Here is why. A `Set` or `Map` first groups objects by `hashCode`, then compares them with `==` inside each group. If two "equal" objects get different hash codes, they go to different groups, so `==` never runs. The `Set` keeps both, and a `Map` lookup fails.

```dart
final set = {Point(1, 2), Point(1, 2)};
// hashCode overridden → length is 1  ✅
// == overridden but NOT hashCode → length is 2  💥
```

## Dart 3 shortcut: records

If you only need value equality for a few fields, **records** give it to you for free — no extra code:

```dart
final a = (1, 2);
final b = (1, 2);
a == b;   // true — records compare by value automatically
```

This is often the fastest way to compare "just some values" without writing a class.

## The link to const

Remember canonicalization? Because two identical `const` values share one object, identity is `true`:

```dart
identical(const Point(1, 2), const Point(1, 2)); // true — same shared object
```

So `const` gives you identity equality for free. You don't need to override anything.

## Why this matters in Flutter

Flutter uses `==` a lot. `BlocBuilder`, `Selector`, and `ValueListenableBuilder` skip a rebuild when the new value `==` the old one. If your state objects don't compare by value, Flutter thinks every new object is different and rebuilds too much. This is exactly why people use `Equatable`, `freezed`, or records for state.

## How I would answer this in an interview

> "`identical` checks if two variables point to the same object in memory. `==` checks equality, and by default it is the same as `identical`, unless the class overrides it. When I override `==`, I always override `hashCode` too, because equal objects must have equal hash codes. If not, Sets and Maps break, because they group by hashCode before they compare. In Dart 3 I often use records when I just need value equality without a class. And const values are shared, so identical const objects are already `identical`."

## Key points

- `identical` → the same object. `==` → the same value (if you define it).
- Override `==` → **always** override `hashCode`. It is a rule, not a choice.
- A wrong hashCode = a quietly broken `Set` or `Map`.
- Dart 3 **records** give value equality for free.
- `const` sharing makes identical const values `identical`.

---

*Thanks for reading. If this helped, follow for more simple Dart & Flutter guides.*
