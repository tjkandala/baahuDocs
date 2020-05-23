---
id: cheatsheet
title: Baahu Cheat Sheet
sidebar_label: Cheat Sheet
---

API Reference/Cheatsheet

## Creating Elements

Without JSX:

`b(TagName | Component, Props, Children)`

With JSX:

`<div class="container"><p>child<p></div>`

## SFC (Stateless Functional Components)

Just create a function that returns a VNode

```tsx
import { VNode } from "baahu";

function MySFC(): VNode {
  return <p>hi</p>;
}
```

Baahu also exports a type definition for arrow functions

```tsx
import { SFC } from "baahu";

const MySFC: SFC = () => <p>hi</p>;
```

## Memo (memoized SFC)

Wrap an SFC in `memo`

```tsx
import { memo, SFC } from "baahu";

const MySFC: SFC = () => <p>my sfc</p>;

const MyMemoSFC = memo(MySfc);

/** you can define the SFC inside of memo */

const FunMemo = memo(() => <p>fun</p>);
```

## Lazy (code-splitting)

`lazy` takes up to 4 arguments:

1. Function that returns a dynamically imported component

2) (Optional) fallback VNode (element)

3) (Optional) timeout in ms. Delay before displaying fallback. Default is 300ms.

4) (Optional) error VNode (element) in case of network problems.

```tsx
import { lazy } from "baahu";

const LazyComponent = lazy(
  () => import("./Component"),
  <p>loading</p>,
  500,
  <p>error</p>
);
```

## Router

### Root

```tsx
import { router } from "baahu";

const MyRouter = router({
  "/": () => <p>home</p>,
  "/about": () => <p>about</p>,
  "/param/:id": (params) => <p>id: {params.id}</p>,
  "*": () => <p>fallback</p>,
});
```

### Nested

```tsx
import { router } from "baahu";

const MyRouter = router(
  {
    "/": () => <p>path is /nested</p>,
    "/route": () => <p>path is /nested/route"</p>,
  },
  "/nested"
);
```

### Links

```tsx
import { Link, linkTo } from "baahu";

/** Component */
const AboutLink = <Link to="/about">Go to about</Link>;

/** Programmatic */
linkTo("/about");
```

## Machine Components

Use `machine` to create a Machine Component.

```tsx
import { machine } from "baahu";

const Toggle = machine({
  id: "toggle",
  initial: "inactive",
  context: () => ({}),
  when: {
    inactive: {
      on: { TOGGLE: { to: "active" } },
    },
    active: { on: { TOGGLE: { to: "inactive" } } },
  },
  render: (state) => <p>{state}</p>,
});
```

### `id`:

`id` is the identifier of a machine. When baahu is rendering and finds a machine component, it checks, by id, whether an instance of this component is in the "machine registry" (internal map of machine instances).

Fixed string ids work for "singleton" machines; machines that you know will only have one instance on the page.

```tsx
import { machine } from "baahu";

const MinimalMachine = machine({
  id: "minimal",
  //  rest of component
});
```

For dynamic machines, derive the id from props:

```tsx
import { machine } from "baahu";

const MinimalMachine = machine({
  id: (props) => `minimal-${props.id}`,
  //  rest of component
});
```

## Events

Use `emit` to emit events.

The first argument of `emit` is the event object. The event object must have a `type` property (string) that identifies the event. You can send arbitrary payloads in this object. The second (optional) argument is the id of the machine to target.

### Targeted

```tsx
import { emit } from "baahu";

emit({ type: "TOGGLE", name: "Baahu" }, "toggle");
```

Baahu will only check to see if the machine with the id `"toggle"` listens to this event.

### Global

If you omit the second argument, the event becomes a global event.

```tsx
import { emit } from "baahu";

emit({ type: "TOGGLE", name: "Baahu" });
```

Baahu will check every machine in the registry to see if it listens to this event. Do not fear global events; Baahu only reenders the components that listened to the event.

### State

#### Initial State

```tsx
import { machine } from "baahu";

const MinimalMachine = machine({
  initial: "inactive",
  //  rest of component
});
```

Specify the intial state with the `initial` property.

```tsx
import { machine } from "baahu";

const MinimalMachine = machine({
  initial: (props) => props.initial,
  //  rest of component
});
```

You can also derive initial state from props/within a function.

#### Initial Context

```tsx
import { machine } from "baahu";

const MinimalMachine = machine({
  context: (props) => ({}),
  //  rest of component
});
```

The initial context the return value of the `context` method.

### When

The `when` property of a machine component defines which events the machine listens to depending on which state it is in, and how the machine behaves on those events.

The following machine is intentionally verbose in order to demonstrate all of the optional properties of a baahu machine.

```tsx
import { machine } from "baahu";

const EvenOdd = machine({
  when: {
    /** name of state */
    even: {
      /** entry/exit actions */
      entry: () => console.log("entered even state"),
      exit: (ctx, e) => console.log("exited even state"),
      on: {
        /** event types */
        BECOME_ODD: {
          /** target state */
          to: "odd",
          /** 'do' actions */
          do: (ctx, e) => console.log("do function"),
          /** 'if' condition */
          if: (ctx, e) => true,
        },
      },
    },
    odd: {
      /** entry/exit actions */
      entry: () => console.log("entered odd state"),
      exit: () => console.log("exited odd state"),
      on: {
        /** event types */
        BECOME_EVEN: {
          /** target state */
          to: (ctx, e) => "even",
          /** 'do' actions */
          do: [
            (ctx, e) => console.log("do array"),
            (ctx, e) => console.log("do array"),
          ],
          /** 'if' condition */
          if: (ctx, e) => true,
        },
      },
    },
  },
  //  rest of component
});
```

The properties of the `when` object are the names of the states that the machine can be in.

The state can have up to three (**optional**) properties:

- **entry**: Function called with context + event when the machine **enters** this state.

- **exit**: Function called with context + event when the machine **exits** this state.

- **on**: On object that defines event listeners. Its keys are the event`type`s that the machine listens to, and each key's value is the event listener specification.

An event listener specification can have up to three (**optional**) properties

- **to**: The target state. This can be a string, or a function that returns a string (called with context + state)

- **do**: Perform side-effects, such as mutating context. This can be a function, or an array of functions. `do` functions are called with context + event.

- **if**: Function that determines whether to execute the transition and/or `do` actions. Called with context + event, returns boolean. If the event listener does not have an `if` function, it will always execute the transition and/or `do` actions

When an instance of this machine component is in the `even` state, it listens to the `BECOME_ODD` event. On that event, Baahu will check if the event listener has an `if` property. It does, so Baahu will call the function with the machine instance's context + event. The `if` function returns true, so it execute its `do` action and transition to the `odd` state. As a result of the transition, baahu will execute `even`s `exit` action, and `odd`s `entry` action.

Remember, machines don't _have_ to transition to other states on actions; they can simply perform side effects with `do` function(s)!

### Mutating Context

Example:

```tsx
import { b, machine, emit } from "baahu";

const Counter = machine({
  id: "counter",
  initial: "default",
  context: () => ({
    count: 0,
  }),
  when: {
    default: {
      on: {
        INCREMENT: {
          do: (ctx, e) => ctx.count++,
        },
      },
    },
  },
  render: (_state, ctx) => <p>Count {ctx.count}</p>,
});
```
