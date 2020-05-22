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

const MinimalMachine = machine({
  id: "minimal",
});
```

### `id`:

`id` is the identifier of a machine. When baahu is rendering and finds a machine component, it checks, by id, whether an instance of this component is in the "machine registry" (internal map of machine instances).

Fixed string ids work for "singleton" machines; machines that you know will only have one instance on the page.

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

## TypeScript
