---
title: "RL Study Notes: Value Function Approximation"
publishDate: 2026-02-21 21:15:00
description: "Summary of value function approximation in RL, covering linear/non-linear forms, state distributions, gradient methods, DQN, and experience replay."
tags: ["Reinforcement Learning", "Value Function Approximation", "DQN", "Study Notes"]
language: "English"
---

# Value Function Approximation

## Linear Function Form

$$
\hat{v}(s, w) = as + b = \underbrace{[s, 1]}_{\phi^T(s)} \underbrace{\begin{bmatrix} a \\ b \end{bmatrix}}_{w} = \phi^T(s)w
$$

Where:

- $w$ is the parameter vector.
- $\phi(s)$ is the feature vector of state $s$.
- $\hat{v}(s, w)$ is linear with respect to $w$.

## Non-linear Function Form

$$
\hat{v}(s, w) = as^2 + bs + c = \underbrace{[s^2, s, 1]}_{\phi^T(s)} \underbrace{\begin{bmatrix} a \\ b \\ c \end{bmatrix}}_{w} = \phi^T(s)w
$$

In this case:

- The dimensions of $w$ and $\phi(s)$ increase, potentially making the numerical fitting more accurate.
- Although $\hat{v}(s, w)$ is non-linear with respect to state $s$, it remains linear with respect to parameter $w$. The non-linear features are encapsulated in the mapping $\phi(s)$.

## State Value Estimation

Objective Function:

$$
J(w) = \mathbb{E}[(v_\pi(S) - \hat{v}(S, w))^2]
$$

- The core objective is to find the optimal parameters $w$ to minimize this objective function.
- $S$ is a random variable. Its probability distribution mainly considers the following two types:

### Uniform Distribution

$$
J(w) = \frac{1}{|\mathcal{S}|} \sum_{s \in \mathcal{S}} (v_\pi(s) - \hat{v}(s, w))^2
$$

- The uniform distribution treats all states equally. However, in actual reinforcement learning, some states are visited more frequently and are more critical, so this distribution is often unsuitable.

### Stationary Distribution

The stationary distribution describes the long-run behavior of a Markov process. Here, $\{d_{\pi}(s)\}_{s \in \mathcal{S}}$ represents the set of state distributions, satisfying $d_{\pi}(s) \geq 0$ and $\sum_{s \in \mathcal{S}} d_{\pi}(s) = 1$.

$$
J(w) = \sum_{s \in \mathcal{S}} d_{\pi}(s)(v_{\pi}(s) - \hat{v}(s, w))^2
$$

- $d_{\pi}(s)$ represents the stationary probability of being in a specific state under policy $\pi$. Using the stationary distribution allows for smaller fitting errors on frequently visited states.
- The stationary distribution satisfies the following formula:

$$
d_{\pi}^T = d_{\pi}^T P_{\pi}
$$

Where $P_{\pi}$ is the state transition matrix in the Bellman equation.

## Optimization Methods

Update parameters using gradient descent:

$$
w_{k+1} = w_k - \alpha_k \nabla_w J(w_k)
$$

The derivation of the true gradient is as follows:

$$
\begin{aligned}
\nabla_w J(w) &= \nabla_w \mathbb{E}[(v_\pi(S) - \hat{v}(S, w))^2] \\
&= \mathbb{E}[\nabla_w (v_\pi(S) - \hat{v}(S, w))^2] \\
&= 2\mathbb{E}[(v_\pi(S) - \hat{v}(S, w))(-\nabla_w \hat{v}(S, w))] \\
&= -2\mathbb{E}[(v_\pi(S) - \hat{v}(S, w))\nabla_w \hat{v}(S, w)]
\end{aligned}
$$

In practice, Stochastic Gradient Descent (SGD) is commonly used:

$$
w_{t+1} = w_t + \alpha_t (v_\pi(s_t) - \hat{v}(s_t, w_t)) \nabla_w \hat{v}(s_t, w_t)
$$

Where $s_t$ is a sample of $S$. For brevity, the constant $2$ is absorbed into the learning rate $\alpha_t$. Since the true $v_{\pi}(s_t)$ is unknown, we need to replace it with an estimate:

- **Monte Carlo (MC) based**: Use the discounted return $g_t$ in an episode to approximate $v_{\pi}(s_t)$.

$$
w_{t+1} = w_t + \alpha_t (g_t - \hat{v}(s_t, w_t)) \nabla_w \hat{v}(s_t, w_t)
$$

- **Temporal Difference (TD) based**: The target value $r_{t+1}+\gamma\hat{v}(s_{t+1},w_t)$ is treated as an approximation of $v_{\pi}(s_t)$.

$$
w_{t+1} = w_t + \alpha_t [r_{t+1} + \gamma \hat{v}(s_{t+1}, w_t) - \hat{v}(s_t, w_t)] \nabla_w \hat{v}(s_t, w_t)
$$

## TD-Linear Algorithm

In the linear case of $\hat{v}(s, w) = \phi^T(s)w$, the gradient is:

$$
\nabla_w \hat{v}(s, w) = \phi(s)
$$

Substituting the gradient into the TD algorithm:

$$
w_{t+1} = w_t + \alpha_t [r_{t+1} + \gamma \phi^T(s_{t+1}) w_t - \phi^T(s_t) w_t] \phi(s_t)
$$

This is the TD learning algorithm with linear function approximation, briefly referred to as **TD-Linear**.

### Derivative Analysis of Linear Approximation

In RL linear approximation, $\hat{v}(s, w)$ is a scalar (predicted state value), and $w$ is a vector (weight parameters).

1. **Deconstructing the Linear Expression**
   For column vectors $\phi(s) = [\phi_1, \dots, \phi_n]^T$ and $w = [w_1, \dots, w_n]^T$, the inner product is:
   
   $$
   \hat{v}(s, w) = \sum_{i=1}^{n} \phi_i w_i
   $$

2. **Deriving with Respect to a Vector**
   The essence of $\nabla_w \hat{v}(s, w)$ is taking the partial derivative of the scalar function with respect to each component of vector $w$:
   
   $$
   \frac{\partial}{\partial w_i} (\phi_1 w_1 + \dots + \phi_n w_n) = \phi_i
   $$
   
   Putting it together, we get $\nabla_w \hat{v}(s, w) = \phi(s)$.

### Tabular Representation

The tabular method is a special case of linear function approximation.
Assume the feature vector of state $s$ is a One-hot vector:

$$
\phi(s) = e_s \in \mathbb{R}^{|\mathcal{S}|}
$$

At this time:

$$
\hat{v}(s, w) = e_s^T w = w(s)
$$

That is, $w(s)$ extracts the $s$-th component of vector $w$ corresponding to state $s$.

## Action Value Function Approximation

### Sarsa with Function Approximation

$$
w_{t+1} = w_t + \alpha_t \left[ r_{t+1} + \gamma \hat{q}(s_{t+1}, a_{t+1}, w_t) - \hat{q}(s_t, a_t, w_t) \right] \nabla_w \hat{q}(s_t, a_t, w_t)
$$

### Q-learning with Function Approximation

$$
w_{t+1} = w_t + \alpha_t \left[ r_{t+1} + \gamma \max_{a \in \mathcal{A}(s_{t+1})} \hat{q}(s_{t+1}, a, w_t) - \hat{q}(s_t, a_t, w_t) \right] \nabla_w \hat{q}(s_t, a_t, w_t)
$$

## Deep Q-Network (DQN)

DQN uses neural networks to approximate the non-linear Q function.

**Loss Function**:

$$
J(w) = \mathbb{E} \left[ \left( R + \gamma \max_{a \in \mathcal{A}(S')} \hat{q}(S', a, w) - \hat{q}(S, A, w) \right)^2 \right]
$$

This is essentially minimizing the Bellman Optimality Error. Define the target value $y$ as:

$$
y \doteq R + \gamma \max_{a \in \mathcal{A}(S')} \hat{q}(S', a, w)
$$

To ensure training stability and prevent the target value from constantly shifting with network updates, DQN introduces a dual-network architecture:

- **Main Network**: $\hat{q}(S, A, w)$, responsible for current action evaluation and real-time parameter updates.
- **Target Network**: $\hat{q}(S', A, w_T)$, providing a stable target value $y$.

With the target network introduced, the loss function becomes:

$$
J(w) = \mathbb{E} \left[ \left( R + \gamma \max_{a \in \mathcal{A}(S')} \hat{q}(S', a, w_T) - \hat{q}(S, A, w) \right)^2 \right]
$$

During computation, assuming $w_T$ is a constant (i.e., not involved in gradient calculation), gradient descent only updates $w$:

$$
\nabla_w J(w) = -2\mathbb{E} \left[ \left( R + \gamma \max_{a \in \mathcal{A}(S')} \hat{q}(S', a, w_T) - \hat{q}(S, A, w) \right) \nabla_w \hat{q}(S, A, w) \right]
$$

*Note: The parameters $w$ of the main network are periodically copied to the target network $w_T$.*

### Experience Replay

- **Motivation**: Sequential data collected in reinforcement learning has strong correlations. Using it directly for training can easily lead to network instability.
- **Mechanism**: Store the interaction data generated by the agent and the environment as tuples $(s, a, r, s')$ into a Replay Buffer $\mathcal{B}$.
- **Sampling**: During training, extract a batch of random samples (Mini-batch) from the buffer. This extraction process usually follows a uniform distribution, thereby breaking the temporal correlations between data and significantly improving data utilization efficiency.
