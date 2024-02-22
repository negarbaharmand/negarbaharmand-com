---
author: Negar Baharmand
pubDatetime: 2023-01-22T23:00:00.000Z
title: TIL "this" keyword in JavaScript
slug: this-keyword
featured: true
draft: false
tags:
  - exception
  - java
description:
  Navigating the nuances of "this" in JavaScript objects, from its behaviour
  within object methods to potential pitfalls with arrow functions.
---

In JavaScript, the `this` keyword is used to refer to the current object. It can be used within an object's method to refer to the object itself, or it can be used alone to refer to the global window object.
It is also used to refer to the object that a function is a method of. It provides a way to access the object from within the function.
In the code below `this` refers to the object `user`:

```javascript
let user = {
  name: "Negar",
  age: 32,
  email: "ngdezfouli@gmail.com",
  log: function () {
    console.log(this);
  },
};
user.log();
```

Upon invoking user.log(), the console reveals an insightful snapshot of the user object:

```
{
  name: 'Negar',
  age: 32,
  email: 'ngdezfouli@gmail.com',
  log: ƒ
}
age: 32
email: "ngdezfouli@gmail.com"
log: ƒ ()
name: "Negar"
[[Prototype]]: Object

```

Now, let's explore what happens when `this` is used outside the object scope:

```javascript
let user = {
  name: "Negar",
  age: 32,
  email: "ngdezfouli@gmail.com",
};
console.log(this);
```

Executing this code snippet yields a different result, pointing to the global window object:

```
Window {window: Window, self: Window, document: document, name: '', location: Location, …}
```

## Arrow Functions and Global Context

A crucial aspect to consider is the behavior of `this` within arrow functions when used inside methods:

```javascript
let user = {
  name: "Negar",
  age: 32,
  email: "ngdezfouli@gmail.com",
  log: () => {
    console.log(this);
  },
};
```

In this case, `this` within the arrow function references the global window object, not the intended user object. To maintain the correct context, it's essential to use a regular function instead of an arrow function.
