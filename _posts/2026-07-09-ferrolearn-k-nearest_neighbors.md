---
layout: post
title: "ferrolearn #3 — k-nearest neighbors"
date: 2026-07-09
description: implementing k-nearest neighbors from scratch in rust and wasm — euclidean distance, majority voting, choosing k by cross-validation, and what the abalone dataset teaches us about lazy, instance-based learning.
tags: [ferrolearn, rust, wasm, machine-learning, knn]
thumbnail: assets/img/ferrolearn-knn-grid_search.png
---

The first two models in this series learned a set of weights: linear and logistic regression both ran gradient descent to fit parameters, then threw the training data away. K-nearest neighbors does the opposite. It learns nothing — no weights, no equation, no optimization. It keeps the training data around and classifies a new point by looking at the examples nearest to it. This makes it the simplest model in the series to state and, in some ways, the most different.

We apply it to the abalone dataset, predicting sex from physical measurements. Unlike the [logistic regression post](https://jjginga.com/blog/2026/ferrolearn-logistic_regression/), which dropped infants to get a binary problem, here we keep all three categories — M, F, and I — because majority voting handles any number of classes without changing anything. That makes this the first genuinely multi-class problem in the series.

→ [open the interactive demo](https://jjginga.github.io/ferrolearn/web/demos/knn/){:target="_blank"}

---

## the model

The idea is almost embarrassingly simple. To classify a new abalone, find the $k$ training abalone most similar to it, and let them vote — the majority class among those $k$ neighbors is the prediction.

"Most similar" means closest in feature space. We measure closeness with **Euclidean distance** — the straight-line distance between two points across all $n$ features:

$$d(\mathbf{x}, \mathbf{x}_i) = \sqrt{\sum_{j=1}^{n} (x_j - x_{ij})^2}$$

where $\mathbf{x}$ is the point we want to classify and $\mathbf{x}_i$ is one training sample. We compute this distance to every training point, sort ascending, take the $k$ smallest, and return the most common label among them:

$$\hat{y} = \text{mode}\{\, y_{(1)}, y_{(2)}, \ldots, y_{(k)} \,\}$$

where $y_{(1)}, \ldots, y_{(k)}$ are the labels of the $k$ nearest neighbors. Because it is just a count, this generalizes to three classes — or thirty — with no extra machinery, which is why we can keep infants in the problem.

**No training, no parameters.** This is the defining feature of KNN. There is no gradient descent, no loss function, nothing to minimize. `fit()` only stores the (normalized) training data; all the actual work happens at prediction time, when the distances are computed. This style is called **lazy** or **instance-based** learning: the model *is* the training set. It is the mirror image of the regression models, which spent all their effort in `fit()` and then predicted instantly.

**The decision boundary is jagged.** KNN never writes down a boundary, but one exists implicitly. If you colored every point in feature space by the class KNN would assign it, you would see regions — one per class — and the borders between them are not straight. They bend and wander around individual training points. At $k=1$ the regions are the Voronoi cells around each training sample, maximally irregular; as $k$ grows they smooth out. This is the sharp contrast with logistic regression, whose boundary is always a single flat hyperplane. KNN can carve arbitrary, curved regions — it trades the linear model's rigidity for flexibility. As we will see, that flexibility is also its weakness.

> **regression, too.** KNN is not only a classifier. To predict a continuous target — ring count, say — you swap the majority vote for the *mean* of the $k$ neighbors' values. Everything else stays identical. This post stays on classification, but the same code is one line away from the regression setting.
{: .block-tip }

---

## distance and normalization

Euclidean distance sums contributions from every feature, which means a feature's *scale* decides how much it matters. The abalone measurements live on very different scales — weights in grams, lengths in millimetres, ring counts as integers up to 29 — so without rescaling, whichever feature has the largest numeric range would dominate the distance and effectively pick the neighbors on its own. The others would barely register.

KNN is therefore even more sensitive to feature scaling than the gradient-descent models, where scaling only affected convergence speed. Here it changes the answer. So we standardize every feature to zero mean and unit variance before computing any distance:

$$z = \frac{x - \mu}{\sigma}$$

> **data leakage.** as in the regression posts, $\mu$ and $\sigma$ are computed from the **training set only** and reapplied to the test data without recomputing — see [linear regression](https://jjginga.com/blog/2026/ferrolearn-linear_regression/#normalizing-features). Computing them from the full dataset would leak information about the held-out points into the model.
{: .block-warning }

With every feature on the same scale, a millimetre of length and a gram of shell weight contribute comparably to the distance, and the neighbors are chosen on overall similarity rather than on whichever column happened to have the biggest numbers.

---

## choosing k

$k$ — the number of neighbors — is the model's only knob, and it controls the bias–variance tradeoff directly and visibly.

**Small $k$** makes the model sensitive to individual points. At $k=1$, each prediction is simply the label of the single closest training sample; the decision regions are the jagged Voronoi cells described above. This fits the training data perfectly — including its noise — which is high variance: move one training point and the boundary shifts. On unseen data it is brittle.

**Large $k$** averages over a wide neighborhood. The boundary smooths, small local structures wash out, and in the limit $k \to m$ every query just returns the global majority class. That is high bias: the model is too rigid to follow the real structure.

The right $k$ sits between these extremes, and we find it by cross-validation rather than by guessing.

**Ties.** With two classes an odd $k$ can never tie, but with three (M, F, I) it still can — $k=5$ might split 2–2–1. We break such ties toward the nearest neighbor: among the classes with the top count, the one whose closest member is nearest wins. This keeps predictions deterministic and gently favors the more similar class.

---

## k-fold cross-validation

We cannot choose $k$ by training accuracy, and KNN makes the reason especially stark: because the model *is* the training set, predicting on the training data lets every point find itself as its own nearest neighbor (distance zero). At $k=1$ that gives 100% training accuracy — a number that says nothing about how the model generalizes.

So we evaluate on held-out data. The mechanism is the same k-fold cross-validation used in the [regression posts](https://jjginga.com/blog/2026/ferrolearn-linear_regression/#k-fold-cross-validation): split the data into folds, and for each candidate neighbor count, train on the other folds and score on the held-out one, then average the scores.

One naming collision is worth flagging: the $k$ in "k-nearest neighbors" (the number of neighbors) is not the $k$ in "k-fold" (the number of folds). We use 5 folds to choose the best number of neighbors.

---

## grid search

To pick the neighbor count automatically, we sweep a range of values, score each by its average validation accuracy across the folds, and keep the best. It is the same procedure as the $\lambda$ grid search in the earlier posts — only the hyperparameter has changed: we are searching over the number of neighbors instead of a regularization strength, and selecting the value with the **highest** validation accuracy.

> **a note on running time.** KNN flips the cost structure of the series. There is no training loop — `fit()` just stores the data — but every prediction compares the query against *all* $m$ training points, so scoring a whole set is $O(m^2)$. That makes the grid search, which re-scores across folds and neighbor counts, the expensive part. Reduce the **k max** or **cv folds** sliders if you want faster results; raise them for a finer search.
{: .block-tip }

---

## interpreting this run

<!--
  Analysis to write. Figures below are the ones the demo produces:
  confusion matrix, per-class accuracy, prediction distribution, and the
  k-vs-accuracy grid search. Fill in the discussion under each.
-->

**k = ? found by grid search · 5-fold cross-validation · z-score normalization**

---

**Confusion matrix**

Rows are the actual class, columns the predicted class; each row sums to 100%, and a darker diagonal means better classification.

<img src="{{ site.baseurl }}/assets/img/ferrolearn-knn-confusion.png"
     style="max-width: 420px; display: block; margin: 2rem auto;">

---

**Per-class accuracy**

The fraction of samples correctly classified within each sex category — this is the diagonal of the confusion matrix, read one class at a time.

<img src="{{ site.baseurl }}/assets/img/ferrolearn-knn-per_class.png"
     style="max-width: 420px; display: block; margin: 2rem auto;">

---

**Prediction distribution**

For each actual class, where the model's predictions land — the off-diagonal dots are where confusion happens.

<img src="{{ site.baseurl }}/assets/img/ferrolearn-knn-pred_dist.png"
     style="max-width: 420px; display: block; margin: 2rem auto;">

---

**Grid search — k vs validation accuracy**

Cross-validation accuracy across the candidate neighbor counts, with the best $k$ highlighted.

<img src="{{ site.baseurl }}/assets/img/ferrolearn-knn-grid_search.png"
     style="max-width: 420px; display: block; margin: 2rem auto;">

---

## where KNN struggles

**Every feature counts equally.** Distance treats all features the same, so an irrelevant or noisy feature adds noise to every comparison. A linear model can drive a useless coefficient to zero; KNN has no such mechanism — it cannot down-weight a feature, only be dragged around by it. Careful feature selection and scaling matter more here than in any model so far.

**No model, no explanation.** KNN returns predictions but learns no structure. There are no weights to interpret, no equation, nothing to tell you *why* two sexes are hard to separate — only that the neighbors disagreed. The regression posts at least handed us coefficients to argue over; KNN hands us nothing but the data itself.

**The cost lives at prediction time.** Lazy learning has a price: there is no compact model to ship, and every prediction scans the whole training set. Cost grows with the data, exactly where the regression models stayed constant.

None of this makes KNN a bad choice — on the right problem, with few, well-scaled, relevant features, a model that simply asks "what do the nearest examples look like?" is hard to beat. It just asks more of the data, and of us, than a model that compresses everything into a handful of weights.

---

## references

- Russell, S. and Norvig, P. *Artificial Intelligence: A Modern Approach*, 4th ed. Pearson, 2021. §12.6 (nearest-neighbor models), §19.7.1 (nonparametric classification).

- James, G., Witten, D., Hastie, T., Tibshirani, R., and Taylor, J. *An Introduction to Statistical Learning with Applications in Python*. Springer, 2023. §3.5 (KNN regression), §4.4.4 (KNN classification).

- [K-Nearest Neighbor (KNN) Algorithm — GeeksForGeeks](https://www.geeksforgeeks.org/machine-learning/k-nearest-neighbours/)

- [The Bias-Variance Tradeoff — Towards Data Science](https://towardsdatascience.com/the-bias-variance-tradeoff-cf18d3ec54f9/)

---

→ [open the interactive demo](https://jjginga.github.io/ferrolearn/web/demos/knn/){:target="_blank"}

**source code:** [github.com/jjginga/ferrolearn](https://github.com/jjginga/ferrolearn){:target="_blank"}