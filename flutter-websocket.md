# WebSockets in Flutter: Real-Time, Two-Way Connections Explained Simply

*What a WebSocket is, how it differs from HTTP/REST, how the handshake works, and the connection lifecycle you need to know.*

---

REST is great when the app asks and the server answers. But what about a chat app, live prices, or a ride you track on a map? There, the **server needs to push data to the app** the moment it changes. That is what WebSockets are for. Let me explain them simply.

## REST vs WebSocket: the core difference

With **REST/HTTP**, the app asks and the server answers — one request, one response, then the connection closes. If you want fresh data, you ask again and again. This is called **polling**, and it wastes time and battery.

With a **WebSocket**, you open **one connection that stays open**. After that, both sides can send messages at any time — no need to ask first. The server can **push** data to the app instantly.

![REST is ask-and-answer polling; WebSocket is one open, two-way connection](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/websocket/fig-1.png)
*REST keeps asking. A WebSocket stays open, and the server can push at any time.*

Quick comparison:

- **REST** → request → response → closed. One direction at a time. Good for normal data.
- **WebSocket** → stays open, both directions, server can push. Good for real-time data.

## How the connection starts: the handshake

A WebSocket does not start as its own protocol. It **starts as a normal HTTP request** and then asks to "upgrade".

1. The app sends an HTTP GET with a special header: `Upgrade: websocket`.
2. If the server agrees, it replies with status **`101 Switching Protocols`**.
3. From that moment, the same connection becomes a WebSocket — open and two-way.

![The WebSocket handshake upgrades an HTTP request with a 101 Switching Protocols response](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/websocket/fig-2.png)
*It begins as HTTP, then upgrades to a WebSocket with a 101 response.*

This is why WebSockets work over the same ports as web traffic (80 and 443). The secure version is **`wss://`**, which runs over TLS — always use it in production.

## The connection lifecycle

A WebSocket moves through a few clear states. Knowing them helps you handle errors and clean up correctly.

![The WebSocket lifecycle — connecting, open, messages, closing, closed](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/websocket/fig-3.png)
*connect → open → messages both ways → close (or error). Each has an event.*

- **connecting** → you called connect; the handshake is happening.
- **open** → the connection is ready; now messages flow both ways.
- **message** → data arrives (or you send data). This fires many times.
- **close / error** → the connection ends, cleanly (a close code) or with an error.

**Close codes** tell you *why* it ended: `1000` normal, `1001` going away, `1006` abnormal (the connection dropped). `1006` usually means you should reconnect.

## Staying alive: reconnect and heartbeat

Real networks drop. Two patterns keep a WebSocket healthy:

- **Reconnect with backoff** — if the connection drops, don't retry instantly in a tight loop. Wait a bit longer each time (1s, 2s, 4s…). This is **exponential backoff**, and it avoids hammering the server.
- **Heartbeat (ping/pong)** — send a small "ping" now and then. If you don't get a "pong" back, the connection is dead even if it looks open, so you reconnect. This catches "silent" drops.

## In Flutter

The common package is `web_socket_channel`. It exposes the connection as a **Stream**, so it fits Dart perfectly — you `listen` to incoming messages and call `.sink.add(...)` to send.

```dart
final channel = WebSocketChannel.connect(Uri.parse('wss://example.com/socket'));

// receive (this is a stream — remember single vs broadcast!)
channel.stream.listen((message) => print('Got: $message'));

// send
channel.sink.add('hello');

// clean up when done — very important
channel.sink.close();
```

One key point for Flutter: **close the socket when you're done** — for example in `dispose()` when a screen closes. If you don't, you leak the connection and keep receiving data in the background.

## Putting it together: a small socket service

Here is a compact service that shows the real patterns in one place — listening, sending, heartbeat, reconnect with backoff, and cleanup:

```dart
class SocketService {
  WebSocketChannel? _channel;
  StreamSubscription? _sub;
  Timer? _heartbeat;
  int _retryDelay = 1; // seconds

  void connect() {
    _channel = WebSocketChannel.connect(Uri.parse('wss://example.com/socket'));

    _sub = _channel!.stream.listen(
      _onMessage,
      onError: (_) => _reconnect(),
      onDone: _reconnect,        // fires when the socket closes
    );

    _startHeartbeat();
    _retryDelay = 1;             // reset backoff after a good connect
  }

  void _onMessage(dynamic data) {
    if (data == 'pong') return;  // heartbeat reply, ignore
    print('Got: $data');
  }

  void send(String msg) => _channel?.sink.add(msg);

  void _startHeartbeat() {
    _heartbeat = Timer.periodic(const Duration(seconds: 15), (_) {
      _channel?.sink.add('ping');
    });
  }

  void _reconnect() {
    _cleanUp();
    Future.delayed(Duration(seconds: _retryDelay), connect);
    _retryDelay = (_retryDelay * 2).clamp(1, 32); // exponential backoff, capped at 32s
  }

  void _cleanUp() {
    _heartbeat?.cancel();
    _sub?.cancel();
    _channel?.sink.close();
  }

  void dispose() => _cleanUp(); // call this from State.dispose()
}
```

A few things to notice:

- `onDone` runs when the socket closes, so both `onError` and `onDone` trigger a reconnect.
- `_retryDelay` doubles each time and is capped, so retries slow down instead of hammering the server.
- After a successful connect, the delay resets to 1 second.
- Everything is cancelled in `_cleanUp()` — the timer, the subscription, and the sink — so nothing leaks.

You can also read the close code after the socket ends, to decide what to do:

```dart
final code = _channel?.closeCode; // e.g. 1000 normal, 1006 dropped
```

## How I would answer this in an interview

> "A WebSocket is a persistent, two-way connection. Unlike REST, where the app asks and the server answers and then closes, a WebSocket stays open so the server can push data at any time — which is what you want for chat or live updates. It starts as an HTTP request with an Upgrade header, and the server replies with 101 Switching Protocols, then the same connection becomes a WebSocket. In production I use `wss` over TLS. The lifecycle is connect, open, messages, then close or error, and close codes like 1006 tell me the connection dropped so I should reconnect — usually with exponential backoff, plus a ping/pong heartbeat to detect silent drops. In Flutter I use `web_socket_channel`, which gives me a stream, and I always close the socket in dispose to avoid leaks."

## Key points

- **REST** = ask/answer, then close. **WebSocket** = one open, two-way connection; the server can push.
- Handshake: HTTP request with `Upgrade` → **`101 Switching Protocols`** → WebSocket.
- Use **`wss://`** (TLS) in production.
- Lifecycle: **connect → open → message → close/error.** Close code `1006` = dropped.
- Keep it alive with **reconnect (exponential backoff)** and a **heartbeat (ping/pong)**.
- In Flutter: `web_socket_channel` gives a **Stream**; always **close in `dispose()`**.

---

*Thanks for reading. If this helped, follow for more simple Dart & Flutter guides.*
