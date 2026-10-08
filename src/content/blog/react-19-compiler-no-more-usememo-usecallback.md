---
author: Negar Baharmand
pubDatetime: 2025-11-22T16:37:00.000Z
title: "React 19 Compiler — No More useMemo and useCallback"
slug: react-19-compiler-no-more-usememo-usecallback
featured: true
draft: false
tags:
  - react
  - react19
  - performance
  - compiler
  - javascript
description: "React Compiler v1.0 landed and it quietly took over the job I was doing badly — deciding what to memoize and when."
---

I remember the first time I wrapped literally everything in `useMemo` and `useCallback` on a product listing page. I was convinced I was being smart about performance. The component had a filter function, a sort function, a
formatted price value, all wrapped up nice and tight. My mentor looked at
the PR and said, _"Do you actually know what these are memoizing?"_ I did not.
I just knew that re-renders were bad and memoization was the cure. Spoiler:
it wasn't. Half of those hooks were doing nothing useful, and the dependency
arrays were a maintenance nightmare waiting to happen. Ajaaaab...!

Fast forward to today, and React Compiler v1.0 just shipped on
**October 7, 2025**. And it basically says: _let us handle that for you._

## TABLE OF CONTENTS

## What is the React Compiler?

The React Compiler is a **build-time tool**. A Babel plugin that
analyzes your component code and automatically applies memoization where it
actually matters. You don't add new APIs. You don't change how you write
components. You just write plain, idiomatic React, and the compiler figures
out what needs to be cached and what doesn't.

Under the hood, it parses your code into an Abstract Syntax Tree (AST),
converts it into its own intermediate representation (HIR), and traces the
dependency chain of every value inside your components like objects, arrays,
functions, JSX. From there it classifis each expression as either **static**
(never changes between renders) or **dynamic** (might change), and injects
memoization only where it provides a real benefito.

## What Problems Does It Solve?

Before the compiler, React's default behavior was: when a parent re-renders,
all its children re-render too, unless you manually opted out with
`React.memo`, `useMemo`, or `useCallback`. That put the performance burden
entirely on the developer.

The compiler focuses on two specific pain points:

- **Skipping cascading re-renders** when `<Parent />` re-renders but its
  children haven't actually changed
- **Skipping expensive calculations** like filtering or sorting a large
  array on every render when the inputs haven't changed

Both of these used to require manual memoization. Now the compiler handles
them automatically.

## So... Can I Delete All My useMemo and useCallback?

Kind of, but with some nuance.

For **new code**, the React team recommends relying on the compiler and only
reaching for `useMemo`/`useCallback` when you need precise manual control, like when a memoized value is a `useEffect` dependency and you need
fine-grained control over when that effect fires.

For **existing code**, they recommend leaving your current memoization in
place or at least testing carefully before removing it, since removing hooks
can change the compiler's output.

And `React.memo`? You can safely remove it when using the compiler. It
automatically applies the equivalent to all components.

One thing the compiler does that manual memoization _can't_: it can memoize
**conditionally computed values** things after an early return, for example.
That's simply not possible with `useMemo` or `useCallback`.

## One Catch: Rules of React

The compiler isn't magic. It requires your code to follow the
[Rules of React](https://react.dev/reference/rules). Things like not mutating
state directly, keeping components pure, not calling hooks conditionally. If a
component breaks these rules, the compiler skips it rather than producing
broken output. It fails gracefully, which is a nice design choice.

You can also explicitly opt out of compilation for a specific component with a
`"use no memo"` directive at the top of the function body — an escape hatch
for when you need full manual control.

## What Does the Compiled Output Actually Look Like?

Here's a simplified example. You write this:

```jsx
function ProductList({ products, selectedCategory }) {
  const filtered = products.filter(p => p.category === selectedCategory);
  return <List items={filtered} />;
}
```

The compiler transforms it into something like:

```jsx
function ProductList({ products, selectedCategory }) {
  const $ = useMemoCache(2);
  let filtered;

  if ($[0] !== products || $[1] !== selectedCategory) {
    filtered = products.filter(p => p.category === selectedCategory);
    $[0] = products;
    $[1] = selectedCategory;
  } else {
    filtered = $[2];
  }

  return <List items={filtered} />;
}
```

## My Take

Honestly, this feels like the React team finally belakhare admitting what a lot of
developers already knew: manual memoization is error-prone, easy to overuse,
and hard to maintain. Dependency arrays are a constant source of bugs, and the
cognitive overhead of deciding when to memoize is real.
The compiler doesn't make React simpler to understand! You still need to know
why memoization matters. But it removes the part where you have to do it
perfectly by hand, every time, in every component.
For me, the most exciting part isn't even the performance gains it's that my
components can just be components again. No more defensive useCallback
wrapping every handler just in case it ends up as a prop somewhere.
