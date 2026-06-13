---
layout: post
title: "ferrolearn #0 — exploratory data analysis with rust + wasm"
date: 2026-06-13
description: before we train anything, we look at the data — parsing abalone shell measurements in rust and visualising the results in the browser
thumbnail: assets/img/abalone.jpg
tags: [rust, wasm, machine-learning, ferrolearn]
---

*this post is part of the [ferrolearn series](/blog/2026/ferrolearn-series/), where i reimplement ml algorithms from my master's course in rust + wasm.*

---

before training a single model, you have to look at the data. not glance at it — actually look. every assumption you make about what algorithm to use, what features matter, what preprocessing is needed, lives or dies on what the data actually looks like. this first post is about that: exploratory data analysis on the abalone dataset, with all the computation running in rust compiled to webassembly.

## the dataset

abalone are marine molluscs. they live on rocky coasts, graze on algae, and happen to have been the subject of a 1994 population biology study out of tasmania by nash, sellers, talbot, cawthorn and ford. their data has since become one of the classic regression benchmarks on the [uci machine learning repository](https://archive.ics.uci.edu/dataset/1/abalone).

the problem is this: how old is an abalone? you can find out exactly by cutting the shell through the cone, staining a cross-section, and counting the growth rings under a microscope — the same way you'd age a tree. rings + 1.5 gives the age in years. but that process is slow, destructive, and requires lab equipment. the question is whether you can get a good estimate just from physical measurements taken in the field.

the dataset has 4177 samples and 8 features: sex (male, female, or infant), longest shell length, diameter perpendicular to length, height with meat in shell, whole weight, shucked weight (meat only), viscera weight (gut after bleeding), and shell weight after drying. **the target is the ring count**.

![abalone]({{ site.baseurl }}/assets/img/abalone.jpg)

## why eda first

a few things you can only learn by looking. is the target normally distributed or heavily skewed? are there outliers that will distort training? do features have enough variance to carry signal, or are some nearly constant? are any features so correlated that they'll cause problems for certain models?

these aren't rhetorical questions. the answers change what you build.

## interactive demo

the eda below runs entirely in rust compiled to wasm. the rust code parses the csv, computes summary statistics, and calculates the correlation matrix — all client-side. d3 handles the visualisation.

<div style="width:100%; height:700px; border:1px solid #ddd; border-radius:4px; overflow:hidden;">
  <iframe 
    src="https://jjginga.github.io/ferrolearn/web/demos/abalone_eda/" 
    style="width:100%; height:100%; border:none;"
    title="Abalone EDA demo">
  </iframe>
</div>

## what the data is telling us

**the target is well-behaved.** the rings histogram is roughly bell-shaped, centred around 9–10 rings (10.5–11.5 years), with a modest right tail out to 29. this is good news for linear regression — we're not fighting a heavily skewed or bimodal target. it won't be trivial either: there's genuine spread, and the upper tail thins out fast, which means the model will see very few examples of old abalone.

**the infant category is an age effect, not a sex effect.** the bar chart of rings by sex group shows infants with systematically fewer rings than males and females, which makes biological sense — they haven't grown as long. but this matters for modelling: sex isn't a clean categorical feature on equal footing with the others. "I" encodes age information directly. if you one-hot encode sex naively and throw it into a regression alongside the physical measurements, you're doubling up on the age signal in a way that's hard to reason about.

**the physical measurements are highly collinear.** the correlation matrix makes this immediately obvious: length, diameter, whole weight, shucked weight, viscera weight, and shell weight are all correlated with each other at 0.9 or higher. this is expected — bigger shells are heavier in every dimension. but it has a direct consequence for linear regression: **when predictors are this correlated, the coefficient estimates become unstable**. a small change in the data can produce wildly different weights. this is exactly the problem regularisation (l1 and l2) exists to solve, and it's why we'll spend time on it in the next post before fitting anything.

**shell weight is the strongest individual predictor of rings.** this makes biological sense — shell mass accumulates continuously over an abalone's life, so it carries more age signal than a single linear dimension. you can see this in the scatter plots: shell weight vs rings shows the clearest trend of any single feature.

**height has outliers.** there are a handful of samples with height values near zero or suspiciously large. the histogram makes them visible as isolated bars away from the main distribution. the uci page notes that examples with missing values were removed, but it doesn't say anything about measurement errors in height specifically. for now i've kept them — our rust parser validates field count and type but doesn't filter by range — but it's worth knowing they're there.

## what this sets up

the collinearity issue is the key takeaway going into linear regression. when predictors move together, the model can't isolate the contribution of each one — there are infinitely many weight combinations that produce the same predictions on the training set, and tiny perturbations send the weights in different directions. l2 regularisation (ridge) addresses this by penalising large weights, shrinking correlated coefficients toward each other rather than letting them cancel out. we'll implement gradient descent with both l1 and l2 penalties, and use cross-validation to pick the regularisation strength.

next post: linear regression, from scratch, with a gradient descent animation.