---
author: Negar Baharmand
pubDatetime: 2026-08-20T15:33:00.000Z
title: "Two Bugs, One API, and Why You Should Read Your Render Emails"
slug: two-bugs-one-api-read-your-render-emails
featured: true
draft: false
tags:
  - nodejs
  - mongodb
  - arcjet
  - debugging
  - backend
description:
  How ignoring Render failure emails led to a deactivated MongoDB Atlas cluster,
  and how Arcjet's bot detection broke behind a proxy — and what I did to fix both.
---

I've been building **sub-tracker** — a Node.js REST API for managing personal subscriptions. The idea is simple: you add your subscriptions, and the app tracks renewal dates and sends you reminder emails before they hit. JWT auth, rate limiting, automated workflows via Upstash, emails via Nodemailer. Nothing groundbreaking, but a solid project to work with real-world tooling in a backend-only context.

At some point I stopped actively working on it. And then I started ignoring the failure emails from Render.

That was a mistake.

---

## What Is Arcjet and Why Did I Use It?

Before getting into the bugs, a quick intro to one of the tools in this project: [Arcjet](https://arcjet.com/).

Arcjet is a developer-first security layer for Node.js (and other runtimes). You add it as middleware and it handles things like rate limiting, bot detection, and shield protection basically a lightweight WAF that lives in your code rather than in your infrastructure. The config is just JavaScript, which I appreciated. No dashboards to wrestle with, no separate service to maintain.

For sub-tracker, I used it to rate-limit the auth endpoints and block suspicious traffic. It felt like the right call for a project that's publicly accessible, even if it's just a side project, I didn't want it to be trivially abusable.

What I didn't fully think through was how it would behave behind a proxy. More on that in a moment.

---

## Bug #1: The Sleeping Cluster

When I came back to the project, nothing worked. Requests were failing at the database level. Mongoose couldn't connect at all.

Turns out MongoDB Atlas had **deactivated my cluster** due to inactivity. This is something Atlas does on the free tier after a certain period, and since I'd been cheerfully ignoring the Render failure emails, I had no idea it had happened.

The fix was simple: log into Atlas, reactivate the cluster, wait for it to spin back up. A few minutes and it was done.

I thought that was it. It wasn't.

---

## Bug #2: Arcjet Thought My Users Were Bots

Once the database was back online, I started testing the API and immediately hit a wall. Requests were being blocked, not by MongoDB this time, but by Arcjet. It was flagging legitimate requests as bot traffic.

This one took me a bit longer to figure out. The issue is about **how Render (and most cloud platforms) handle incoming traffic**. Your app doesn't receive requests directly from the client, they come through Render's proxy layer first. So the IP address that Arcjet sees isn't the actual user's IP, it's the proxy's IP. And a proxy IP, especially a shared one, can look very suspicious to a bot detection system.

The fix involves two things. First, telling Express to trust the proxy so it reads the real client IP from the `X-Forwarded-For` header:

```js
app.set("trust proxy", true);
```

Second, making sure Arcjet is configured to use the correct IP source. In my case I also needed to double-check that the `characteristics` field was set up to resolve properly in a proxied environment:

```js
const aj = arcjet({
  key: process.env.ARCJET_KEY,
  characteristics: ["ip.src"],
  rules: [
    shield({ mode: "LIVE" }),
    detectBot({
      mode: "LIVE",
      allow: ["CATEGORY:SEARCH_ENGINE"],
    }),
  ],
});
```

Once Express was correctly forwarding the real IP, Arcjet stopped treating everyone like a threat.

---

## What I Took Away From This

The MongoDB issue was entirely self-inflicted. I knew Render was sending failure emails and I chose to ignore them. On a production project that would have been a real problem. Lesson learned.

The Arcjet issue was more interesting. Security middleware doesn't exist in a vacuum. It depends on your deployment environment behaving in a predictable way, and "predictable" changes when you add a proxy layer. It's the kind of thing that works perfectly in local development and then silently breaks in production, or in this case, silently breaks when you wake a project back up after a few months.

Both bugs were fixable in under an hour once I actually sat down with them. The hard part was the two weeks I spent pretending the emails weren't there.

---

Sub-tracker is live at [sub-tracker-yzxy.onrender.com](https://sub-tracker-yzxy.onrender.com) — free tier, so give it 30–60 seconds if it's been sleeping. Code is on [GitHub](https://github.com/negarbaharmand/sub-tracker) if you want to look around.
