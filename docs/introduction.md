---
id: introduction
title: Introducing Baahu
sidebar_label: Introduction/Pitch
---

## What is Baahu?

Baahu is a zero-dependency state-machine-based SPA framework for Javascript + TypeScript

## Features

- [Faster and smaller than major frameworks/libraries](performance.md) (Svelte, Preact, Vue, React, and Angular) [0]
- Built-in robust state management: Finite State Machines!
- Event-driven, not change-driven/reactive
- Built-in trie-based router & code-splitting
- First-class TypeScript support: type-checked JSX, props, states, events.
- O(1) component rendering for _all_ components, not just leaves.

\[0] Note that this doesn't mean that baahu is better than any of these frameworks. Their creators and communities have poured countless man-hours into improving developer experience. These are just tech-specs.

## Contributors

- [TJ Kandala](https://github.com/tjkandala)

## Tradeoffs/Anti-Pitch

baahu is able to maintain its size and performance while including batteries because it is not a flexible view library or reactive framework. The built-in features, such as routing, state management, and dynamically imported components, are deeply coupled to baahu's internal architecture. A framework like react lends itself to boundless composition. Userland routing and global state management libraries can be implemented a little more elegantly in React than in baahu.

- API is slightly more restrictive compared to larger libraries. For instance, components cannot return arrays (single child only)

## Why was baahu created?

The purpose of baahu is to make complex UI easier to implement through explicit state machines, without sacrificing end user experience (bundle-size + performance).

[Read more](/blog/2020-05-17-why-baahu.md)
