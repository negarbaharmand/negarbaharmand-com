---
author: Negar Baharmand
pubDatetime: 2023-10-24T22:00:00.000Z
title: Virtual environment in Python
slug: virtual-environment-python
featured: true
draft: false
tags:
  - python
description: Talking about the convenience of Python's virtual environments—a helpful toolbox that keeps your projects tidy by managing dependencies and versions separately, ensuring a stable system.
---

A virtual environment is like a handy toolbox for Python developers. It helps us keep our different projects organized and prevents them from stepping on each other's toes, especially when they use different versions of the same tool.

This tool is useful because, well, let's say you start one project today and another one next month. They might need different versions of some software, and a virtual environment lets them peacefully coexist.

Imagine you have a collection of tools, and you want a separate box for each project. That's what a virtual environment does—it keeps the tools (or dependencies) for each project neatly tucked away in its own box.

Now, here's how you get this tool (virtual environment) on your computer: open your terminal and type:

```shell
pip install virtualenv
```

Once you've got it, creating a new virtual environment for your project is as simple as typing:

```shell
python<version> -m venv <virtual-environment-name>
```

And when you're ready to work on your project, you just need to activate your virtual environment:

```shell
source env/bin/activate
```

This keeps everything nice and tidy, making sure your projects run smoothly without causing chaos on your computer. Source: freecodecamp.org
