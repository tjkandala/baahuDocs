---
id: setup
title: Setup
---

## Quick Start

[Code Sandbox template](https://codesandbox.io/s/hello-baahu-2zzv7?file=/src/index.tsx)

## Bundling

Most baahu apps will have a build step. Bundling gives you huge performance, and therefore UX, returns for how little work you have to put in. You _can_ use baahu with just a script tag, but you'd be missing out on JSX, typescript, and easy code-splitting.

Parcel is used in this guide because it is quicker to set up, but any bundler will do the job.

## Language

TypeScript is the recommended language to get the 'full baahu experience.' For example, TypeScript will not compile your code if you forget to handle all the states that a machine could be in. You can also get full autocomplete for both machine logic and JSX.

However, using JavaScript has advantages as well. While you sacrifice help from the TypeScript compiler, machines are much faster to write in JavaScript. Either way, you'll need a build step to use JSX.

## Installation

### Installing Parcel

First, follow this short guide on [getting started with parcel](https://parceljs.org/getting_started.html).

### Installing baahu

In the root directory of your Parcel project, run:

`npm install baahu`

### Setting up JSX for TypeScript

If you don't already have typescript, run:

`npm install -g typescript`

In the root directory of your baahu project, run:

`tsc --init`.

Add these lines to your `tsconfig.json` file:

```json
{
  // ...other config
  "module": "ESNext",
  "moduleResolution": "node",
  "jsx": "react",
  "jsxFactory": "b"
}
```

The first two lines will help with code-splitting/dynamic imports, and the second two lines tell the TypeScript compiler to compile `<p>hi<p>` to `b("p", null, "hi")` instead of `React.createElement("p", null, "hi)`

Make sure to use the `.tsx` file extension for any file with JSX.

Add:

```json
 "browserslist": [
    "since 2017-06"
  ],
```

to your `package.json` to use async/await. If you need to support outdated browsers, use a babel polyfill.

### Setting up JSX for JavaScript

If you are using JavaScript, add this to your `package.json` file:

```json
{
  // ...other config
  "babel": {
    "plugins": [
      [
        "@babel/plugin-transform-react-jsx",
        {
          "pragma": "b"
        }
      ]
    ]
  }
}
```

Then, run:

`npm install --save-dev @babel/core @babel/plugin-transform-react-jsx`

## Mount

Use `mount` to initialize your application.

```tsx
import { mount, b } from "baahu";

const App = () => <p>App</p>;

mount(App, document.body);
```

The first argument of `mount` is the root component of your app. The second argument is the DOM node you want to append your app to.
