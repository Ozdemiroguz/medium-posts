# Widget, Element, RenderObject: The Three Trees Behind Every Flutter App

*A deep but simple look at what really happens when Flutter builds your UI — and why `const`, keys, and cheap rebuilds all make sense once you see it.*

---

Most Flutter developers think in one tree: the widget tree. But Flutter actually keeps **three** trees working together. Once you understand them, a lot of "magic" — why rebuilds are cheap, why `const` helps, why keys matter, why state survives a rebuild — suddenly becomes obvious. This is a favorite deep interview topic, so let's go slow and clear.

## The problem the three trees solve

Your `build()` method runs **a lot** — often 60 times per second during an animation. If every rebuild threw away the real UI and made a new one, Flutter would be far too slow.

So Flutter splits the job into three layers, each with a different lifetime and cost:

- **Widget** — a cheap, throwaway *description* of the UI.
- **Element** — a *living* object that stays alive across rebuilds and holds state.
- **RenderObject** — the *expensive* object that actually measures, lays out, and paints.

![The three trees — Widget config, Element the living bridge, RenderObject the painter](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/three-trees/fig-1.png)
*Widget describes, Element remembers, RenderObject draws.*

## 1. Widget — a description, not the real thing

A widget is **immutable** and cheap to create. It is just a *configuration*: "I want a red container that is 100 wide." It holds no state and does no drawing. When you call `setState`, Flutter throws away the old widgets and builds new ones. This is fine — widgets are cheap.

Think of a widget as a **blueprint**, not the building.

## 2. Element — the living bridge

The **Element** is where the real work of continuity happens. For each widget, Flutter creates an Element, and — this is the key part — **the Element stays alive across rebuilds.** It is the thing that actually sits in the tree at a position.

The Element does two important jobs:

- It **holds the State** of a `StatefulWidget`. This is *why your state survives a rebuild* — the widget is replaced, but the Element (and its State) lives on.
- It is the **bridge** between the throwaway widget and the expensive RenderObject.

When a rebuild happens, the Element looks at the new widget and asks: *"Is this the same kind of widget as before?"* (same `runtimeType` and same `key`). If yes, it **keeps itself and its RenderObject**, and just updates them with the new config. If no, it throws the old one away and makes a new Element.

![On rebuild the Element is reused and just updates its RenderObject, instead of recreating it](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/three-trees/fig-2.png)
*A new widget appears, but the Element is reused — it just updates the RenderObject.*

This "reuse the Element, update the config" step is **why rebuilds are cheap.** The expensive RenderObject is not recreated; it is only updated.

## 3. RenderObject — the one that actually draws

The **RenderObject** does the heavy work: layout (measuring size and position), painting, and hit-testing (which widget did you tap?). Creating one is expensive, so Flutter tries hard **not** to recreate them — it updates them through the Element instead.

## Now everything makes sense

Once you see the three trees, three "tricks" stop being magic:

- **Why `const` helps performance.** A `const` widget is the *same instance* every rebuild. When the Element compares old and new and sees they are identical, it knows nothing changed and **skips the update entirely**. Free performance.

- **Why keys matter.** If you reorder items in a list, the widgets change position. By default the Element matches by type, so it can attach the wrong state to the wrong item. A **key** gives the Element a stable identity, so it matches the right widget and **keeps the right state**.

- **Why state survives a rebuild.** Because state lives in the **Element**, not the widget. The widget is replaced every build; the Element (and its State) stays.

![Why const, keys, and surviving state all follow from the Element being reused](https://raw.githubusercontent.com/Ozdemiroguz/medium-posts/main/images/three-trees/fig-3.png)
*const skips updates, keys fix identity, and state lives in the Element.*

## How I would answer this in an interview

> "Flutter has three trees. Widgets are immutable, cheap descriptions of the UI, and they get rebuilt often. Elements are the living objects that stay alive across rebuilds — each one sits at a position in the tree, holds the State of a StatefulWidget, and bridges the widget to the RenderObject. RenderObjects are the expensive objects that do layout and painting. On a rebuild, Flutter reuses the Element if the new widget has the same type and key, and just updates the RenderObject instead of recreating it — that's why rebuilds are cheap. It also explains const, which lets the Element skip updates because the widget instance is identical, and keys, which give the Element a stable identity so the right state stays with the right widget."

## Key points

- Three trees: **Widget** (config, throwaway), **Element** (living, holds state, the bridge), **RenderObject** (layout + paint, expensive).
- On rebuild, Flutter **reuses the Element** if type and key match, and just **updates** the RenderObject → cheap rebuilds.
- **`const`** → identical widget → Element skips the update.
- **Keys** → stable Element identity → correct state stays with the correct widget.
- **State survives rebuilds** because it lives in the Element, not the widget.

---

*Thanks for reading. If this helped, follow for more simple Dart & Flutter guides.*
