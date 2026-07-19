# Dependency Injection in Flutter with get_it: Looser Code, Easier Tests

*A simple guide to what dependency injection really means, why it makes testing easy, and how `get_it` gives it to you in a few lines.*

---

"Dependency injection" sounds heavy, but the idea is simple, and it's a favorite interview topic because it touches testing, clean architecture, and SOLID all at once. Once you see it, you'll want it in every project. Let's build it up slowly.

## The problem: classes that build their own dependencies

Look at this class. It creates the thing it depends on, inside itself:

```dart
class LoginService {
  final api = ApiClient(); // built inside — tightly coupled

  Future<User> login(...) => api.post(...);
}
```

This works, but it has two problems:

1. **It's tightly coupled.** `LoginService` is locked to that exact `ApiClient`. You can't swap it.
2. **It's hard to test.** In a test you don't want a real network call, but you can't replace `api` — it's created inside.

## The idea: pass dependencies in from outside

**Dependency injection (DI)** just means: don't build your dependencies inside a class — **receive them from outside.**

```dart
class LoginService {
  final ApiClient api;
  LoginService(this.api); // given from outside — injected
}
```

That's the whole concept. The class no longer *creates* its dependency; it *asks for* it. This one change makes the class **loosely coupled** and **easy to test**.

![Without DI a class builds its own dependency; with DI it receives one](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/dependency-injection/fig-1.png)
*Without DI: the class is glued to a concrete dependency. With DI: you hand it in, and can swap it.*

## Why this makes testing easy

Because the dependency comes from outside, in a test you can pass in a **fake** one:

```dart
class FakeApiClient implements ApiClient { /* returns test data */ }

final service = LoginService(FakeApiClient()); // no real network!
```

This is where it connects to interfaces: if `LoginService` depends on an **abstraction** (an interface) instead of a concrete class, you can give it the real one in the app and a fake one in tests. This is the "D" in SOLID — *depend on abstractions, not concretions*.

## get_it: a place to find your dependencies

Injecting by hand is fine for small apps, but soon you're passing objects through many constructors. `get_it` solves this. It's a **service locator** — a central place where you **register** your objects once, then **get** them anywhere.

```dart
import 'package:get_it/get_it.dart';

final getIt = GetIt.instance;

void setup() {
  getIt.registerSingleton<ApiClient>(ApiClient());
  getIt.registerFactory<LoginService>(() => LoginService(getIt<ApiClient>()));
}

// anywhere in the app:
final service = getIt<LoginService>();
```

You set it up once at startup, then ask `getIt` for what you need. No passing objects through ten widgets.

## Singleton vs lazy singleton vs factory

`get_it` gives you three ways to register, and interviewers like to hear the difference:

![get_it registration types — singleton, lazy singleton, and factory](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/dependency-injection/fig-2.png)
*singleton: one now. lazySingleton: one, on first use. factory: a new one every time.*

- **`registerSingleton`** → creates the object **now**, and always returns the **same** one.
- **`registerLazySingleton`** → creates it **the first time** you ask, then always the same one. Good for expensive objects you might not need.
- **`registerFactory`** → creates a **new** object **every time** you ask. Good for things that shouldn't be shared, like a form controller.

## Depend on the abstraction, swap the implementation

The real power shows when you register an **interface** and give it a concrete implementation — then swap it for tests:

```dart
// register the interface → real implementation
getIt.registerSingleton<AuthRepository>(AuthRepositoryImpl());

// in a test, register a fake for the same interface
getIt.registerSingleton<AuthRepository>(FakeAuthRepository());
```

Your app code only ever asks for `getIt<AuthRepository>()`. It never knows or cares which implementation it gets. That's loose coupling in action.

![The app depends on an interface; get_it swaps real or fake behind it](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/dependency-injection/fig-3.png)
*The app asks for the interface; get_it hands back the real one in production or a fake one in tests.*

## How I would answer this in an interview

> "Dependency injection means a class receives its dependencies from outside instead of creating them itself. That makes it loosely coupled and easy to test, because in a test I can pass a fake — especially if the class depends on an interface rather than a concrete type, which is the 'D' in SOLID. In Flutter I use get_it as a service locator: I register my objects once at startup and get them anywhere, so I don't pass them through many constructors. I use registerSingleton for one shared instance, registerLazySingleton for one created on first use, and registerFactory for a new instance each time. And I usually register an interface with a real implementation, so I can swap a fake one in tests."

## Key points

- **DI** = a class **receives** its dependencies instead of **creating** them.
- Result: **loose coupling** + **easy testing** (pass a fake).
- Depend on an **interface**, not a concrete class — the "D" in SOLID.
- **`get_it`** = a service locator: **register** once, **get** anywhere.
- **`registerSingleton`** (one now) · **`registerLazySingleton`** (one on first use) · **`registerFactory`** (new each time).

---

*Thanks for reading. If this helped, follow for more simple Dart & Flutter guides.*
