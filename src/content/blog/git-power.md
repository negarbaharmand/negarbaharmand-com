---
author: Negar Baharmand
pubDatetime: 2022-11-27T23:00:00.000Z
title: Why is Git so incredible?
slug: git-power
featured: true
draft: false
tags:
  - thoughts
  - git
description: Git is like a magic wand for developers, offering local freedom and smart.
---

![](/src/assets/images/git-unsplash.jpg)
I believe that Git is brilliant because it allows developers to work locally at will, and synchronize centrally only as needed. Making a new commit is like opening an entry within the history so you can get back to it whenever required.

Something that I recently learned about Git is that you can edit your last commit if haven’t pushed it to a remote repository yet using:

```shell
git commit --amend -m "an updated commit message"
```

Another cool action that I learned is **git stash** which temporarily shelves changes you’ve made to your working copy so you can work on something else, and then come back and re-apply them later on. It’s useful when you are in the middle of something and need to quickly switch to another context to work on for a while.

![](/src/assets/images/git-vscode.png)
