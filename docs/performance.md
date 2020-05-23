---
id: performance
title: Performance
author: TJ Kandala
---

A "low-cost" framework is better for both developers (more focus on logic, not micro-management of renders) and users (bad perf == bad UX). Baahu is not optimized for benchmarks, but it maintains good runtime performance and minimal size by design.

This page covers Baahu's benchmark performance, its higher and lower-level internal optimizations, and techniques (such as `memo`) you can use to speed up your Baahu apps.

## Benchmarks

These are Baahu's results in the reputable [js-framework-benchmark](https://github.com/krausest/js-framework-benchmark). While the application being tested does not resemble a "real world" application, the benchmark gives us a statistically rigorous look at the performance characteristics of DOM libraries. Theory is great, but we must test our ideas in order to get any "real world" value from them.

Be aware that these results are from my machine (2017 13" MBP, 3.1 GHz dual-core Intel Core i5 Kaby Lake), not from the benchmark creator himself. Regardless, all of the frameworks that I tested had the same (relative) performance in this run and the official results.

### main runtime performance benchmark (keyed)

[view full image](/img/keyed.png)

![main runtime performance benchmark (keyed)](/img/keyed.png)

### startup performance (lighthouse)

![startup performance](/img/startup.png)

Baahu is near best-in-class here. It has essentially the same TTI as vanillajs and Svelte, and has the same script bootup time as all of the top performers.

The category in which Baahu is bested is kilobyte weight ("bundle size"). Both vanillajs and Svelte are slightly smaller than baahu for this application. However, this measurement includes baahu's router, dynamic import component, and state management logic, none of which are used in this application.

Also, keep in mind that a virtual DOM runtime is a fixed cost; a Baahu component will grow at a slower rate than a [Svelte component](https://github.com/sveltejs/svelte/issues/2546)

### memory allocation

![memory allocation](/img/memory.png)

Virtual DOM libraries are at a disadvantage here. See how Choo, which directly traverses the DOM, fares much better in this metric. While the virtual DOM model will inevitably lead to more memory usage, Baahu still does OK.

Interestingly, virtual DOM libraries have lower memory overhead for components. In this benchmark, Svelte only uses [one component](https://github.com/krausest/js-framework-benchmark/blob/master/frameworks/keyed/svelte/src/Main.svelte). Read this [article](https://medium.com/better-programming/the-real-cost-of-ui-components-6d2da4aba205) for more information on the cost of the component abstraction.

## Higher-Level Internal Optimizations

The main "optimization" in Baahu is that only components that SHOULD re-render, re-render. At first, this sounds like this should be a property of every framework. However, this isn't possible without (potentially expensive) prop equality checks. In traditional virtual DOM frameworks, state updates are propagated by passing props down to children. Because of this, shared state between distant leaves will result in many unnecessary rerenders of intermediate components. "Global state management" libraries help to overcome this, but only by working _around_ the framework, not with it.

**This optimization is not useful in the benchmark reviewed above**, which has less complex state than TodoMVC. Regardless, it can really help [0] when sending messages between machines (i.e, global state). Here is a visual explanation of the internals:

### Global event visualization

[View full image](/img/rendering-opts.svg)

![rendering opts](/img/rendering-opts.svg)

[0] I'm a hypocrite. I don't have any solid numbers for "global events in Baahu vs. other frameworks," mostly because I haven't thought of any good apples-to-apples comparisons. Also, there is the major confounding variable of performance of the DOM manipulation layer (Baahu vs. react-redux would test react more than redux). In this case, we can rely on our intuition that "doing less work is good"; a component will only rerender if it reacted to an event!

## Lower-Level Internal Optimizations

baahu is not micro-optimized for benchmarks, but it tries to take advantage of monomorphic inline caching. [Read more](https://en.wikipedia.org/wiki/Inline_caching#Monomorphic_inline_caching)

## Developer Optimizations: What should YOU do?

### Machine Components

You don't need to do anything to optimize baahu machines.

### `memo`

However, if you are rendering large lists of SFCs (Stateless Functional Components), you can memoize the SFC with `memo`.

```tsx
import { b, memo, SFC } from "baahu";

type ItemProps = { name: string; price: number };

// This SFC will always rerender its VNode tree; even if the props didn't change!
const Item: SFC<ItemProps> = ({ name, price }) => {
  return (
    <div>
      <h3>Item: {name}</h3>
      <p>Price: {price}</p>
    </div>
  );
};

// Now, the memoized component will only create a new VNode tree when a prop has changed
const MemoItem = memo(Item)

// Use it like you would use any other component
<MemoItem name="chair" price={123} />
```

You should be aware that `memo` performs a shallow comparison of props. This is the only part of baahu that holds any opinion on mutability vs. immutability. You can mutate anything else to your hearts content (w/ performance benefits!) becuase machines rerender based on _events_, not referential inequality.

### Don't _always_ use `memo`

If you know that the props passed to an SFC will be different on every render, `memo` only makes your app slower! This is because baahu has to compare the props _and_ rerender + diff. `memo` is beneficial in many cases, but this is the reason that functional component memoization is opt-in in most frameworks.
