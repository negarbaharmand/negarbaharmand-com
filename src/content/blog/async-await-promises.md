---
author: Negar Baharmand
pubDatetime: 2023-02-09T23:00:00.000Z
title: Writing more readable promises using async and await
slug: async-await
featured: true
draft: false
tags:
  - promise
  - javascript
description:
  Talking about the power of async and await in JavaScript for improved promise
  readability, encapsulating asynchronous code and streamlining chaining for
  enhanced clarity.
---

Today I want to talk about the power of `async` and `await` in JavaScript, where these keywords allow us to encapsulate asynchronous code within a single function, facilitating a more readable and logical chaining of promises.

This is how Async and Await works:

- When using `async`, the function returns a promise.
- Instead of employing the traditional `.then` method, we utilize `await` within the function to seamlessly chain promises.
- The `await` keyword prevents JavaScript from assigning a value until the promise has resolved, providing a clean way to handle asynchronous operations.

## Utilizing Async and Await

With `async` and `await`, we eliminate the need for numerous `.then` calls inside our asynchronous function, streamlining the promise chaining process. However, an initial `.then` is still required when calling an asynchronous function to avoid blocking.

Here is an example code, refactored from the previous post, showcasing the use of `async` and `await`:

```javascript
const getTodos = async (resource) => {
  const res = await fetch(resource);
  const data = await res.json();
  return data;
};

getTodos("todos/luigi.json").then((data) => {
  console.log("Success", data);
});

getTodos("todos/shaun.json").then((data) => {
  console.log("Success", data);
});

getTodos("todos/mario.json").then((data) => {
  console.log("Success", data);
}
```

Alternatively, in a more compact form:

```javaScript
const getTodos = async () => {
  const data1 = await fetch("todos/luigi.json").then((res) => res.json());
  console.log(data1);

  const data2 = await fetch("todos/shaun.json").then((res) => res.json());
  console.log(data2);

  const data3 = await fetch("todos/mario.json").then((res) => res.json());
  console.log(data3);
};

getTodos();
```

## Achievements of Async and Await

1. Encapsulation: Bundling asynchronous code within a function for flexible usage.
2. Non-Blocking: Asynchronous functions do not block the rest of the application code.
3. Readability: Provides a cleaner and more readable way to chain promises, enhancing code clarity.

Explore the simplicity and power of async and await for a more elegant approach to managing asynchronous operations.
