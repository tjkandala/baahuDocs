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
    inactive: { on: { TOGGLE: { to: "active" } } },
    active: { on: { TOGGLE: { to: "inactive" } } },
  },
  render: (state) => (
    <div>
      <p>Current state: {state}</p>
      <button onClick={() => emit({ type: "TOGGLE" }, "toggle")}>toggle</button>
    </div>
  ),
});
```

## High-level overview

Baahu machine components are loosely based on More machines; ["a finite-state machine whose output values are determined only by its current state"](https://en.wikipedia.org/wiki/Moore_machine). The render function is roughly equivalent to the output function of a Moore machine.

The [message-passing semantics](https://en.wikipedia.org/wiki/Message_passing) are inspired by the actor model. The second argument of `emit` is the "mailing address" (`id` of the machine instance) of the machine you want to send the message to. If you omit the target, the event becomes a global event.

## Properties of a Baahu machine

- **id**: A unique identifier (string) of a machine, _or_ a function that returns a unique identifier.

- **initial**: The initial state (string), or a function that returns the initial state.

- **context**: This is a function that returns an object. This extended state is stored in the 'machine instance',

- **when**: Represents a finite set of states. It is a 'declarative config object' that describes how the machine should behave.

* **render\***: The output function, returns a VNode.

* **mount** (optional):

* **unmount** (optional):

## Props & id

In Baahu, props provided to a machine have two uses:

1. Providing a stable identity

2. Initializing state and/or context

When Baahu reaches a component in the virtual DOM tree, it will resolve its id. The id property is either the id, or a function that returns the id based on props.

Baahu will then check the machine registry (map of active machine instances) to see if an instance with its id already exists. If not, it will create an instance by:

1. Resolving the initial state. If it is a string, that is the initial state. If it is a function, calls the function with props.

2. Resolving the initial context. This will always be a function that returns an object; calls the function with props.

It is imperative that each id belongs to only one machine instance at a time.

### Singleton Machines

A considerable amount of components in web applications should only appear once / have one instance. For these components, we can use a fixed string for the `id` property:

```tsx
import { b, machine } from "baahu";

const TodoList = machine({
  id: "todo-list",
  // rest of component
});
```

### Dynamic Machines

This is how you can create a dynamic machine that derives an id from props:

```tsx
import { b, machine } from "baahu";

const TodoItem = machine({
  id: (props) => `item-${props.id}`,
  // rest of component
});
```

## Context

The context of a machine instance is any data that is _not_ a finite state that the machine can be in.

Context is initialized with the `context` property:

```tsx
import { b, machine } from "baahu";

const TodoItem = machine({
  context: (props) => {
    return {
      label: props.label,
    };
  },
  // rest of component
});
```

The `context` property is a function that is called with props, and must return an object. The context object is updated by **mutation** as a response to events.

## When

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

## Lifecycle Hooks

Occasionally, you will need do perform actions based not on events, but on the lifecycle of the component. For example, subscribing to a data source on `mount` and unsubscribing on `unmount`.

### mount

`mount` is an **optional** method that is called after the first time a machine (by id) has been rendered.

```tsx
import { b, machine } from "baahu";

const TodoList = machine({
  mount: () => console.log("i just mounted"),
  // rest of component
});
```

`mount` is passed the context of the machine instance

```tsx
import { b, machine } from "baahu";

const TodoList = machine({
  mount: (ctx) => console.log(ctx),
  // rest of component
});
```

### unmount

`unmount` is an **optional** method that is called after a machine has been removed from the virtual DOM tree.

```tsx
import { b, machine } from "baahu";

const TodoList = machine({
  unmount: () => console.log("i just unmounted"),
  // rest of component
});
```

`unmount` is passed the context _and_ state of the machine instance before it was unmounted.

```tsx
import { b, machine } from "baahu";

const TodoList = machine({
  unmount: (ctx, state) => console.log(ctx, props),
  // rest of component
});
```

## Events

To emit an event, use the `emit` function.

```tsx
import { emit } from "baahu";

emit({ type: "TOGGLE", name: "Baahu" }, "toggle");
```

The first argument of `emit` is the event object. The event object must have a `type` property (string) that identifies the event. You can send arbitrary payloads in this object. The second (optional) argument is the id of the machine to target.

### Emit Events

### Targeted

[View full image](/img/targeted-event.svg)

![targeted events](/img/targeted-event.svg)

### Global

[View full image](/img/global-event.svg)

![global events](/img/global-event.svg)

When a global event is emitted, Baahu iterates through the machine registry (map of active machine instances).

If the machine listens to the emitted event type, Baahu will process its transition (`to` state) and effects (`entry`, `exit`, `do`).

(Note: If an effect synchronously causes another event, Baahu will process the resulting transitions + actions before rendering. This ensures that in any given [task](https://developer.mozilla.org/en-US/docs/Web/API/HTML_DOM_API/Microtask_guide#Tasks_vs_microtasks), Baahu only renders virtual nodes once.)

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

## Render

The final property of machine components is the `render` method. `render` must return either a `VNode` or `null`.

```tsx
import { b, machine } from "baahu";

const Machine = machine({
  // rest of machine
  render: () => <p>Hello, world!</p>,
});
```

Baahu passes 4 arguments to `render`:

1. The current state of the machine instance

2. The context object of the machine instance

3. The id of the machine instance (for convenience when emitting)

4. Children

```tsx
import { b, machine, emit } from "baahu";

const Machine = machine({
  // rest of machine
  render: (state, ctx, id, children) => (
    <div>
      <h3>Current state is: {state}</h3>
      <button onClick={() => emit({ type: "TOGGLE" }, id)}>
        Send event to self
      </button>
      {children}
    </div>
  ),
});
```

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

### Machines

Machines take 4 type arguments:

1. **Props**: Essentially an object

2. **State**: String union of possible states

3. **Events**: [discriminated union](https://www.typescriptlang.org/docs/handbook/advanced-types.html#discriminated-unions), where the type is the discriminant. You may prefer to split up event types by domain, so that for very large applications, you are not suggested every possible event by Intellisense.

4. **Context**: Essentially an object

```tsx
import { b, machine } from "baahu";

type ToggleState = "inactive" | "active";

type ToggleEvent = { type: "TOGGLE" };

const Toggle = machine<{}, ToggleState, ToggleEvent, {}>({
  id: "toggle",
  state: "inactive",
  context: () => ({}),
  when: {
    inactive: {},
    active: {},
  },
  render: (state) => <p>{state}</p>,
});
```

### Emit

Events are type-checked inside of machines, but not by `emit`.

If you find yourself making lots of mistakes with event payloads and/or spelling event types in `emit`, you can wrap `emit` with your own typed version:

```tsx
import { emit } from "baahu";

type AppEvent = { type: "TOGGLE" } | { type: "SEARCH"; term: string };

function emitter(event: AppEvent, target?: string) {
  emit(event, target);
}
```

## Prior Art

The "declarative config object" API was inspired by [XState FSM](https://xstate.js.org/docs/packages/xstate-fsm/).
