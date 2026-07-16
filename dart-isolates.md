# Isolates in Dart: True Parallelism Without Shared Memory

*A simple guide to isolates — how they run in parallel, how they pass data, and when to use them instead of async/await.*

---

Dart runs your code on a single thread. So how do you do heavy work — parsing a huge JSON, resizing an image, encryption — without freezing the UI? The answer is **isolates**. But isolates confuse many developers, because they don't work like threads in other languages. Let me make it simple.

## First: async/await is NOT parallelism

This is the most common mistake. `async`/`await` does **not** run code in parallel. It runs on the **same thread**, and it only helps while you are **waiting** — for a network call, a file, a timer.

Think of one chef in a kitchen. While the oven heats, the chef chops vegetables. One person, switching between tasks while waiting. That is `async`/`await` — great for I/O, useless for heavy CPU work. If the chef has to peel 1000 potatoes (real work, no waiting), one chef is still slow.

For real CPU work that has no "waiting", you need a **second chef**. That is an isolate.

![async/await vs isolate — one worker switching tasks vs two workers in parallel](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/isolates/fig-1.png)
*async/await is one worker filling the waiting time. An isolate is a second worker running at the same time.*

## What is an isolate?

An isolate is an independent worker with its **own memory** and its **own event loop**. Two isolates run truly at the same time, on different CPU cores.

The key rule: **isolates do not share memory.** In most languages, threads share memory, so you need locks to avoid race conditions. Dart avoids this whole problem. Each isolate is fully separate, so there are **no locks and no race conditions**.

## How do isolates talk? Message passing

If they don't share memory, how do they exchange data? They **send messages** through ports:

- A **`ReceivePort`** listens for incoming messages.
- A **`SendPort`** sends messages to a `ReceivePort`.

And here is the important detail: **the message is copied, not shared.** When isolate A sends a list to isolate B, B gets its **own copy**. Changing it in B does not affect A.

![Isolates pass copied messages through SendPort and ReceivePort](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/isolates/fig-2.png)
*No shared memory. Data is copied from one isolate to the other through ports.*

```dart
final receivePort = ReceivePort();
await Isolate.spawn(worker, receivePort.sendPort);

// worker runs in a separate isolate
void worker(SendPort sendPort) {
  final result = doHeavyWork();
  sendPort.send(result); // this result is COPIED back
}

receivePort.listen((message) {
  print('Got result: $message');
});
```

Because data is copied, there is a cost: sending a very large object takes time and memory. For small results this is fine; for huge data it is something to keep in mind.

## The easy way: `compute()`

You rarely need to set up ports by hand. For a simple "run this function in the background and give me the result", Flutter gives you `compute()`:

```dart
// Runs parseJson in a separate isolate, returns the result
final users = await compute(parseJson, jsonString);
```

`compute()` spawns an isolate, runs your function, sends the result back, and closes the isolate. One line. Use this for most cases. Use manual `Isolate.spawn` only when you need a long-running isolate that receives many messages.

## When should you use an isolate?

This is the interview question. The rule is simple:

![Decision guide — CPU-heavy work uses isolate, waiting uses async/await](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/isolates/fig-3.png)
*Heavy CPU work → isolate. Waiting on I/O → async/await.*

- **Heavy CPU work** (parsing big JSON, image/video processing, encryption, compression) → **isolate** (or `compute`). This work has no "waiting"; it just needs another core.
- **Waiting on I/O** (network requests, database, files) → **async/await**. There is nothing to compute; you are just waiting, so a second thread would be wasted.

A quick test: *"Is my code busy calculating, or just waiting?"* Busy calculating → isolate. Just waiting → async/await.

## Limits to remember

- Messages must be **sendable**: simple data, lists, maps, strings, numbers. You can't send things like open sockets or most closures.
- Copying large data has a **cost** (time + memory).
- Isolates have a **startup cost**, so don't spawn one for tiny work — the overhead can be bigger than the work itself.

## How I would answer this in an interview

> "Dart is single-threaded, so async/await is concurrency, not parallelism — it only helps while waiting on I/O, on the same thread. For heavy CPU work, I use an isolate, which is a separate worker with its own memory that runs on another core. Isolates don't share memory, so there are no locks or race conditions. They communicate by message passing through SendPort and ReceivePort, and the messages are copied, not shared. In practice I usually use `compute()` for a one-off heavy task, like parsing a large JSON, and I keep isolates for CPU work — not for network calls, where async/await is enough."

## Key points

- `async`/`await` = concurrency on one thread; only helps while **waiting**.
- Isolate = true parallelism, own memory, another core.
- Isolates **don't share memory** → no locks, no race conditions.
- They talk by **message passing** (SendPort / ReceivePort); messages are **copied**.
- Use `compute()` for simple background tasks.
- Rule: **busy calculating → isolate. Just waiting → async/await.**

---

*Thanks for reading. If this helped, follow for more simple Dart & Flutter guides.*
