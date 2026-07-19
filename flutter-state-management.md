# State Management in Flutter: setState vs Bloc/Cubit vs Riverpod

*A simple map of the main options — what each one is for, and how to choose without overthinking it.*

---

"Which state management do you use, and why?" is one of the most common Flutter interview questions. The trap is listing names. The real answer is understanding **what problem each one solves**, so you can pick the right tool instead of the trendy one. Let's map them out.

## First: what is "state", really?

**State** is any data that can change and that the UI needs to show — a counter, a loading flag, a list of messages, the logged-in user. State management is about three things:

1. **Where** does the state live?
2. **How** do widgets read it?
3. **When** does the UI rebuild?

Every tool below answers those three questions differently.

## setState: local state, one widget

`setState` keeps state **inside a single widget** and rebuilds that widget. It's built in, simple, and perfect for small, local things — a checkbox, an animation flag, a text field's value.

```dart
int _count = 0;
void _increment() => setState(() => _count++); // rebuilds this widget
```

The limit: the state lives in **one place**. If a widget far away also needs it, `setState` can't share it. You'd have to pass it down manually through many constructors ("prop drilling").

![setState keeps state local; shared state must be lifted out of the widget](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/state-management/fig-1.png)
*setState is local to one widget. Shared state needs to live outside the widget tree.*

## Cubit and Bloc: business logic, outside the UI

**Bloc** (and its simpler sibling **Cubit**) move state and logic **out of the widget** into a separate class. The UI just sends inputs and listens for new states. This makes the logic easy to test and reuse.

- **Cubit** — the simple one. You call a **method**, and it **emits** a new state.
- **Bloc** — you add an **event**, it maps that event to a new **state**. More structure, good for complex flows with a clear history of events.

![Cubit calls a method to emit a state; Bloc maps an event to a state](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/state-management/fig-2.png)
*Cubit: method → emit state. Bloc: event → state. Both keep logic out of the UI.*

```dart
// Cubit — simple
class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);
  void increment() => emit(state + 1); // method → new state
}

// Bloc — event-driven
class CounterBloc extends Bloc<CounterEvent, int> {
  CounterBloc() : super(0) {
    on<Increment>((event, emit) => emit(state + 1)); // event → new state
  }
}
```

The UI reacts with a `BlocBuilder`, which rebuilds when the state changes. Under the hood, Bloc is built on **streams** — a new state is a new value on a stream.

## Riverpod: providers, safe and flexible

**Riverpod** holds state in **providers** — objects you can read from anywhere, without needing `BuildContext`. It's compile-safe (mistakes are caught by the compiler, not at runtime) and makes derived state and dependencies easy.

```dart
final counterProvider = StateProvider<int>((ref) => 0);

// read it anywhere
final count = ref.watch(counterProvider);
ref.read(counterProvider.notifier).state++;
```

Riverpod is a modern, flexible choice for whole-app state, and it avoids some old problems of the original Provider package (like needing the right context).

## How to choose

![Decision guide — local UI uses setState, app state uses Riverpod/Provider, complex flows use Bloc](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/state-management/fig-3.png)
*Local → setState. Shared app state → Riverpod/Provider. Complex, event-driven flows → Bloc.*

- **Small, local UI state** (a toggle, a form field) → **`setState`**. Don't add a library for this.
- **Shared app state**, read in many places, medium complexity → **Riverpod** (or Provider).
- **Complex, event-driven logic** with clear states and history, team wants structure → **Bloc/Cubit**.

The honest truth: there is no single "best". The best answer in an interview is *"it depends on the size and complexity of the state"* — and then explain the trade-offs, like you just read.

## How I would answer this in an interview

> "State is any data that changes and that the UI shows, and state management decides where it lives, how widgets read it, and when they rebuild. I use setState for small local state, like a toggle — it's simple and built in. For shared app state I reach for Riverpod, because it's compile-safe and I can read providers without context. For complex, event-driven logic I use Bloc or Cubit, which move the logic out of the UI into a testable class — Cubit calls a method to emit a state, and Bloc maps an event to a state, built on streams. So my answer is really 'it depends on the complexity', and I pick the lightest tool that fits."

## Key points

- **State** = data that changes and the UI shows. Manage **where it lives, how it's read, when it rebuilds.**
- **`setState`** → local state, one widget. Simple, built in, can't share.
- **Cubit** → method → emit state (simple). **Bloc** → event → state (structured, stream-based). Logic outside the UI, testable.
- **Riverpod** → providers, compile-safe, no context needed; great for app state.
- Choose by **size and complexity** — the lightest tool that fits.

---

*Thanks for reading. If this helped, follow for more simple Dart & Flutter guides.*
