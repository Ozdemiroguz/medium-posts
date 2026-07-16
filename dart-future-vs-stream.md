# Future vs Stream in Dart: One Value vs Many, and the Event Loop

*A simple guide to Future and Stream — how they differ, how the event loop orders your code, and why microtasks run before events.*

---

Almost every async bug I have seen comes from one of two confusions: mixing up `Future` and `Stream`, or not knowing the order the event loop runs things. Let me make both clear.

## Future vs Stream: the core idea

- A **`Future`** gives you **one value, later**. Like ordering a coffee — you wait once, you get one coffee.
- A **`Stream`** gives you **many values, over time**. Like a conveyor belt — items keep coming until it ends.

![Future is one value later; Stream is many values over time](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/future-vs-stream/fig-1.png)
*A Future completes once. A Stream delivers many values until it closes.*

```dart
// Future — one value
Future<int> fetchCount() async => 42;
final count = await fetchCount(); // wait once, get one value

// Stream — many values
Stream<int> ticks() async* {
  for (var i = 1; i <= 3; i++) yield i;
}
await for (final t in ticks()) print(t); // 1, 2, 3
```

`async` + `await` is for a `Future`. `async*` + `yield` is for a `Stream`. `await for` reads a stream value by value.

## When do you use each?

- **`Future`** → a single result: an HTTP GET, reading a file, one database query.
- **`Stream`** → a series of events: a WebSocket, button clicks, location updates, a live database query.

Rule of thumb: **"one answer" → Future. "keeps sending" → Stream.**

## The event loop: how Dart orders async code

Dart runs on a single thread with an **event loop**. The event loop has two queues, and the order matters:

1. First, all **synchronous** code runs.
2. Then the **microtask queue** runs — and it fully empties before anything else.
3. Then **one** item from the **event queue** runs (I/O, timers, `Future` callbacks).
4. Repeat.

The key rule: **microtasks always run before events.**

![The event loop runs sync code, then drains the microtask queue, then the event queue](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/future-vs-stream/fig-2.png)
*Sync first, then all microtasks, then the event queue — one round at a time.*

This explains a classic interview question. What does this print?

```dart
print('1');
Future(() => print('2'));                 // event queue
Future.microtask(() => print('3'));       // microtask queue
print('4');
```

The answer is **1, 4, 3, 2**:

- `1` and `4` are synchronous → they run first, in order.
- `3` is a microtask → runs before any event.
- `2` is a normal `Future` → goes to the event queue → runs last.

If you can explain this, you understand the event loop.

## Single vs broadcast streams

There are two kinds of streams, and mixing them up is a common error:

- A **single-subscription** stream (the default) allows **one** listener. If you listen twice, it throws.
- A **broadcast** stream allows **many** listeners at the same time.

![Single-subscription allows one listener; broadcast allows many](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/future-vs-stream/fig-3.png)
*A single-subscription stream has one listener. A broadcast stream has many.*

```dart
final controller = StreamController<int>();            // single
final controller2 = StreamController<int>.broadcast();  // many listeners
```

Use single-subscription for a one-time flow (like reading a file). Use broadcast when several parts of the app must react to the same events (like a shared connection status).

## How I would answer this in an interview

> "A Future is a single value that arrives later, and a Stream is a sequence of values over time. I use `async`/`await` for a Future — like an HTTP call — and `async*`/`yield` with `await for` for a Stream, like a WebSocket. Dart is single-threaded with an event loop, and the important rule is that microtasks run before events: sync code first, then the whole microtask queue, then one event at a time. So `Future.microtask` runs before a normal `Future`. And I always remember that a normal stream is single-subscription — I use a broadcast stream when I need more than one listener."

## Key points

- **Future** = one value, later. **Stream** = many values, over time.
- `async`/`await` for Futures; `async*`/`yield` + `await for` for Streams.
- Event loop order: **sync → all microtasks → one event → repeat.**
- **Microtasks run before events** (`Future.microtask` before a normal `Future`).
- Streams are **single-subscription** by default; use **broadcast** for many listeners.

---

*Thanks for reading. If this helped, follow for more simple Dart & Flutter guides.*
