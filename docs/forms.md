---
id: forms
title: Forms
sidebar_label: Forms
---

There are two types of forms in Baahu: controlled and uncontrolled forms.

## Controlled Forms

In controlled forms, the "owner" of state is the machine. Let's make a simple form in which the user inputs their name:

```tsx
import { b, emit, machine } from "baahu";

const Form = machine({
  id: "form",
  initial: "pristine",
  context: () => ({ name: "" }),
  when: {
    pristine: {
      on: {
        INPUT: {
          do: (ctx, e) => (ctx.name = e.text),
          to: "touched",
        },
      },
    },
    touched: {
      on: {
        INPUT: {
          do: (ctx, e) => (ctx.name = e.text),
        },
        SUBMIT: {
          to: "submitted",
        },
      },
    },
    submitted: {},
  },
  render: (state, ctx) => (
    <form onSubmit={handleSubmit}>
      <p>{state}</p>
      <p>{ctx.name || "no name"}</p>
      <input
        placeholder="Enter your name"
        type="text"
        value={ctx.name}
        onInput={(e) => emit({ type: "INPUT", text: e.currentTarget.value })}
      />
      <button disabled={state === "submitted"} type="submit">
        Submit
      </button>
    </form>
  ),
});

function handleSubmit(e) {
  e.preventDefault();
  emit({ type: "SUBMIT" });
}
```

We call this a controlled form because on each browser input event, we emit a Baahu event with the new text value. The machine handles the event (updates its state/context), then rerenders (screen reflects new state/context).

This process was necessary because we wanted to use the value before it was submitted to render it above the form. Any machine, at any position in the virtual DOM tree, can stay in sync with the state of controlled forms.

## Uncontrolled Forms

When you don't need Baahu to rerender until the form has been submitted, you can use uncontrolled forms. You can get data out of the forms using the `name` attribute.

```tsx
const Form = machine({
  id: "form",
  initial: "default",
  context: () => ({ name: "" }),
  when: {
    default: {
      on: {
        SUBMIT: {
          to: "submitted",
          do: (ctx, e) => (ctx.name = e.value),
        },
      },
    },
    submitted: {},
  },
  render: (state, context) => (
    <form onSubmit={handleSubmit}>
      <Link to={{ path: "/about", state: { name: "Baahu" } }}>Go to about</Link>
      <input name="user" type="text" placeholder="Enter your name" />
      <button type="submit" disabled={state === "submitted"}>
        Submit
      </button>
      {state === "submitted" && <p>Your name is {context.name}</p>}
    </form>
  ),
});

function handleSubmit(e: JSX.TargetedEvent<HTMLFormElement, Event>) {
  e.preventDefault();
  const input = e.currentTarget.user;
  emit({ type: "SUBMIT", value: input.value });
  input.value = "";
}
```

The input has the `name` "user". On the submit event, emit a Baahu event to let interested machines know that the form has been submitted + any relevant values.

Credit to [swyx for popularizing this pattern in React](https://www.swyx.io/writing/no-controlled-forms/).
