---
title: "RL Study Notes: Bellman Optimality Equation"
publishDate: 2026-02-17 17:00:00
description: "Derives Bellman Optimality and fixed-point properties. Analyzes Value Iteration (contraction mapping) and how models/rewards determine the optimal policy."
tags: ["Reinforcement Learning", "Bellman Optimality", "Value Iteration", "Study Notes"]
language: "English"
---

# Bellman Optimality Equation

## Definition

For all states $s \in \mathcal{S}$, if the state value function $v_{\pi^*}(s)$ of policy $\pi^*$ is not less than the state value function $v_{\pi}(s)$ of any other policy $\pi$, that is:

$$
v_{\pi^*}(s) \geq v_{\pi}(s), \quad \forall s \in \mathcal{S}, \forall \pi
$$

Then policy $\pi^*$ is called the **optimal policy**. The state value corresponding to the optimal policy is called the **optimal state value function**, denoted as $v^*(s)$.

## Derivation of the Optimality Equation

The optimal state value function $v^*(s)$ satisfies the Bellman Optimality Equation. The core idea is: the optimal value equals the expected return obtained by taking the **optimal action** in the current state.

### Scalar Form

$$
\begin{aligned}
v^*(s) &= \max_{a \in \mathcal{A}} q^*(s, a) \\
&= \max_{a \in \mathcal{A}} \left( \sum_{r \in \mathcal{R}} p(r|s,a)r + \gamma \sum_{s' \in \mathcal{S}} p(s'|s,a)v^*(s') \right)
\end{aligned}
$$

If written as a maximization over policy $\pi$, we have:

$$
v^*(s) = \max_{\pi} \sum_{a \in \mathcal{A}} \pi(a|s) q^*(s, a)
$$

Since a weighted average cannot exceed the maximum value:

$$
\sum_{a \in \mathcal{A}} \pi(a|s) q^*(s, a) \leq \max_{a \in \mathcal{A}} q^*(s, a)
$$

The condition for equality is that the policy $\pi$ assigns probability completely to the action that maximizes $q^*(s,a)$. This implies that the optimal policy $\pi^*$ is **deterministic**:

$$
\pi^*(a|s) = 
\begin{cases} 
1, & a = \arg\max_{a' \in \mathcal{A}} q^*(s, a') \\ 
0, & \text{otherwise}
\end{cases}
$$

### Vector Form

We treat the process of solving for $v^*$ as an operator operation. Defining the optimal Bellman operator $\mathcal{T}^*$, the Bellman Optimality Equation is a fixed-point equation:

$$
v^* = \mathcal{T}^*(v^*)
$$

Specifically expanded as:

$$
v^* = \max_{\pi} (r_{\pi} + \gamma P_{\pi} v^*)
$$

Where:

* $r_{\pi}$ is the average immediate reward vector under policy $\pi$, defined as $[r_{\pi}]_s = \sum_{a} \pi(a|s) \sum_{r} p(r|s,a)r$
* $P_{\pi}$ is the state transition matrix under policy $\pi$, defined as $[P_{\pi}]_{s,s'} = \sum_{a} \pi(a|s) p(s'|s,a)$

## Contraction Mapping and Fixed Points

The Bellman optimal operator $\mathcal{T}^*$ satisfies the **Contraction Mapping Theorem** when $\gamma \in [0, 1)$. This implies:

1. **Existence**: There exists a unique fixed point $v^*$ satisfying $v^* = \mathcal{T}^*(v^*)$.
2. **Convergence**: For any initial value $v_0$, the iterative sequence $v_{k+1} = \mathcal{T}^*(v_k)$ inevitably converges to $v^*$.
   * That is, $\lim_{k \to \infty} v_k = v^*$.
   * The convergence speed is geometric (exponential convergence), controlled by the discount factor $\gamma$.

### The Essence of Value Iteration

The Value Iteration algorithm utilizes the aforementioned fixed-point property with the iterative formula:

$$
v_{k+1} = \max_{\pi} (r_{\pi} + \gamma P_{\pi} v_k)
$$

This step actually contains two implicit processes:

1. **Implicit Policy Improvement**:
   Based on the current value estimate $v_k$, find a greedy policy, i.e., select the action that currently appears to have the highest $q$ value.
   
   $$
   \pi_{greedy} = \arg\max_{\pi} (r_{\pi} + \gamma P_{\pi} v_k)
   $$

2. **Value Update (Policy Evaluation)**:
   Assuming the greedy action is taken, calculate its one-step expected return as the new value estimate $v_{k+1}$.

**Summary**: Value Iteration essentially "seizes" the currently best action in every round, calculates its value, and then "seizes" the best action again in the next round based on the new value.

## Determinants of the Optimal Policy

The optimal policy $\pi^*$ is determined by the following formula:

$$
\pi^*(s) = \arg\max_{a} \left( \sum_{r} p(r|s,a)r + \gamma \sum_{s'} p(s'|s,a)v^*(s') \right)
$$

### Key Factors

1. **System Dynamics**: $p(s'|s, a)$ and $p(r|s, a)$. These are the physical laws of the environment and are generally immutable.
2. **Discount Factor** $\gamma$:
   * $\gamma \to 0$: The agent becomes "myopic," focusing only on immediate rewards.
   * $\gamma \to 1$: The agent becomes "far-sighted," valuing long-term cumulative returns.
3. **Reward Function** $r$:
   * The **relative numerical values** of the reward are more important than the absolute values.

### Affine Transformation of the Reward Function

If a linear transformation is applied to the reward function:

$$
r'(s, a, s') = \alpha \cdot r(s, a, s') + \beta
$$

Where $\alpha > 0$ and $\beta$ is a constant.

* **Impact on Value Function**: The new value function $v'$ has a linear relationship with the original value function $v$.

$$
v'(s) = \alpha v(s) + \frac{\beta}{1-\gamma}
$$

* **Impact on Policy**: The optimal policy **remains unchanged**.

$$
\arg\max_a q'(s,a) = \arg\max_a \left( \alpha q(s,a) + \frac{\beta}{1-\gamma} \right) = \arg\max_a q(s,a)
$$

This indicates that as long as the partial order relations and relative proportions between rewards are preserved, the specific numerical magnitude does not alter the optimal behavior pattern.
