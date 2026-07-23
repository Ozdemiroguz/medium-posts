# Dart & Flutter Deep Dives

Clear, example-driven articles on the Dart and Flutter topics that come up most in real work — and in interviews. Each post explains one concept in plain English, with diagrams and interview-ready answers.

Written and published on Medium. This repo holds the source and the images.

## Posts

| Post | What it covers |
|------|----------------|
| [const vs final vs late](dart-const-final-late.md) | Compile-time vs runtime, canonicalization, lazy `late`, and the common traps |
| [`==` vs `identical`](dart-equality.md) | Identity vs value equality, the `hashCode` contract, records, and Flutter rebuilds |
| [Sealed Classes](dart-sealed-class.md) | Closed type sets, exhaustive `switch`, and `sealed` vs `abstract` |
| [Isolates](dart-isolates.md) | True parallelism, message passing, `compute()`, and isolate vs `async`/`await` |
| [Future vs Stream](dart-future-vs-stream.md) | One value vs many, the event loop, microtasks vs events, single vs broadcast |
| [WebSockets in Flutter](flutter-websocket.md) | REST vs WebSocket, the handshake, the connection lifecycle, reconnect & heartbeat |
| [Null Safety](dart-null-safety.md) | Non-nullable types, `?.` `??` `!` `??=`, `late`, and sound null safety |
| [Mixin vs Abstract vs Interface](dart-mixin-abstract-interface.md) | `extends` vs `implements` vs `with`, and when to use each |
| [The Three Trees](flutter-three-trees.md) | Widget, Element, RenderObject — and why `const`, keys, and cheap rebuilds work |
| [Records & Pattern Matching](dart-records-patterns.md) | Dart 3 records, destructuring, pattern `switch`, guards, and `if-case` |
| [Signals in Flutter](flutter-signals.md) | Fine-grained, synchronous reactive state — `signal`, `computed`, `effect`, `Watch`, `batch` |
| [State Management](flutter-state-management.md) | setState vs Bloc/Cubit vs Riverpod, and how to choose |
| [Dependency Injection](flutter-dependency-injection.md) | DI with `get_it` — loose coupling, easy tests, register/get patterns |
| [Event Tracking & Attribution](flutter-event-tracking.md) | Facebook/TikTok/AppsFlyer SDKs, standard vs custom events, ATT & SKAdNetwork |
| [Analytics & Crash Reporting](flutter-analytics-crash.md) | Firebase/Sentry, catching every error source, fatal vs non-fatal, breadcrumbs |
| [Flutter 3.44 & Dart 3.12](flutter-3-44-whats-new.md) | Dot shorthands, private named params, Impeller-only Android, AI-agentic tooling |

## About the images

Every diagram is a self-contained PNG under [`images/`](images/). Posts reference them by their raw GitHub URL, so the article renders correctly anywhere — including when pasted straight into the Medium editor.

## Author

**Oğuzhan Özdemir** — Flutter & Node.js developer.
📦 [pub.dev/publishers](https://pub.dev) · ⭐ [github.com/Ozdemiroguz](https://github.com/Ozdemiroguz)

## License

Articles and images © Oğuzhan Özdemir. Code snippets are free to use.
