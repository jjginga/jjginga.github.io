---
layout: post
title: "ferrolearn — ml algorithms from scratch in rust + wasm"
date: 2026-06-13
description: a series reimplementing classic ml algorithms in rust, compiled to wasm, with interactive browser demos
tags: [rust, wasm, machine-learning, ferrolearn]
---

i'm currently doing the [master's degree in computational engineering and mathematics at urv](https://www.urv.cat/en/studies/master/courses/computational-engineering-mathematics/), and one of the subjects is **artificial intelligence**. we cover the usual suspects — linear regression, decision trees, SVMs, neural networks — and every time we finish a topic i find myself wanting to go deeper and understand a little bit more.

that's where this series comes from. i decided to reimplement everything from the course in rust, compile it to wasm, and build interactive browser demos so you can actually watch the algorithms do their thing. the project is called **ferrolearn**. the code lives at [github.com/jjginga/ferrolearn](https://github.com/jjginga/ferrolearn).

---

the plan is to cover these algorithms, roughly in course order:

| # | algorithm | status |
|---|-----------|--------|
| 0 | [exploratory data analysis](/blog/2026/ferrolearn-eda/) | ✅ done |
| 1 | [linear regression](/blog/2026/ferrolearn-linear_regression/) | ✅ done |
| 2 | [logistic regression](/blog/2026/ferrolearn-logistic_regression/) | ✅ done |
| 3 | [k-nearest neighbours](/blog/2026/ferrolearn-k-nearest_neighbors/) | ✅ done |
| 4 | decision trees | comming soon |
| 5 | random forest | — |
| 6 | adaboost | — |
| 7 | support vector machines | — |
| 8 | pca / eigenfaces | — |
| 9 | multilayer perceptron | — |
| 10 | genetic algorithms | — |
| 11 | deep q-network | — |


each post pairs with a live demo you can interact with directly in the browser. the rust code compiles to wasm via [wasm-pack](https://rustwasm.github.io/wasm-pack/) and the demos are built with vanilla js and d3. no framework, no bundler — just a script tag and a canvas.

i'm also learning rust as i go, so expect the posts to spend time on the language itself alongside the maths.

---

**source code:** [github.com/jjginga/ferrolearn](https://github.com/jjginga/ferrolearn){:target="_blank"}