---
title: "RL Study Notes: Monte Carlo Methods"
publishDate: 2026-02-18 20:00:00
description: "RL Monte Carlo methods: MC Basic, Exploring Starts, GPI, and epsilon-Greedy for model-free optimization."
tags: ["Reinforcement Learning", "Monte Carlo Methods", "GPI", "Epsilon-Greedy", "Study Notes"]
language: "English"
---

# Monte Carlo Methods

## The Simplest MC-based RL Algorithm: MC Basic

* **Core Process**: Starting from a state-action pair $(s, a)$, follow a policy $\pi_k$ to generate an episode.

* **Return Calculation**:
  
  * The (discounted) return obtained from this episode is denoted as $g(s,a)$.
  * $g(s,a)$ is a sample of $G_t$, meaning it is a sample of $q_{\pi_k}(s, a) = \mathbb{E}[G_t | S_t = s, A_t = a]$.

* **Law of Large Numbers Estimation**: If many episodes generate a set $\{g^{(j)}(s, a)\}$, then:

$$
q_{\pi_k}(s, a) = \mathbb{E}[G_t | S_t = s, A_t = a] \approx \frac{1}{N} \sum_{i=1}^{N} g^{(i)}(s, a)
$$

* **Core Idea**: When there is no Model, data is required; when there is no data, patterns are required—specifically, Experience.

* **Positioning**: MC Basic is a variant of the Policy Iteration algorithm that removes the model-based components.

* **Drawbacks**: MC Basic is too inefficient and is rarely used directly in practice.

### Considerations on Episode Length

* **Too Short**: Only states that are close enough can find the optimal policy.
* **Increasing Length**: As the length increases, the optimal policy can eventually be found.
* **Conclusion**: Episodes must be sufficiently long, but do not need to be infinitely long.

## MC Exploring Starts

### Sampling Sequence Example

$$
s_1 \xrightarrow{a_2} s_2 \xrightarrow{a_4} s_1 \xrightarrow{a_2} s_2 \xrightarrow{a_3} s_5 \xrightarrow{a_1} \dots
$$

### Definition of a Visit

In an episode, every occurrence of a state-action pair is called a **visit**.
Based on the decomposition of the sequence above:

$$
\begin{aligned}
    s_1 \xrightarrow{a_2} s_2 \xrightarrow{a_4} s_1 \xrightarrow{a_2} s_2 \xrightarrow{a_3} s_5 \xrightarrow{a_1} \dots & \quad [\text{Original episode}] \\
    s_2 \xrightarrow{a_4} s_1 \xrightarrow{a_2} s_2 \xrightarrow{a_3} s_5 \xrightarrow{a_1} \dots & \quad [\text{Episode starting from } (s_2, a_4)] \\
    s_1 \xrightarrow{a_2} s_2 \xrightarrow{a_3} s_5 \xrightarrow{a_1} \dots & \quad [\text{Episode starting from } (s_1, a_2)] \\
    s_2 \xrightarrow{a_3} s_5 \xrightarrow{a_1} \dots & \quad [\text{Episode starting from } (s_2, a_3)] \\
    s_5 \xrightarrow{a_1} \dots & \quad [\text{Episode starting from } (s_5, a_1)]
\end{aligned}
$$

### Data Efficiency Methods

* **First-visit**: Estimate using only the **first time** each state-action pair appears.
* **Every-visit**: Re-estimate every time a state-action pair appears.

## Generalized Policy Iteration (GPI)

Considerations on Update Timing:

* Wait until all episodes are collected, then average them for estimation.
* Or, start estimating after obtaining a single episode, re-estimating each time.

**Core Concepts of GPI**:

* GPI is not a specific algorithm, but a general term/framework.
* It embodies the process of continuously switching between **Policy Evaluation** and **Policy Improvement**.
* Many Model-based and Model-free algorithms fall within this framework.

## MC-$\epsilon$-Greedy

### Soft Policies

* Definition: A policy is called a "soft policy" if it selects every action with a non-zero probability.
* Role: Eliminates the need to ensure coverage by "generating massive episodes from every state-action pair," thereby removing the strong assumption of Exploring Starts.

### $\epsilon$-Greedy Policy

The formula is as follows:

$$
\pi(a|s) = 
\begin{cases} 
    1 - \dfrac{\epsilon}{|\mathcal{A}(s)|}(|\mathcal{A}(s)| - 1), & \text{for the greedy action, i.e., } a = a^* \\[15pt]
    \dfrac{\epsilon}{|\mathcal{A}(s)|}, & \text{for the other } |\mathcal{A}(s)| - 1 \text{ actions}
\end{cases}
$$

Where $\epsilon \in [0, 1]$ and $|\mathcal{A}(s)|$ is the total number of available actions in that state.

**Balancing Exploitation and Exploration**:

* When $\epsilon = 0$: Becomes Greedy (Full Exploitation).
* When $\epsilon = 1$: Becomes a Uniform Distribution (Full Exploration).

### Policy Improvement

The goal is to maximize the action-value function:

$$
\pi_{k+1}(s) = \arg \max_{\pi \in \Pi_{\varepsilon}} \sum_{a} \pi(a|s) q_{\pi_k}(s, a)
$$

The derived update rule is:

$$
\pi_{k+1}(a|s) = \begin{cases} 
1 - \frac{|\mathcal{A}(s)|-1}{|\mathcal{A}(s)|}\varepsilon, & a = a_k^* \\ 
\frac{1}{|\mathcal{A}(s)|}\varepsilon, & a \neq a_k^* \end{cases}
$$

**Conclusion**: By introducing $\epsilon$-Greedy, the "exploring starts" condition (the assumption allowing episodes to start from any state) is no longer required.
