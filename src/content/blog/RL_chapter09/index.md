---
title: "RL Study Notes: Policy Gradient Methods"
publishDate: 2026-02-22 16:30:00
description: "Core concepts of RL policy gradient methods: objective functions, the log-derivative trick, theorem derivation, and the REINFORCE algorithm."
tags: ["Reinforcement Learning", "Policy Gradient", "REINFORCE", "Study Notes"]
language: "English"
---

# Policy Gradient Methods

In policy gradient methods, the policy is represented as a parameterized function.

$$
\pi(a|s,\theta)
$$

Where $\theta \in \mathbb{R}^m$ is the parameter vector.

* This function can be a neural network, with the state $s$ as input and the probabilities of taking each action as output, parameterized by $\theta$.
* When the state space is large, tabular representation is inefficient in terms of storage and generalization. Function approximation can effectively solve this problem.
* The function representation is also commonly written as $\pi(a, s, \theta)$, $\pi_\theta(a|s)$, or $\pi_\theta(a, s)$.

### Basic Idea

Define an objective function (e.g., $J(\theta)$) to evaluate the performance of the policy.

Use gradient ascent to update the parameters and find the optimal policy:

$$
\theta_{t+1} = \theta_t + \alpha \nabla_\theta J(\theta_t)
$$

## Objective Functions (Metrics)

### 1. Weighted Average of State Values

$$
\bar{v}_{\pi} = \sum_{s \in \mathcal{S}} d(s) v_{\pi}(s)
$$

* $\bar{v}_{\pi}$ is a weighted average.
* $d(s) \ge 0$ is the weight of state $s$, which can be understood as the probability distribution of state occurrences.
* Expressed in expectation form: $\bar{v}_{\pi} = \mathbb{E}[v_{\pi}(S)]$.

Its vector form is:

$$
\bar{v}_{\pi} = d^T v_{\pi}
$$

Where $v_{\pi} \in \mathbb{R}^{|\mathcal{S}|}$ and $d \in \mathbb{R}^{|\mathcal{S}|}$.

In episodic tasks, the objective function can be defined as the expected return starting from the initial state:

$$
\begin{aligned}
J(\theta) &= \mathbb{E} \left[ \sum_{t=0}^{\infty} \gamma^t R_{t+1} \right] \\
&= \sum_{s \in \mathcal{S}} d_0(s) v_\pi(s)
\end{aligned}
$$

**Regarding the setting of the weight $d(s)$:**

* **Independent of policy $\pi$:** In episodic tasks, $d$ is usually set to the initial state distribution $d_0$. For example, if all states are considered equally important, $d_0(s) = 1/|\mathcal{S}|$, or if only a specific initial state $s_0$ is of interest, $d_0(s_0) = 1$.
* **Dependent on policy $\pi$:** In continuing tasks, $d$ depends on the policy $\pi$, and the stationary distribution $d_\pi$ is typically chosen. The stationary distribution satisfies $d_{\pi}^T P_{\pi} = d_{\pi}^T$, where $P_{\pi}$ is the state transition matrix.

### 2. Average One-Step Reward (Average Reward)

$$
\bar{r}_\pi = \sum_{s \in \mathcal{S}} d_\pi(s) r_\pi(s) = \mathbb{E}[r_\pi(S)]
$$

Where state $S \sim d_\pi$. The expected immediate reward in state $s$ is:

$$
r_\pi(s) = \sum_{a \in \mathcal{A}} \pi(a|s) r(s, a)
$$

* The weight $d_\pi$ is the stationary distribution.
* $\bar{r}_\pi$ is the weighted average of the one-step immediate rewards.

The average reward can also be defined as the limit form of long-term rewards:

$$
\begin{aligned}
\lim_{n \to \infty} \frac{1}{n} \mathbb{E} \left[ \sum_{k=1}^{n} R_{t+k} \mid S_t = s_0 \right] &= \sum_{s \in \mathcal{S}} d_\pi(s) r_\pi(s) = \bar{r}_\pi
\end{aligned}
$$

Here, the influence of the initial state $s_0$ is eliminated in the limit, making the two definitions equivalent.

**Metrics Comparison:**

* The above metrics all depend on the policy $\pi$, so they are essentially functions of the parameter $\theta$.
* Intuitively, $\bar{r}_\pi$ focuses more on immediate rewards (myopic), while $\bar{v}_\pi$ cares more about long-term returns.
* In the discounted case with a discount factor $\gamma$, there is a mathematical relationship between the two: $\bar{r}_\pi = (1 - \gamma) \bar{v}_\pi$.

## Policy Gradient Theorem (Gradients of the Metrics)

Whether the objective function is $\bar{v}_{\pi}$ or $\bar{r}_{\pi}$, its gradient can be uniformly expressed in the following form (proportional to):

$$
\nabla_{\theta} J(\theta) \propto \sum_{s \in \mathcal{S}} \eta(s) \sum_{a \in \mathcal{A}} \nabla_{\theta} \pi(a|s, \theta) q_{\pi}(s, a)
$$

Where $\eta$ is the distribution weight of the states.

### Derivation of Gradients and Transformation to Expectation

**1. Core Objective:** To transform the complex "double summation over states and actions" form into a "mathematical expectation" form that allows for approximate calculation through data sampling.

**2. Log-Derivative Trick:** According to the calculus rule, taking the natural logarithm of the policy function and computing the gradient yields:

$$
\nabla_{\theta} \ln \pi(a|s, \theta) = \frac{1}{\pi(a|s, \theta)} \nabla_{\theta} \pi(a|s, \theta)
$$

Rearranging the terms gives:

$$
\nabla_{\theta} \pi(a|s, \theta) = \pi(a|s, \theta) \nabla_{\theta} \ln \pi(a|s, \theta)
$$

This step ingeniously constructs the probability term $\pi(a|s, \theta)$ out of nowhere, which is a prerequisite for transforming the formula into an expectation.

**3. Formula Substitution:** Substituting the above result into the gradient formula:

$$
\nabla_{\theta} J(\theta) \propto \sum_{s \in \mathcal{S}} \eta(s) \sum_{a \in \mathcal{A}} \pi(a|s, \theta) \nabla_{\theta} \ln \pi(a|s, \theta) q_{\pi}(s, a)
$$

**4. Transformation to Mathematical Expectation:** The equation above contains a two-layer probability-weighted summation (the outer layer based on the state distribution $\eta(s)$, and the inner layer based on the action probability $\pi$), which is equivalent to the mathematical expectation over the random variables $S$ and $A$:

$$
\nabla_{\theta} J(\theta) = \mathbb{E}[\nabla_{\theta} \ln \pi(A|S, \theta) q_{\pi}(S, A)]
$$

**5. Practical Significance:** After completing this mathematical transformation, the algorithm no longer needs to iterate through all states and actions in the environment. The agent only needs to explore the environment according to the current policy, and the collected trajectory data $(S, A)$ will naturally follow this expected probability distribution, thus providing a theoretical basis for single-step sampling approximation.

If sampling is used to approximate the gradient, the single-step update direction is:

$$
\nabla_{\theta} J \approx \nabla_{\theta} \ln \pi(a|s, \theta) q_{\pi}(s, a)
$$

## Parameterization of the Policy Function (Softmax)

To ensure the probability properties $\pi(a|s, \theta) > 0$ and the sum of probabilities equals 1, the Softmax function is commonly used to map real-valued preferences into probabilities.

For any vector $x = [x_1, \dots, x_n]^T$:

$$
z_i = \frac{e^{x_i}}{\sum_{j=1}^n e^{x_j}}
$$

Where $z_i \in (0, 1)$ and $\sum_{i=1}^n z_i = 1$.

Applied to the policy function:

$$
\pi(a|s, \theta) = \frac{e^{h(s,a,\theta)}}{\sum_{a' \in \mathcal{A}} e^{h(s,a',\theta)}}
$$

Where $h(s, a, \theta)$ is the action preference function, which can be parameterized by a neural network.

## REINFORCE and Policy Optimization Algorithms

Using the true gradient to maximize the objective function:

$$
\begin{aligned}
\theta_{t+1} &= \theta_t + \alpha \nabla_{\theta} J(\theta) \\
&= \theta_t + \alpha \mathbb{E}[\nabla_{\theta} \ln \pi(A|S, \theta_t) q_{\pi}(S, A)]
\end{aligned}
$$

In practical applications, stochastic gradients are used for the update:

$$
\theta_{t+1} = \theta_t + \alpha \nabla_{\theta} \ln \pi(a_t|s_t, \theta_t) q_{\pi}(s_t, a_t)
$$

Since the true action-value function $q_\pi$ is unknown, it needs to be approximately estimated:

* **REINFORCE Algorithm:** Uses the Monte Carlo method, taking the full trajectory return $G_t$ as an unbiased estimate of $q_\pi(s_t, a_t)$ for gradient updates.
* **Actor-Critic Algorithm:** Combines with Temporal Difference (TD) algorithms to train a value function network to approximately estimate $q_\pi(s_t, a_t)$.

Combining the reverse expansion of the log-derivative formula $\nabla_{\theta} \ln \pi = \frac{\nabla_{\theta} \pi}{\pi}$, the parameter update rule can be intuitively rewritten as:

$$
\begin{aligned}
\theta_{t+1} &= \theta_t + \alpha \nabla_{\theta} \ln \pi(a_t | s_t, \theta_t) q_t(s_t, a_t) \\
&= \theta_t + \alpha \underbrace{\left( \frac{q_t(s_t, a_t)}{\pi(a_t | s_t, \theta_t)} \right)}_{\beta_t} \nabla_{\theta} \pi(a_t | s_t, \theta_t)
\end{aligned}
$$

That is:

$$
\theta_{t+1} = \theta_t + \alpha \beta_t \nabla_{\theta} \pi(a_t | s_t, \theta_t)
$$

The coefficient $\beta_t$ here balances exploration and exploitation very well: it is proportional to the return $q_t$ (encouraging high-return actions) and inversely proportional to the action probability $\pi$ (giving larger update steps to rare actions, thereby encouraging exploration).
