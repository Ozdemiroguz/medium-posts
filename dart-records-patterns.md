# Records and Pattern Matching in Dart 3: Cleaner Data, Smarter `switch`

*A deep but simple guide to two of Dart 3's best features — records for bundling values, and patterns for taking them apart.*

---

Dart 3 added two features that change how you write everyday code: **records** and **patterns**. Records let you group values without a class. Patterns let you pull those values back out — in a variable, a `switch`, or an `if`. Together they make a lot of boilerplate disappear. Let's go through them slowly.

## Records: group values without a class

A **record** bundles a few values together. You don't need to write a class for it.

```dart
(int, String) pair = (1, 'hello'); // a record with two fields
print(pair.$1); // 1
print(pair.$2); // 'hello'
```

You can also give the fields **names**, which is much clearer:

```dart
({int x, int y}) point = (x: 3, y: 4);
print(point.x); // 3
print(point.y); // 4
```

![A record bundles values with positional or named fields](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/records-patterns/fig-1.png)
*A record groups values — by position ($1, $2) or by name (.x, .y).*

Records also have **value equality for free** — two records with the same fields are equal, no `==` or `hashCode` needed.

## The best use: returning more than one value

Before records, returning two values meant writing a class or using a `List`. Now it's one line:

```dart
(double, double) minMax(List<double> nums) {
  var min = nums.first, max = nums.first;
  for (final n in nums) {
    if (n < min) min = n;
    if (n > max) max = n;
  }
  return (min, max); // return two values at once
}
```

## Patterns: take the values back apart

A **pattern** destructures a value — it pulls the pieces out into variables. The simplest pattern unpacks a record:

```dart
final (min, max) = minMax(scores); // min and max are now separate variables
final (x: px, y: py) = point;      // unpack named fields
```

![A pattern destructures a record into separate variables](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/records-patterns/fig-2.png)
*A pattern unpacks a value straight into named variables.*

## Patterns in `switch`: the real power

Patterns shine in a `switch`. You can match the shape of an object **and** pull out its fields at the same time. This pairs perfectly with sealed classes (which make the switch exhaustive):

```dart
sealed class Shape {}
class Circle extends Shape { final double r; Circle(this.r); }
class Square extends Shape { final double side; Square(this.side); }

double area(Shape shape) => switch (shape) {
  Circle(r: var r)     => 3.14 * r * r,
  Square(side: var s)  => s * s,
};
```

You can also add a **guard** with `when` — an extra condition on a case:

```dart
String size(Shape shape) => switch (shape) {
  Circle(r: var r) when r > 100 => 'huge circle',
  Circle(r: var r)              => 'circle',
  Square(side: var s)           => 'square',
};
```

![A switch matches an object's type, destructures its fields, and can add a guard](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/records-patterns/fig-3.png)
*One switch: match the type, pull out the fields, and filter with a guard.*

## Patterns in `if`: validate and unpack in one line

The `if-case` form is great for checking a shape and unpacking it together — very handy for JSON:

```dart
final json = {'name': 'Oguz', 'age': 30};

if (json case {'name': String name, 'age': int age}) {
  print('$name is $age'); // only runs if the shape AND types match
}
```

If the map has a `name` that is a `String` and an `age` that is an `int`, the pattern matches, and `name` and `age` are ready to use. If not, the block is skipped. No manual null checks, no casting.

## How I would answer this in an interview

> "Records let me bundle values without a class — either positional, accessed with `$1` and `$2`, or named, which is clearer. Their best use is returning more than one value from a function, and they have value equality for free. Patterns are the other side: they destructure a value into variables. I use them to unpack a record in one line, and especially in a `switch`, where I can match an object's type and pull out its fields at once — which works great with sealed classes for exhaustive switches. I can add a `when` guard for extra conditions, and I use `if-case` to validate and unpack something like a JSON map in a single line."

## Key points

- **Records** group values with no class — positional (`$1`) or named (`.x`).
- Records have **value equality for free**.
- Best use: **return multiple values** from a function.
- **Patterns** destructure values into variables — in an assignment, a `switch`, or an `if`.
- In a `switch`, patterns **match a type and unpack fields together** (great with sealed classes).
- Add conditions with a **`when` guard**; use **`if-case`** to validate and unpack (e.g. JSON) in one line.

---

*Thanks for reading. If this helped, follow for more simple Dart & Flutter guides.*
