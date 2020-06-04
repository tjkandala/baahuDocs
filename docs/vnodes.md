---
id: vnodes
title: VNodes and SFCs
---

## VNodes

Virtual DOM nodes, or VNodes for short, are JavaScript objects that represent [DOM](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model) nodes. UIs in baahu are represented by trees of VNodes. When a machine (component) reacts to an event, it renders a new tree of VNodes. This new tree is compared to the old tree, and any differences between the trees are applied to the DOM.

### Creating VNodes

If you're familiar with React, it may be helpful to consider `baahu.b` as analogous to `React.createElement`.

```ts
import { b } from "baahu";

const vnode = b("p", { class: "hello" }, "hello world");
```

This is a simplified type signature of b:

```ts
function b(type: TagName | SFC | Machine, props: Props, children: ...VNode | string): VNode
```

The first argument is the type of VNode you want to create. TagName refers to the name of a DOM element. For example, "div" would create a VNode corresponding to a div element. You can also provide an SFC (Stateless Functional Component) or a Machine Component.

The second argument is the props object. For element nodes, the key-value pairs of the props object become DOM attributes. Later, we will see what the props object means for functional and machine components.

The recommended way to create VNodes in baahu is to use JSX. In baahu, JSX will provide you with more type-safety. See [Setup](setup.md) to learn how to use JSX with baahu.

```tsx
const vnode = <p class="hello">hello world</p>;
```

## Stateless Functional Components

These are the first type of "component" in baahu. Aptly named, stateless functional components (SFC) do not keep track of any internal state. They are most useful for encapsulating rendering logic.

### Props

The first argument passed to an SFC is the props object.

```tsx
<AnSFC name="baahu" />;
// becomes
b(AnSFC, { name: "baahu" });
```

Let's make an SFC:

```tsx
interface MyProps {
  name: string;
}

const MySFC: SFC<MyProps> = (props) => {
  return (
    <div>
      <p>hello, {props.name}</p>
    </div>
  );
};

// Alternatively, you could type it like this
import { VNode } from "baahu";

function MySFC(props: MyProps): VNode {
  // ...same code as arrow function
}
```

The type SFC takes one type argument: the structure of the props object. This is important because TypeScript will not accept this code:

```tsx
<MySFC />
```

The compiler will tell us that the property "name" is missing.

```tsx
<MySFC name="baahu" />
```

This works!

### Children

The second argument passed to an SFC is its children. Remember that in HTML:

```html
<div>
  <p>Hello, baahu!</p>
</div>
```

Anything between the opening and closing tags of a node are _children_ of that node. It is just a tree, after all! The "P" element is a child of the "DIV" element, and the "#text" node with the value "Hello, baahu!" is a child of the "P" element.

In JSX, we can pass children to a node in the same way:

```tsx
<MySFC>
  <p>first child</p>
  <p>second child</p>
</MySFC>;

// becomes

b(MySFC, null, b("p", null, "first child"), b("p", null, "second child"));
```

Let's modify our SFC to _use_ the children.

```tsx
const MySFC: SFC<MyProps> = (props, children) => {
  return (
    <div>
      <p>hello, {props.name}</p>
      {children}
    </div>
  );
```

Our SFC now renders two more paragraph nodes!

An SFC does not know or care about _what_ its children are. If you want to type-check children, or you want more control over how children are rendered, consider passing child VNodes and SFCs as props to SFCs.

```tsx
interface MyProps {
  name: string;
  TopChild: VNode;
  BottomChild: VNode;
}

const MySFC: SFC<MyProps> = (props) => {
  return (
    <div>
      {props.TopChild}
      <p>hello, {props.name}</p>
      {props.BottomChild}
    </div>
  );
};

// using MySFC
<MySFC
  name="baahu"
  TopChild={<p>first child</p>}
  BottomChild={<p>second child</p>}
/>;
```

A more powerful pattern is the 'render prop'. Instead of passing in an eagerly-evaluated 'createElement' expression (resolves to a VNode), you can pass in a function that returns a VNode (essentially an SFC).

```tsx
interface MyProps {
  name: string;
  error: boolean;
  RenderError: () => VNode;
}

const MySFC: SFC<MyProps> = (props) => {
  if (props.error) {
    return <props.RenderError />;
  }

  return <p>hello, {props.name}!</p>;
};

// using the pattern
<MySFC
  name="baahu"
  error={true}
  RenderError={() => <p>an error has occured!</p>}
/>;
```

Now, the SFC can decide when to render the VNode. The SFC can also supply arguments to the function:

```tsx
interface MyProps {
  name: string;
  error: boolean;
  RenderError: (name: string) => VNode;
}

const MySFC: SFC<MyProps> = (props) => {
  if (props.error) {
    return <props.RenderError name={props.name} />;
  }

  return <p>hello, {props.name}!</p>;
};

// using the pattern
<MySFC
  name="baahu"
  error={true}
  RenderError={(name: string) => <p>an error for {name}!</p>}
/>;
```

This becomes even more useful with machines, because they have their own state to pass to render props.

### Javascript Expressions

If you are not familiar with [JSX](https://reactjs.org/docs/introducing-jsx.html), you may be wondering what the `{}` mean. Inside curly braces, you can evaluate any [JavaScript expression](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators#Expressions).

In our examples so far, we have just used this feature to render strings from the function scope. There are endless rendering possibilities when your "templates" are turing-complete.

Adding numbers:

```tsx
interface MyProps {
  first: number;
  second: number;
}

// look up "destructuring assignment" if this line confuses you
const MySFC: SFC<MyProps> = ({ first, second }) => {
  return <p>the sum is {first + second}</p>;
};
```

Rendering a list:

```tsx
interface MyProps {
  // an array of strings
  names: string[];
}

// this expression returns an array of VNodes
const MySFC: SFC<MyProps> = ({ names }) => {
  return (
    <p>
      {names.map((name) => (
        <p>hello, {name}!</p>
      ))}
    </p>
  );
};
```

### Conditional Rendering

Sometimes, you want to render different trees based on conditions. If the trees are essentially completely different, you can acheieve conditional rendering this way:

```tsx
// should prefer machines (coming up soon) for loading state management
interface MyProps {
  loading: boolean;
  name: string;
}

const MySFC: SFC<MyProps> = ({ loading, name }) => {
  if (loading) {
    return <p>loading...</p>;
  }

  return (
    <div>
      <h1>hello, {name}!</h1>
      <p>loaded</p>
    </div>
  );
};
```

However, there are many scenarios in which what is essentially the same view must render/not render just a few nodes based on a condition. In this scenario, use conditional rendering in JSX!

This will also help baahu perform better by correctly comparing nodes by position (baahu creates placeholders for falsy values other than 0).

```tsx
interface MyProps {
  winner: boolean;
  name: string;
}

const MySFC: SFC<MyProps> = ({ winner, name }) => {
  return (
    <div>
      <h1>hello, {name}!</h1>
      {winner && <p>congrats! you won!</p>}
    </div>
  );
};
```

Don't shy away from conditional rendering in JSX! Ask yourself whether the view being rendered has the same "identity" regardless of the condition. If the answer is yes, JSX conditional rendering is the right tool, even if you have to use it e.g. 10 times in the same subtree.
