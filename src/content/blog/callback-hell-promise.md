---
author: Negar Baharmand
pubDatetime: 2023-02-06T23:00:00.000Z
title: "TIL Discovering Java's OOP Magic: Abstract and Interface Made Simple"
slug: callback-hell-promise-chaining
featured: true
draft: false
tags:
  - promise
  - javascript
  - til
description:
  Addressing the challenges of callback hell in JavaScript by exploring
  solutions like Promises, Async/Await, and modularization for enhanced code
  readability and maintainability.
---

In the dynamic landscape of asynchronous programming, the occurrence of callback hell, where callbacks are nested within functions, poses a common challenge. Initially executing tasks one after another, where each subsequent task depends on the result of the previous one, can lead to a pyramid-shaped set of nested callbacks, making code difficult to follow and maintain.

```javascript
getTodos("todos/luigi.json", (err, data) => {
  console.log(data);
  getTodos("todos/shaun.json", (err, data) => {
    console.log(data);
    getTodos("todos/mario.json", (err, data) => {
      console.log(data);
    });
  });
});
```

The aforementioned code structure, resembling a callback hell, raises concerns about code readability and maintainability. Recognizing these challenges, developers often seek alternative solutions.

## Exploring Solutions

### 1. Promises

One effective solution to callback hell is the adoption of Promises. Promise chaining offers a cleaner and more readable code structure, passing results through a chain of `.then` handlers.

```javascript
getTodos("todos/luigi.json")
  .then(data => {
    console.log("promise resolved", data);
    return getTodos("todos/mario.json");
  })
  .then(data => {
    console.log("promise2 resolved", data);
    return getTodos("todos/shaun.json");
  })
  .then(data => {
    console.log("promise3 resolved", data);
  })
  .catch(err => {
    console.log("promise rejected", err);
  });
```

### 2. Async/Await

Another approach is using Async/Await, which simplifies asynchronous code by allowing developers to write it in a synchronous style.

```javaScript
async function fetchTodos() {
  try {
    const luigiData = await getTodos("todos/luigi.json");
    console.log("Async/Await resolved:", luigiData);

    const marioData = await getTodos("todos/mario.json");
    console.log("Async/Await resolved:", marioData);

    const shaunData = await getTodos("todos/shaun.json");
    console.log("Async/Await resolved:", shaunData);
  } catch (err) {
    console.log("Async/Await rejected:", err);
  }
}

fetchTodos();
```

### 3. Modularization

Breaking down complex tasks into smaller, modular functions can also alleviate callback hell. This promotes code reusability and enhances readability.

```javaScript
function fetchAndLogTodos(filename) {
  return getTodos(filename)
    .then((data) => {
      console.log("Task resolved:", data);
    })
    .catch((err) => {
      console.log("Task rejected:", err);
    });
}

Promise.all([
  fetchAndLogTodos("todos/luigi.json"),
  fetchAndLogTodos("todos/mario.json"),
  fetchAndLogTodos("todos/shaun.json"),
])
  .then(() => {
    console.log("All tasks completed successfully!");
  })
  .catch((err) => {
    console.log("At least one task failed:", err);
  });
```

By exploring and implementing various solutions, including Promises, Async/Await, and modularization, developers can enhance the readability and maintainability of asynchronous JavaScript code, mitigating the challenges posed by callback hell.
