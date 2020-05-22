---
id: keys
title: Keys
---

## Why are keys important?

[View full image](/img/keys.svg)

![keys](/img/keys.svg)

## Rules of keys

1. Every key in a list of children must be keyed.

2. Each key must be unique amongst its siblings. This is intuitive, because the key is what marks a nodes _identity_

## How to use keys

```tsx
import { b, SFC } from "baahu";

const myTodoList = [
  { id: "uno", todo: "wake up" },
  { id: "dos", todo: "brush teeth" },
  { id: "tres", todo: "eat breakfast" },
];

const TodoList: SFC = () => (
  <div>
    <h3 key="header">TodoList</h3>
    {myTodoList.map((item) => (
      <p key={item.id}>{item.todo}</p>
    ))}
  </div>
);
```
