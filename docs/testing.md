---
id: testing
title: Testing
sidebar_label: Testing
---

## Unit Testing

For unit testing your transition and effect functions, you can use a testing framework like [Jest](https://jestjs.io/en/). For particulary tricky or essential functions, consider using [fast-check](https://github.com/dubzzz/fast-check).

## UI Tests

For UI tests, use [DOM Testing Library](https://testing-library.com/docs/dom-testing-library/intro).

You can test individual components by passing them to `mount` and emitting events.

## Internal Tests

Baahu machine logic was written in TypeScript, and has been heavily tested with both 'example-based' unit tests and property-based tests with [fast-check](https://github.com/dubzzz/fast-check). As a result, if your code compiles, it is likely to work. When testing a baahu app, refrain from testing the framework itself; test the functionality of your app instead.

However, if you find a bug in the baahu source, or you believe that a bug in your app is caused by a bug in baahu, please [let us know](https://github.com/tjkandala/baahu/issues)! Also, feel free to add more tests to baahu; PRs are welcome!
