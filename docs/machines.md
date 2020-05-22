---
id: machines
title: Machines
---

Here is a minimal machine component in Baahu:

```tsx
import { b, machine } from "baahu";

const Toggle = machine({
  id: "toggle",
  initial: "inactive",
  context: () => ({}),
  when: {
    inactive: { on: {} },
    active: {},
  },
});
```

This guide will teach you how to read and write machine components.

## Props & id

In Baahu, props provided to a machine have two uses:

1. Providing a stable identity

2. Initializing

### Singleton Machines

A large amount of components in web applications should only appear once / have one instance.

### Dynamic Machines

#### Derive id

## State and Context

## Lifecycle Hooks

### mount

`mount` is an **optional** method that is called after the first time a machine (by id) has been rendered.

### unmount

`unmount` is an **optional** method that is called after a machine has been removed from the virtual DOM tree.

## Events

To emit an event, use the `emit` function.

```tsx
import { emit } from "baahu";

emit({ type: "TOGGLE", name: "Baahu" }, "toggle");
```

The first argument of `emit` is the event object. The event object must have a `type` property (string) that identifies the event. You can send arbitrary payloads in this object. The second (optional) argument is the id of the machine to target.

### Transitions + Effects

### Targeted

[View full image](/img/targeted-event.svg)

![targeted events](/img/targeted-event.svg)

### Global

[View full image](/img/global-event.svg)

![global events](/img/global-event.svg)

When a global event is emitted, Baahu iterates through the machine registry (map of active machine instances).

If the machine listens to the emitted event type, Baahu will process its transition (`to` state) and effects (`entry`, `exit`, `do`).

(Note: If an effect synchronously causes another event, Baahu will process the resulting transitions before rendering. This ensures that in any given [task](https://developer.mozilla.org/en-US/docs/Web/API/HTML_DOM_API/Microtask_guide#Tasks_vs_microtasks), Baahu only renders virtual nodes once.)

After machines that listen to the event have transitioned, Baahu renders new virtual nodes for those machines, then diffs them against their old virtual nodes, updating the DOM wherever it finds changes.

Read [this section](performance#higher-level-internal-optimizations) if you are interested in how Baahu ensures minimal rerenders on global events.

Baahu rendering is a synchronous process, making it very predictable.

### "Root" listeners

Root listeners allow machines to listen to an event regardless of its state. These can save you from excess boilerplate.

```tsx
import { b, machine } from "baahu";

const Toggle = machine({
  id: "toggle",
  initial: "inactive",
  context: () => ({}),
  on: {
    ALERT: {
      do: () => alert("hi"),
    },
  },
  when: {
    active: {
      on: { TOGGLE: { to: "inactive" } },
    },
    inactive: {
      on: { TOGGLE: { to: "active" } },
    },
  },
  render: (state) => (
    <div>
      <p>state: {state}</p>
      <button onClick={() => emit({ type: "TOGGLE" })}>toggle</button>
      <button onClick={() => emit({ type: "ALERT" })}>alert</button>
    </div>
  ),
});
```

The `"ALERT"` action does the same thing regardless of what state the Toggle machine is in.

## Reference to DOM node

Sometimes, you will need to have a reference to a DOM node, either to read a value or to imperatively manipulate it.

### getElementById

If the machine is a singleton, you can get a reference to a DOM node by simply using `document.getElementById`:

```tsx
import { b, machine } from "baahu";

const Machine = machine({
  id: "machine",
  initial: "default",
  context: () => ({}),
  when: {
    default: {},
  },
  mount: (ctx) => {
    ctx.dom = document.getElementById("hello");
  },
  render: () => <p id="hello">Hello, world!</p>,
});
```

An important part to understand is the `mount` method. `mount` is called _after_ rendering, so even after the first render, `context.dom` holds a reference to the DOM node.

### Refs

For dynamic machines, use refs.

```tsx
import { b, machine } from "baahu";

const Machine = machine({
  id: "machine",
  initial: "default",
  context: () => ({}),
  when: {
    default: {},
  },
  render: (_state, ctx) => <p ref={(ref) => (ctx.ref = ref)}>Hello, world!</p>,
});
```

Provide a function that takes a reference to the DOM node as the `ref` prop. You can do whatever you want in this function, but you will usually just want to assign the ref to a property on the context object.

Note: The getElementById on `mount` approach can still work with parameterized ids, but refs usually easier in this case.

## Hydrate from localStorage

If you want to persist state between routes or browser sessions, you can use `localStorage`. With derived ids, state, and context, it feels natural to hydrate from `localStorage`.

You could persist and hydrate toggle state like this:

```tsx
import { b, machine, emit } from "baahu";

const PersistedToggle = machine({
  id: "toggle",
  inital: deriveInitialState,
  context: () => ({}),
  when: {
    inactive: {},
    active: {},
  },
  render: (state) => (
    <div>
      <p>state: {state}</p>
      <button onClick={() => emit({ type: "TOGGLE" })}>toggle</button>
    </div>
  ),
});

/** the return value of this function is the initial state. this function
 * is passed props, but we do not need props for this machine */
function deriveInitialState(props) {
  return localStorage.getItem("toggleState");
}
```

## TypeScript

Here are some ways you can get more help from TypeScript

### Events

Events are type-checked inside of machines, but not by `emit`.

#### Machines

Use a [discriminated union](https://www.typescriptlang.org/docs/handbook/advanced-types.html#discriminated-unions), where the type is the discriminant.

If you find yourself making lots of mistakes with event payloads and/or spelling event types in `emit`, you can wrap `emit` with your own typed version:

```tsx
import { emit } from "baahu";

type AppEvent = { type: "TOGGLE" } | { type: "SEARCH"; term: string };

function emitter(event: AppEvent, target?: string) {
  emit(event, target);
}
```

### Namespaced Emitters

## Make your first machine

Make your first baahu machine: Traffic Light Machine

## Prior Art

The "declarative config object" API was inspired by [XState FSM](https://xstate.js.org/docs/packages/xstate-fsm/).
