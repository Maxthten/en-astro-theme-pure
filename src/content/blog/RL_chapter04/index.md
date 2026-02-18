---
title: "RL Notes: Value Iteration and Policy Iteration"
publishDate: 2026-02-18 10:30:00
description: "Analyzes Value & Policy Iteration, showing how Truncated PI unifies them via evaluation steps."
tags: ["Reinforcement Learning", "Value Iteration", "Policy Iteration", "Truncated Policy Iteration", "Study Notes"]
language: "English"
---

# Value Iteration and Policy Iteration

## Value Iteration

Value Iteration approximates the optimal value function $v_*$ by iteratively updating the value function $v_k$. The core iteration formula is:

$$
v_{k+1} = f(v_k) = \max_{\pi} (r_{\pi} + \gamma P_{\pi} v_k), \quad k = 1, 2, 3 \dots
$$

This process can be decomposed into two steps:

1. **Policy Update**:
   
   $$
   \pi_{k+1} = \arg\max_{\pi} (r_{\pi} + \gamma P_{\pi} v_k)
   $$

2. **Value Update**:
   
   $$
   v_{k+1} = r_{\pi_{k+1}} + \gamma P_{\pi_{k+1}} v_k
   $$

*Note: Here, $v_k$ represents the estimated value vector at the $k$-th iteration, not the final state value.*

### 1. Policy Update

$$
\pi_{k+1} = \arg\max_{\pi} (r_{\pi} + \gamma P_{\pi} v_k)
$$

Its elementwise form is:

$$
\pi_{k+1}(s) = \arg\max_{\pi} \sum_{a} \pi(a|s) \underbrace{\left( \sum_{r} p(r|s,a)r + \gamma \sum_{s'} p(s'|s,a)v_k(s') \right)}_{q_k(s,a)}, \quad s \in \mathcal{S}
$$

The resulting $\pi_{k+1}$ is a greedy policy, which selects the action $a_k^*(s)$ that maximizes $q_k(s,a)$ in state $s$:

$$
a_k^*(s) = \arg\max_{a} q_k(a, s)
$$

$$
\pi_{k+1}(a|s) = \begin{cases} 1, & a = a_k^*(s) \\ 0, & a \neq a_k^*(s) \end{cases}
$$

### 2. Value Update

$$
v_{k+1} = r_{\pi_{k+1}} + \gamma P_{\pi_{k+1}} v_k
$$

Its elementwise form is:

$$
v_{k+1}(s) = \sum_{a} \pi_{k+1}(a|s) \underbrace{ \left( \sum_{r} p(r|s,a)r + \gamma \sum_{s'} p(s'|s,a)v_k(s') \right) }_{q_k(s,a)}, \quad s \in \mathcal{S}
$$

Since $\pi_{k+1}$ is a greedy policy, the above equation is equivalent to:

$$
v_{k+1}(s) = \max_{a} q_k(a, s)
$$

---

## Policy Iteration

Policy Iteration consists of the following steps:

1. **Initialization**: Given a random initial policy $\pi_0$.

2. **Policy Evaluation (PE)**: Calculate the state value $v_{\pi_k}$ of the current policy.
   
   $$
   v_{\pi_k} = r_{\pi_k} + \gamma P_{\pi_k} v_{\pi_k}
   $$

3. **Policy Improvement (PI)**: Generate a better policy based on the current value.
   
   $$
   \pi_{k+1} = \arg \max_{\pi} (r_\pi + \gamma P_\pi v_{\pi_k})
   $$

### 1. Policy Evaluation

Solving for $v_{\pi_k}$ usually employs an iterative method:

* **Matrix-vector form**:
  
  $$
  v_{\pi_k}^{(j+1)} = r_{\pi_k} + \gamma P_{\pi_k} v_{\pi_k}^{(j)}, \quad j = 0, 1, 2, \dots
  $$

* **Elementwise form**:
  
  $$
  v_{\pi_k}^{(j+1)}(s) = \sum_{a} \pi_k(a|s) \left( \sum_{r} p(r|s, a)r + \gamma \sum_{s'} p(s'|s, a)v_{\pi_k}^{(j)}(s') \right), \quad s \in \mathcal{S}
  $$

Stopping condition: Stop iterating when $j \to \infty$ or when $\| v_{\pi_k}^{(j+1)} - v_{\pi_k}^{(j)} \|$ is sufficiently small.

### 2. Policy Improvement

* **Matrix-vector form**:
  
  $$
  \pi_{k+1} = \arg \max_{\pi} (r_\pi + \gamma P_\pi v_{\pi_k})
  $$

* **Elementwise form**:
  
  $$
  \pi_{k+1}(s) = \arg \max_{\pi} \sum_{a} \pi(a|s) \underbrace{\left( \sum_{r} p(r|s, a)r + \gamma \sum_{s'} p(s'|s, a)v_{\pi_k}(s') \right)}_{q_{\pi_k}(s, a)}, \quad s \in \mathcal{S}
  $$
  
  

Let $a_k^*(s) = \arg \max_{a} q_{\pi_k}(a, s)$, the updated policy is a deterministic greedy policy:

$$
\pi_{k+1}(a|s) = \begin{cases} 1, & a = a_k^*(s) \\ 0, & a \neq a_k^*(s) \end{cases}
$$

---

## Truncated Policy Iteration

We can unify Value Iteration and Policy Iteration from the perspective of "the number of policy evaluation steps":

$$
\begin{alignat*}{2}
\text{Policy Iteration: } & \pi_0 \xrightarrow{PE} v_{\pi_0} \xrightarrow{PI} \pi_1 \xrightarrow{PE} v_{\pi_1} \xrightarrow{PI} \pi_2 \dots \\
\text{Value Iteration: }  & \phantom{\pi_0 \xrightarrow{PE}} v_0 \xrightarrow{PU} \pi_1' \xrightarrow{VU} v_1 \xrightarrow{PU} \pi_2' \dots
\end{alignat*}
$$

* **Policy Iteration**: $\text{P} \to \text{vvvv...} \to \text{P} \to \text{vvvv...}$ (Evaluation to convergence)
* **Value Iteration**: $\text{P} \to \text{v} \to \text{P} \to \text{v} \to \text{P} \to \text{v}$ (Evaluation for only one step)

**Algorithm comparison under a unified perspective:**

$$
\begin{array}{rll}
    & v_{\pi_1}^{(0)} = v_0 & \text{Initial value} \\
    \text{Value Iteration} \leftarrow v_1 \longleftarrow & v_{\pi_1}^{(1)} = r_{\pi_1} + \gamma P_{\pi_1} v_{\pi_1}^{(0)} & \text{(Iterate only 1 time)} \\
    & v_{\pi_1}^{(2)} = r_{\pi_1} + \gamma P_{\pi_1} v_{\pi_1}^{(1)} & \\
    & \quad \vdots & \\
    \text{Truncated Policy Iteration} \leftarrow \bar{v}_1 \longleftarrow & v_{\pi_1}^{(j)} = r_{\pi_1} + \gamma P_{\pi_1} v_{\pi_1}^{(j-1)} & \text{(Iterate j times)} \\
    & \quad \vdots & \\
    \text{Policy Iteration} \leftarrow v_{\pi_1} \longleftarrow & v_{\pi_1}^{(\infty)} = r_{\pi_1} + \gamma P_{\pi_1} v_{\pi_1}^{(\infty)} & \text{(Iterate to convergence)}
\end{array}
$$

**Note**: Standard Policy Iteration requires an exact solution for $v_{\pi_k}$ at each step (i.e., $j \to \infty$), which is often infeasible or inefficient in practical calculations. Therefore, **Truncated Policy Iteration** is commonly used in practice, which limits the number of evaluation steps $j$.
