---
id: introduction
title: Introducing Baahu
sidebar_label: Introduction/Pitch
---

## What is Baahu?

Baahu is a zero-dependency state-machine-based SPA framework for Javascript + TypeScript

## Features

- [Faster and smaller than major frameworks/libraries](performance.md) (Svelte, Preact, Vue, React, and Angular)
- Built-in robust state management: Finite State Machines!
- Event-driven, not change-driven/reactive
- Built-in trie-based router & code-splitting
- First-class TypeScript support: type-checked JSX, props, states, events.
- [O(1) component rendering](performance#higher-level-internal-optimizations) for _all_ components, not just leaves.

## Contributors

- [TJ Kandala](https://github.com/tjkandala)

## Tradeoffs/Anti-Pitch

- API is slightly more restrictive compared to larger libraries. For instance, components cannot return arrays (single child only).

- The built-in features, such as routing, state management, and dynamically imported components, are coupled to baahu's internal architecture. They cannot be tree-shaken away.

## Why was baahu created?

The purpose of baahu is to make complex UI easier to implement through explicit state machines, without sacrificing end user experience (bundle-size + performance).

[Read more](/blog/2020-05-17-why-baahu.md)
