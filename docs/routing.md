---
id: routing
title: Routing & Code-Splitting
sidebar_label: Routing & Code-Splitting
---

## Getting Started

A router in baahu is just a component. This makes it easy to create decoupled nested routers, both statically and dynamically (w/ code splitting).

### Route Schema

```ts
import { router } from "baahu";

const MyRouter = router({}, "/");
```

The first argument of `router` is your route schema. The second argument is the root route. The root route is the implicit prefix of every route in your schema. This argument is optional and defaults to `"/"`.

The route schema is a plain object in which each key is a route (must start with `/`, as in `/about`, unless it is a global wildcard `*`), and each value is a function that returns a VNode.

The function takes three arguments:

1. Route Params (object with param name as key, string value as value)

2. Props (props object passed to the router is passed to the callback)

3. Children (children passed to the router are passed to the callback)

You can import the type `RouterCallback` to type-check your routes if you are defining the function outside of your route schema.

Let's make a basic router:

```tsx
import { router } from "baahu";
import About from "./about";
import Profile from "./Profile";

// this is a "root router" because it uses the default prefix
const MyRouter = router({
  "/": () => <p>you are home</p>,
  "/about": () => <About />, // arbitrary components
  "/user/:name": (params) => <Profile name={params.name} />, // using route params
  "*": (wildcard) => <p>404, {params.wildcard} id not a page</p>, // using wildcard
});
```

### Route Params

Observe the syntax used for the `/user/:name` route. Prefix a URL segment with `:` to make it a named parameter. The value at that position will be passed to the router callback. In this example, you can access the name value at `params.name`.

### Wildcard

If you want to display a fallback when no route is matched, use the `*` route. The route value will be passed to the router callback as `params.wildcard`.

### Links

#### Programmatic

Import the `linkTo` function. The argument of linkTo is just the route you want to link to.

```tsx
import { linkTo } from "baahu";

function linkToAbout() {
  linkTo("/about");
}
```

#### Link Component

Import the `Link` component. There are two ways to use link:

1. No state

Pass the desired path as the `to` prop.

```tsx
import { Link } from "baahu";

const MyComponent = () => (
  <div>
    <h1>Component</h1>
    <Link to="/about">Go to about</Link>
  </div>
);
```

You can pass arbitrary children to Link!

2. State

Pass an object as the `to` prop.

```tsx
import { Link } from "baahu";

const MyComponent = () => (
  <div>
    <h1>Component</h1>
    <Link to={{ path: "/about", state: { name: "Baahu" } }}>Go to about</Link>
  </div>
);
```

`path` is the path you want to link to, `state` is the the state you want to [push to the history stack](https://developer.mozilla.org/en-US/docs/Web/API/History/state).

### Passing props to router

If say you want to pass a prop to a component that will be rendered by a router:

```tsx
const MySFC: SFC<{ name: string }> = ({ name }) => (
  <div>
    <h1>My fun app</h1>
    <MyRouter name={name} />
  </div>
);

// the type argument for a router is the same as for an SFC: props shape
const MyRouter = router<{ name: string }>({
  "/": () => <p>you are home</p>,
  "/profile": (params, props) => <p>hi {props.name}, this is your profile</p>,
});
```

### Passing children to router

If say you want to pass children to a component that will be rendered by a router:

```tsx
const MySFC: SFC<{ name: string }> = ({ name }) => (
  <div>
    <h1>My fun app</h1>
    <MyRouter>
      <p>child node</p>
    </MyRouter>
  </div>
);

const MyRouter = router<{ name: string }>({
  "/": () => <p>you are home</p>,
  "/profile": (params, props, children) => (
    <div>
      <h2>{props.name}'s profile</h2>
      {children}
    </div>
  ),
});
```

You will probably pass render props to routers more often than children, but it is good to have to option if needed.

## Nested Routers

Wildcards aren't only useful as a fallback. You can use them to create nested routers:

```tsx
const RootRouter = router({
  "/": () => <p>home</p>,
  // similar to 'exact' match in React Router
  "/games": () => (
    <div>
      <p>games page</p>
    </div>
  ),
  "/games/*": () => <GamesRouter />,
});

const GamesRouter = router(
  {
    "/tetris": () => <p>play tetris</p>,
    "/madden": () => <p>play madden</p>,
    "*": ({ wildcard }) => <p>you can't play {wildcard} here</p>,
  },
  "/games"
);
```

## Code-Splitting

Using `bLazy`, we can [dynamically import](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import#Dynamic_Imports) components. Because routes return components, you can easily code-split at the route level. In a large application, you should code-split at component and route level.

```ts
// file `./LazyComp`'s default export is a baahu component!
const MyLazyComp = bLazy(() => import("./LazyComp"));
```

bLazy takes up to 4 arguments:

1. lazyComponent: This is a function that returns a promise that resolves to an object. The .default property of that object must be a baahu component.

Type signature of lazyComponent:

```tsx
type LazyComponent = () => Promise<{
  default: MachineComponent<Props> | SFC<Props> | MemoComponent<Props>;
}>;
```

The dynamically loaded component should be a _default_ export.

2. fallback (optional): The VNode to display if the component doesn't load before the timeout. After the component loads, it will replace this.

3. timeout (optional): How long to wait before displaying the fallback. Default is 300ms

4. onError (optional): The VNode to display if there is an error importing the component.

When `/games` is _exactly_ matched, RootRouter will render the games page. When there are segments after `/games`, RootRouter will render the GamesRouter. Because these two routers don't know or care about each other, you can render the nested router from within a component as well! Furthermore, the root router does not have to be the root of your application. As you can see, baahu routers are just components.

### Dynamically import a routeR

Let's combine the powers of code-splitting and nested routers:

`LazyRouter.tsx`

```tsx
import { b, router } from "baahu";

const LazyRouter = router(
  {
    "/": () => <p>lazy nested home</p>,
    "/one": () => <p>lazy nested route one</p>,
    "/two": () => <p>lazy nested route two</p>,
  },
  "/lazy"
);

export default LazyRouter;
```

`RootRouter.tsx`

```tsx
const LazyRouter = bLazy(() => import("./LazyRouter"), <p>loading...</p>);

const RootRouter = router({
  "/": () => <p>you are home</p>,
  "/lazy": () => <LazyRouter />,
  "/lazy/*": () => <LazyRouter />,
});
```

#### Limitations

`bLazy` is fairly naive. It is suitable for importing singleton-like components, like "route" components, or "container" components (like a calendar component). If you use multiple instances of the same bLazy component, it will only render the first one.

As long as you don't use `bLazy` to import components like a reusable button (these can be encapsulated by the route/container component), you will be fine. As you can see from the "nested lazy router" pattern, `bLazy` is still powerful.

## Internals

baahu routers use zero regular expressions. Instead, they are powered by the RouTrie (Router + Trie) class.

Compared to a [radix tree](https://en.wikipedia.org/wiki/Radix_tree), RouTrie trades a little bit of memory in order to reduce map lookups.

You can read the implementation [here](https://github.com/tjkandala/baahu/blob/master/src/router/index.ts)

### Prior Art

There are a few JS SPA routers similar to baahus RouTrie, but RouTrie outperforms them because its implementation is more optimizable by JS engines.

The trie-based internals are inspired by the elegant [nanorouter in choojs](https://github.com/choojs/nanorouter).
