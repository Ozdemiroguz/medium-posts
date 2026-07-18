# Signals in Flutter: Fine-Grained, Synchronous Reactive State

*A simple guide to the `signals` package — reactive values that update synchronously, track their own dependencies, and rebuild only what changed.*

---

Streams are great for async sequences of events. But for **app state** — a counter, a form, a filter — they can feel heavy, and they update through the event loop. The `signals` package offers another way: **reactive values that update synchronously** and rebuild only the exact widget that depends on them. Let's see how it works.

> Note: `signals` is a package by Rody Davis. Add it with `flutter pub add signals`.

## The core idea: a value that others can watch

A **signal** is a box that holds a value. When the value changes, anything that depends on it reacts — automatically. You don't wire up listeners by hand; the system tracks the dependencies for you.

There are three primitives:

- **`signal`** — a writable reactive value.
- **`computed`** — a value derived from other signals; it recomputes itself.
- **`effect`** — a side effect that runs whenever its signals change.

![signal feeds computed feeds effect, with automatic dependency tracking](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/signals/fig-1.png)
*A signal feeds a computed value, which feeds an effect — dependencies are tracked automatically.*

```dart
import 'package:signals/signals.dart';

final count = signal(0);                    // writable
final doubled = computed(() => count() * 2); // derived, auto-updates

effect(() {
  print('Count: ${count()}, Doubled: ${doubled()}');
});

count.value++; // the effect runs again automatically → Count: 1, Doubled: 2
```

Notice what you *didn't* write: no listeners, no `addListener`, no manual "when count changes, update doubled". You read a signal, and the system remembers that you depend on it. That is **automatic dependency tracking**.

## "Synchronous" — and no microtasks

This is the line from the comment that started this post: *"No microtasks will be harmed."*

A `Stream` is asynchronous — a new value is delivered later, scheduled through the event loop (remember microtasks vs events?). A **signal updates synchronously** — the moment you write `count.value++`, the change is applied and its dependents update right away. There is no scheduling, no waiting a turn. For plain state, that is simpler and more predictable.

## In Flutter: rebuild only what changed

The best part is how little rebuilds. You wrap the part of the UI that uses a signal in a **`Watch`** widget. When the signal changes, **only that `Watch` rebuilds** — not the whole widget tree.

![Watch rebuilds only the widget that reads the signal, not the whole tree](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/signals/fig-2.png)
*Only the Watch that reads the signal rebuilds — the rest of the tree stays still.*

```dart
import 'package:signals/signals_flutter.dart';

final counter = signal(0);

// ...
Column(
  children: [
    Watch((context) => Text('Counter: $counter')), // only this rebuilds
    ElevatedButton(
      onPressed: () => counter.value++,
      child: const Text('Increment'),
    ),
  ],
);
```

This is called **fine-grained** reactivity: the update is surgical. Compare that to `setState`, which rebuilds the whole `build` method.

## Grouping updates with `batch`

If you change several signals at once, you don't want the effects to run after each one. Wrap them in **`batch`**, and the effects run just once, after all the writes:

```dart
final name = signal('Jane');
final surname = signal('Doe');
final fullName = computed(() => '${name.value} ${surname.value}');

effect(() => print(fullName.value)); // prints "Jane Doe"

batch(() {
  name.value = 'Foo';
  surname.value = 'Bar';
}); // effect runs once → "Foo Bar"
```

## Signals vs Streams: which one?

They are not rivals — they solve different problems.

![Signals for synchronous state, Streams for async event sequences](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/signals/fig-3.png)
*Signals model state that changes. Streams model a sequence of events over time.*

- **Signals** → observable **state**: a counter, a selected filter, a form value, derived values. Synchronous, fine-grained rebuilds, little boilerplate.
- **Streams** → **async sequences of events** over time: a WebSocket feed, user input events, anything that "keeps sending".

A good rule: if you're asking *"what is the current value, and what depends on it?"* → a signal. If you're asking *"what is the next event, and when?"* → a stream.

## How I would answer this in an interview

> "Signals are a fine-grained reactivity system. A signal holds a value, a computed derives from other signals, and an effect runs when its signals change — and dependencies are tracked automatically, so I don't wire up listeners. Unlike streams, signals update synchronously, with no event-loop scheduling, which is simpler for plain state. In Flutter I wrap the UI in a Watch, and only that part rebuilds when the signal changes, which is more surgical than setState. I'd still use streams for async event sequences like a socket, but for observable app state, signals are lighter."

## Key points

- **`signal`** = writable reactive value; **`computed`** = derived value; **`effect`** = reaction.
- **Automatic dependency tracking** — no manual listeners.
- **Synchronous** updates — no microtask/event-loop scheduling.
- In Flutter, **`Watch`** rebuilds only the widget that reads the signal (fine-grained).
- **`batch`** groups multiple writes into one update.
- **Signals for state; Streams for async event sequences.**

---

*Thanks for reading. If this helped, follow for more simple Dart & Flutter guides.*
