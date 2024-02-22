---
author: Negar Baharmand
pubDatetime: 2023-12-25T23:00:00.000Z
title: "TIL JavaScript Destructuring: Unpacking the Power of Objects and Arrays"
slug: js-destructuring
featured: true
draft: false
tags:
  - array
  - ilt
  - javascript
description:
  Unlock the potential of JavaScript destructuring, a concise and powerful
  technique simplifying the extraction of values from objects and arrays,
  improving code readability and reducing redundancy.
---

Today, let's delve into the concept of JavaScript destructuring, a powerful tool that simplifies the process of extracting values from objects and arrays.
Objects in JavaScript often contain multiple key-value pairs, and traditionally, extracting values required verbose syntax. Enter destructuring, a concise way to access and assign these values. Consider the following example:

```javascript
const dog = {
  fullName: "Fido",
  age: 4,
  bark() {
    console.log("woof");
  },
};

// Traditional approach
const name = dog.name;
const age = dog.age;

// Destructuring assignment
const { name: fullName, age } = dog;
```

In the second block of code, we use destructuring assignment to create variables **`fullName`** and **`age`** with values extracted from the dog object. This syntax enhances code readability and reduces redundancy.

## Array Deconstruction

Arrays, too, benefit from this powerful feature. Let's explore array destructuring:

```javascript
const arr = ["foo", "bar", "baz"];

// Traditional approach
const a = arr[0];
const b = arr[1];
const c = arr[2];
```
