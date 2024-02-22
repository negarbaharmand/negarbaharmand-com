---
author: Negar Baharmand
pubDatetime: 2023-04-19T22:00:00.000Z
title: What are environment variables in React and how to use them?
slug: environmental-variable-react
featured: true
draft: false
tags:
  - react
  - javascript
description:
  Exploring the use of special codes, known as environment variables, in React
  to secure sensitive data like API keys, based on my recent experience
  developing a 'Recipe Application.
---

Let me tell you about my recent adventure creating a 'Recipe Application' using React. I faced a bit of a puzzle when it came to keeping API keys safe, especially when trying to store them in a special file called '.env' in a Vite-built app. In this article, I want to share what I learned and help out others who might be in the same boat.

So, what are these things called environment variables? Well, they're like secret codes that help your app work smoothly in different situations, keeping important stuff like passwords and API keys hidden away. Instead of directly putting these secrets in your code, you use these special codes.

Now, let's get into how you can use these special codes in React. Start by creating a file named '.env' at the main folder of your project. If you're using Vite, add 'VITE\_' before any codes you want your app to use:

```
VITE_API_KEY = ae9518266f9b49
```

And you can access the variable in our React components using the 'process.env' object:

```javascript
const apiKey = import.meta.env.VITE_API_KEY;
```

For those using React App, just change 'VITE\_' to 'REACT_APP\_' inside your '.env':

```
REACT_APP_API_KEY = ae9518266f9b49
```

Then, grab the code in your app like this:

```javascript
const apiKey = process.env.REACT_APP_API_KEY;
```

Using these special codes is like a secret language that makes your app more secure and easy to work with. Oh, and don't forget to tell Git to ignore this special '.env' file so your secrets stay safe.

That's it! I hope this helps others on their coding journey.
