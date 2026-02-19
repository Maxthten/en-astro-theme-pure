---
title: "RL Study Notes: SA and SGD"
publishDate: 2026-02-19 21:50:00
description: "A review of Stochastic Approximation and the Robbins-Monro algorithm, detailing the evolution, convergence properties, and sampling differences of SGD."
tags: ["Stochastic Approximation", "SGD", "Robbins-Monro", "Optimization", "Study Notes"]
language: "English"
---

# Stochastic Approximation Theory and Stochastic Gradient Descent

## Mean Estimation

First, consider the problem of estimating the sample mean:

$$
w_{k+1} \doteq \frac{1}{k} \sum_{i=1}^{k} x_{i}, \quad k=1,2, \dots
$$

For the mean at the previous step, we have:

$$
w_{k} = \frac{1}{k-1} \sum_{i=1}^{k-1} x_{i}, \quad k=2,3, \dots
$$

Through algebraic transformation, the full batch computation can be converted into a recursive form:

$$
w_{k+1} = \frac{1}{k} \sum_{i=1}^{k} x_{i} = \frac{1}{k}\left(\sum_{i=1}^{k-1} x_{i}+x_{k}\right) = \frac{1}{k}\left((k-1) w_{k}+x_{k}\right) = w_{k}-\frac{1}{k}\left(w_{k}-x_{k}\right)
$$

That is:

$$
w_{k+1} = w_k - \frac{1}{k}(w_k - x_k)
$$

* This is an iterative algorithm that does not require storing and recalculating all historical data.
* In the early stages of iteration, the estimate may be inaccurate due to insufficient sample size; however, as the sample size $k$ increases, the result gradually approaches the true mean.

From this, we can derive a more generalized iterative equation:

$$
w_{k+1} = w_{k} - \alpha_{k}(w_{k} - x_{k})
$$

* When the step size sequence $\{\alpha_k\}$ satisfies certain conditions, the estimate $w_k$ will still converge to the expectation $\mathbb{E}[X]$.
* This form can be viewed as a special case of the Stochastic Approximation (SA) algorithm, and it is also the foundational form of the Stochastic Gradient Descent (SGD) algorithm.

## Robbins-Monro (RM) Algorithm

### Stochastic Approximation (SA)

* SA is a broad class of algorithms that rely on stochastic iteration to find the roots of equations or solve optimization problems.
* The core advantage of SA is its **black-box solving capability**: it does not require knowledge of the analytical expression or global properties of the target equation, relying only on noisy observational data to update parameters.

### RM Algorithm Definition

A necessary condition for finding the extremum of a function is that the gradient is zero, i.e., $g(w) \doteq \nabla_{w} J(w) = 0$. Similarly, for the case of $g(w) = c$, it can be transformed into a root-finding problem by moving the terms. SGD is exactly a special case of the RM algorithm.

The RM algorithm solves $g(w) = 0$ through the following iterative format:

$$
w_{k+1} = w_k - a_k \tilde{g}(w_k, \eta_k), \quad k = 1, 2, 3, \dots
$$

Where:

* $w_k$ is the estimate of the root at the $k$-th iteration.
* $\tilde{g}(w_k, \eta_k) = g(w_k) + \eta_k$ is the $k$-th observation with random noise $\eta_k$.
* $a_k$ is a positive coefficient controlling the step size.

### Convergence Theorem and Condition Analysis

If the following conditions hold:

(a) $0 < c_1 \leq \nabla_w g(w) \leq c_2$ for all $w$;
(b) $\sum_{k=1}^{\infty} a_k = \infty$ and $\sum_{k=1}^{\infty} a_k^2 < \infty$;
(c) $\mathbb{E}[\eta_k | \mathcal{H}_k] = 0$ and $\mathbb{E}[\eta_k^2 | \mathcal{H}_k] < \infty$;

Where $\mathcal{H}_k = \{w_k, w_{k-1}, \dots\}$ represents the history up to time $k$, then the sequence $w_k$ will converge **almost surely** (with probability 1) to the root $w^*$ satisfying $g(w^*) = 0$.

* **Condition (a)**: Requires the derivative (or gradient) of the function $g(w)$ to have strict upper and lower bounds. This ensures the function is sufficiently smooth and its slope will not approach zero or infinity, thereby ensuring the update direction points stably and continuously towards the target solution $w^*$.
* **Condition (b)**: The classic step size (learning rate) constraint. $\sum a_k = \infty$ ensures the total step size is infinite, giving the algorithm the capability to cover any initial distance to reach the target point; $\sum a_k^2 < \infty$ ensures the step size decays fast enough so the algorithm can ultimately converge without oscillating perpetually around the root.
* **Condition (c)**: Constraints on the properties of the random noise. A conditional expectation of zero given the historical sequence (forming a martingale difference sequence) implies the noise estimation is unbiased; a finite conditional variance limits the fluctuation amplitude during single-step exploration, preventing the algorithm from diverging.

## Stochastic Gradient Descent (SGD)

SGD is primarily used to solve expected risk minimization problems:

$$
\min_{w} \quad J(w) = \mathbb{E}[f(w, X)]
$$

* $w$ is the parameter to be optimized.
* $X$ is a random variable, and the expectation is calculated with respect to the distribution of $X$.
* The function $f(\cdot)$ outputs a scalar, and both the parameter $w$ and input $X$ can be scalars or vectors.

### Gradient Descent Evolution

**1. Gradient Descent (GD)**

$$
w_{k+1} = w_k - \alpha_k \nabla_w \mathbb{E}[f(w_k, X)] = w_k - \alpha_k \mathbb{E}[\nabla_w f(w_k, X)]
$$

**2. Batch Gradient Descent (BGD)**

Approximating the mathematical expectation through the empirical mean of a finite sample:

$$
\mathbb{E}[\nabla_w f(w_k, X)] \approx \frac{1}{n} \sum_{i=1}^n \nabla_w f(w_k, x_i)
$$

$$
w_{k+1} = w_k - \alpha_k \frac{1}{n} \sum_{i=1}^n \nabla_w f(w_k, x_i)
$$

**3. Stochastic Gradient Descent (SGD)**

$$
w_{k+1} = w_k - \alpha_k \nabla_w f(w_k, x_k)
$$

* Compared to GD: Replaces the true gradient $\mathbb{E}[\nabla_w f(w_k, X)]$ with the single-sample stochastic gradient $\nabla_w f(w_k, x_k)$.
* Compared to BGD: Equivalent to setting the batch size to $n = 1$.

According to RM algorithm theory, if the Hessian matrix is positive definite and bounded ($0 < c_1 \le \nabla^2_w f(w, X) \le c_2$), the learning rate satisfies the Robbins-Monro sequence conditions, and the sample sequence is independent and identically distributed (i.i.d.), then SGD will also converge almost surely (with probability 1) to the optimal solution.

### SGD Relative Error and Randomness

Introduce the relative error $\delta_k$ to measure the degree to which the stochastic gradient deviates from the true gradient:

$$
\delta_k \doteq \frac{|\nabla_w f(w_k, x_k) - \mathbb{E}[\nabla_w f(w_k, X)]|}{|\mathbb{E}[\nabla_w f(w_k, X)]|}
$$

At the optimal solution $w^*$, $\mathbb{E}[\nabla_w f(w^*, X)] = 0$ holds. Substituting this into the denominator and applying the Mean Value Theorem for Integrals:

$$
\delta_k = \frac{|\nabla_w f(w_k, x_k) - \mathbb{E}[\nabla_w f(w_k, X)]|}{|\mathbb{E}[\nabla_w f(w_k, X)] - \mathbb{E}[\nabla_w f(w^*, X)]|} = \frac{|\nabla_w f(w_k, x_k) - \mathbb{E}[\nabla_w f(w_k, X)]|}{|\mathbb{E}[\nabla_w^2 f(\tilde{w}_k, X)(w_k - w^*)]|}
$$

Due to the strong convexity assumption $\nabla_w^2 f \ge c > 0$, we can bound the denominator:

$$
\begin{aligned}
|\mathbb{E}[\nabla_w^2 f(\tilde{w}_k, X)(w_k - w^*)]| &= |\mathbb{E}[\nabla_w^2 f(\tilde{w}_k, X)] \cdot (w_k - w^*)| \\
&= |\mathbb{E}[\nabla_w^2 f(\tilde{w}_k, X)]| \cdot |w_k - w^*| \ge c|w_k - w^*|
\end{aligned}
$$

Thus, we obtain the upper bound of the relative error:

$$
\delta_k \leq \frac{|\overbrace{\nabla_w f(w_k, x_k)}^{\text{stochastic gradient}} - \overbrace{\mathbb{E}[\nabla_w f(w_k, X)]}^{\text{true gradient}}|}{\underbrace{c|w_k - w^*|}_{\text{distance to optimal solution}}}
$$

This inequality strictly reveals an important convergence pattern of SGD:

* The relative error $\delta_k$ is inversely proportional to the distance to the optimal solution $|w_k - w^*|$.
* When $w_k$ is far from the optimal solution, the denominator is large, and $\delta_k$ is small. At this point, the update direction of SGD is very close to the true gradient, exhibiting a descent trajectory highly similar to GD.
* As $w_k$ gradually approaches the optimal solution $w^*$, the denominator tends to zero, and the relative error $\delta_k$ increases significantly. This means that in the neighborhood of the optimal solution, noise interference becomes dominant, causing the algorithm to exhibit strong random oscillation in the late stages of convergence (which is also why SGD requires learning rate decay).

## Sampling Comparison: BGD, MBGD, and SGD

> This random sampling strategy shares similarities with truncated methods.

Assume we want to minimize $J(w) = \mathbb{E}[f(w, X)]$ and have a set of random samples $\{x_i\}_{i=1}^n$ for $X$. The iterative formulas for the three algorithms are compared as follows:

$$
w_{k+1} = w_k - \alpha_k \frac{1}{n} \sum_{i=1}^n \nabla_w f(w_k, x_i) \quad \text{(BGD)}
$$

$$
w_{k+1} = w_k - \alpha_k \frac{1}{m} \sum_{j \in \mathcal{I}_k} \nabla_w f(w_k, x_j) \quad \text{(MBGD)}
$$

$$
w_{k+1} = w_k - \alpha_k \nabla_w f(w_k, x_k) \quad \text{(SGD)}
$$

* **BGD**: Computes the full $n$ samples at each iteration. When $n$ is sufficiently large, the update direction closely approximates the true expected gradient.
* **MBGD**: Draws a subset $\mathcal{I}_k$ of size $m$ from the global samples at each iteration. This set is obtained through $m$ independent and identically distributed (i.i.d.) samples.
* **SGD**: Randomly draws a single sample $x_k$ at time step $k$ during each iteration.

> **Core Difference Note**: Even when $m=n$, MBGD is not equivalent to BGD. This is because the $m$ random samples in MBGD are typically drawn with replacement (potentially drawing duplicate samples), whereas BGD strictly iterates over all unique global sample data.
