---
title: "RL Study Notes: Actor-Critic Algorithm"
publishDate: 2026-02-22 21:40:00
description: "Overview of RL Actor-Critic frameworks, covering derivations and updates for QAC, A2C, importance sampling, and DPG."
tags: ["Reinforcement Learning", "Actor-Critic", "A2C", "DPG", "Study Notes"]
language: "English"
---

# Actor-critic


- **Actor** is responsible for the policy update, deciding which action to take in a given state.
- **Critic** is responsible for policy evaluation or value estimation, used to judge the quality of the policy selected by the Actor.

## QAC (Q-Actor-Critic)

$$
\begin{aligned}
&\text{Critic (value update):} \\
&w_{t+1} = w_t + \alpha_w [r_{t+1} + \gamma q(s_{t+1}, a_{t+1}, w_t) - q(s_t, a_t, w_t)] \nabla_w q(s_t, a_t, w_t) \\
&\text{Actor (policy update):} \\
&\theta_{t+1} = \theta_t + \alpha_\theta \nabla_\theta \ln \pi(a_t | s_t, \theta_t) q(s_t, a_t, w_{t+1})
\end{aligned}
$$

## A2C (Advantage Actor-Critic)

- Introduces a baseline to reduce variance.

$$
\begin{aligned}
\nabla_{\theta} J(\theta) &= \mathbb{E}_{S \sim \eta, A \sim \pi} \left[ \nabla_{\theta} \ln \pi(A|S, \theta_t) q_{\pi}(S, A) \right] \\
&= \mathbb{E}_{S \sim \eta, A \sim \pi} \left[ \nabla_{\theta} \ln \pi(A|S, \theta_t) (q_{\pi}(S, A) - b(S)) \right]
\end{aligned}
$$

To make the above equation hold, it must satisfy:

$$
\mathbb{E}_{S \sim \eta, A \sim \pi} \left[ \nabla_{\theta} \ln \pi(A|S, \theta_t) b(S) \right] = 0
$$

The specific derivation is as follows:

$$
\begin{aligned}
\mathbb{E}_{S \sim \eta, A \sim \pi} \left[ \nabla_{\theta} \ln \pi(A|S, \theta_t) b(S) \right] &= \sum_{s \in \mathcal{S}} \eta(s) \sum_{a \in \mathcal{A}} \pi(a|s, \theta_t) \nabla_{\theta} \ln \pi(a|s, \theta_t) b(s) \\
&= \sum_{s \in \mathcal{S}} \eta(s) \sum_{a \in \mathcal{A}} \nabla_{\theta} \pi(a|s, \theta_t) b(s) \\
&= \sum_{s \in \mathcal{S}} \eta(s) b(s) \nabla_{\theta} \sum_{a \in \mathcal{A}} \pi(a|s, \theta_t) \\
&= \sum_{s \in \mathcal{S}} \eta(s) b(s) \nabla_{\theta} 1 = 0
\end{aligned}
$$

- The baseline function is primarily used to control variance, so the goal is to find an optimal baseline function that minimizes variance. The optimal baseline is:

$$
b^*(s) = \frac{\mathbb{E}_{A \sim \pi} [\|\nabla_{\theta} \ln \pi(A|s, \theta_t)\|^2 q(s, A)]}{\mathbb{E}_{A \sim \pi} [\|\nabla_{\theta} \ln \pi(A|s, \theta_t)\|^2]}
$$

- Since this form is too complex to compute, in practice, the weight term $\mathbb{E}_{A \sim \pi} [\|\nabla_{\theta} \ln \pi(A|s, \theta_t)\|^2]$ is usually removed, approximating it as:

$$
b(s) = \mathbb{E}_{A \sim \pi} [q(s, A)] = v_{\pi}(s)
$$

When $b(s) = v_\pi(s)$:

* The **gradient ascent algorithm** is:
  
$$
\begin{aligned}
\theta_{t+1} &= \theta_t + \alpha \mathbb{E} \left[ \nabla_\theta \ln \pi(A|S, \theta_t) [q_\pi(S, A) - v_\pi(S)] \right] \\
&\doteq \theta_t + \alpha \mathbb{E} \left[ \nabla_\theta \ln \pi(A|S, \theta_t) \delta_\pi(S, A) \right]
\end{aligned}
$$
  
Where:
  
$$
\delta_\pi(S, A) \doteq q_\pi(S, A) - v_\pi(S)
$$

This term is called the **Advantage function**. According to the definition of $v_{\pi}(S)$, it is the expected value of all actions in state $S$. If the $q$ value of a certain action is greater than the average $v$, it indicates that the action possesses an "advantage".

* The **stochastic version** of this algorithm is:

$$
\begin{aligned}
\theta_{t+1} &= \theta_t + \alpha \nabla_\theta \ln \pi(a_t|s_t, \theta_t) [q_t(s_t, a_t) - v_t(s_t)] \\
&= \theta_t + \alpha \nabla_\theta \ln \pi(a_t|s_t, \theta_t) \delta_t(s_t, a_t)
\end{aligned}
$$

Furthermore, this algorithm can be rewritten as:

$$
\begin{aligned}
\theta_{t+1} &= \theta_t + \alpha \nabla_\theta \ln \pi(a_t|s_t, \theta_t) \delta_t(s_t, a_t) \\
&= \theta_t + \alpha \frac{\nabla_\theta \pi(a_t|s_t, \theta_t)}{\pi(a_t|s_t, \theta_t)} \delta_t(s_t, a_t) \\
&= \theta_t + \underbrace{\alpha \left( \frac{\delta_t(s_t, a_t)}{\pi(a_t|s_t, \theta_t)} \right)}_{\text{step size}} \nabla_\theta \pi(a_t|s_t, \theta_t)
\end{aligned}
$$

* The update step size is proportional to the **relative value $\delta_t$**, rather than the **absolute value $q_t$**, which is logically more reasonable.
* It still balances **exploration and exploitation** well.
  
Approximation via TD error:

$$
\delta_t = q_t(s_t, a_t) - v_t(s_t) \approx r_{t+1} + \gamma v_t(s_{t+1}) - v_t(s_t)
$$

* **This approximation is reasonable** because:

$$
\mathbb{E}[q_\pi(S, A) - v_\pi(S)|S = s_t, A = a_t] = \mathbb{E}[R_{t+1} + \gamma v_\pi(S_{t+1}) - v_\pi(S_t)|S = s_t, A = a_t]
$$

* **Advantage**: Only one neural network is needed to approximate $v_\pi(s)$, eliminating the need to maintain two separate networks to approximate $q_\pi(s, a)$ and $v_\pi(s)$.

## Importance Sampling and Off-policy

### Importance sampling technique

Note that:

$$
\mathbb{E}_{X \sim p_0}[X] = \sum_x p_0(x)x = \sum_x p_1(x) \underbrace{\frac{p_0(x)}{p_1(x)} x}_{f(x)} = \mathbb{E}_{X \sim p_1}[f(X)]
$$

* Therefore, we can estimate $\mathbb{E}_{X \sim p_0}[X]$ by estimating $\mathbb{E}_{X \sim p_1}[f(X)]$.
* The estimation method is as follows:
  
Let:
  
$$
\bar{f} \doteq \frac{1}{n} \sum_{i=1}^n f(x_i), \quad \text{where } x_i \sim p_1
$$

Then:

$$
\mathbb{E}_{X \sim p_1}[\bar{f}] = \mathbb{E}_{X \sim p_1}[f(X)]
$$

$$
\text{var}_{X \sim p_1}[\bar{f}] = \frac{1}{n} \text{var}_{X \sim p_1}[f(X)]
$$

So, $\bar{f}$ is a good approximation of $\mathbb{E}_{X \sim p_0}[X]$:

$$
\mathbb{E}_{X \sim p_0}[X] \approx \bar{f} = \frac{1}{n} \sum_{i=1}^n f(x_i) = \frac{1}{n} \sum_{i=1}^n \frac{p_0(x_i)}{p_1(x_i)} x_i
$$

* The ratio $\frac{p_0(x_i)}{p_1(x_i)}$ is called the **importance weight**.
* If $p_1(x_i) = p_0(x_i)$, the importance weight is 1, and $\bar{f}$ degenerates to the standard arithmetic mean.
* If $p_0(x_i) > p_1(x_i)$, it means sample $x_i$ appears more frequently in distribution $p_0$ than in $p_1$. An importance weight greater than 1 will enhance the proportion of this sample in the expectation calculation.
  
The objective function is defined as:

$$
J(\theta) = \sum_{s \in \mathcal{S}} d_\beta(s) v_\pi(s) = \mathbb{E}_{S \sim d_\beta} [v_\pi(S)]
$$

Its gradient is:

$$
\nabla_\theta J(\theta) = \mathbb{E}_{S \sim \rho, A \sim \beta} \left[ \frac{\pi(A | S, \theta)}{\beta(A | S)} \nabla_\theta \ln \pi(A | S, \theta) q_\pi(S, A) \right]
$$

The off-policy gradient also has invariance to the baseline function $b(s)$:

$$
\nabla_{\theta} J(\theta) = \mathbb{E}_{S \sim \rho, A \sim \beta} \left[ \frac{\pi(A|S, \theta)}{\beta(A|S)} \nabla_{\theta} \ln \pi(A|S, \theta) (q_{\pi}(S, A) - b(S)) \right]
$$

* To reduce the estimation variance, we similarly choose the baseline function $b(S) = v_{\pi}(S)$, obtaining:
  
$$
\nabla_{\theta} J(\theta) = \mathbb{E} \left[ \frac{\pi(A|S, \theta)}{\beta(A|S)} \nabla_{\theta} \ln \pi(A|S, \theta) (q_{\pi}(S, A) - v_{\pi}(S)) \right]
$$

The corresponding stochastic gradient ascent algorithm is:

$$
\theta_{t+1} = \theta_t + \alpha_{\theta} \frac{\pi(a_t|s_t, \theta_t)}{\beta(a_t|s_t)} \nabla_{\theta} \ln \pi(a_t|s_t, \theta_t) (q_t(s_t, a_t) - v_t(s_t))
$$

Similar to the on-policy case:

$$
q_t(s_t, a_t) - v_t(s_t) \approx r_{t+1} + \gamma v_t(s_{t+1}) - v_t(s_t) \doteq \delta_t(s_t, a_t)
$$

The final form of the algorithm becomes:

$$
\theta_{t+1} = \theta_t + \alpha_{\theta} \frac{\pi(a_t|s_t, \theta_t)}{\beta(a_t|s_t)} \nabla_{\theta} \ln \pi(a_t|s_t, \theta_t) \delta_t(s_t, a_t)
$$

Rewritten to highlight the step size relationship:

$$
\theta_{t+1} = \theta_t + \alpha_{\theta} \left( \frac{\delta_t(s_t, a_t)}{\beta(a_t|s_t)} \right) \nabla_{\theta} \pi(a_t|s_t, \theta_t)
$$

## Deterministic Actor-Critic (DPG)

Evolution of policy representation:

* Previously, the general policy was denoted as $\pi(a|s, \theta) \in [0, 1]$, which is usually stochastic.
* Now, a deterministic policy is introduced, denoted as:
  
$$
a = \mu(s, \theta) \doteq \mu(s)
$$

* $\mu$ is a direct mapping from the state space $\mathcal{S}$ to the action space $\mathcal{A}$.
* In practice, $\mu$ is often parameterized by a neural network, where the input is $s$, the output is directly the action $a$, and the parameters are $\theta$.

The gradient of the objective function is:

$$
\begin{aligned}
\nabla_{\theta} J(\theta) &= \sum_{s \in \mathcal{S}} \rho_{\mu}(s) \nabla_{\theta} \mu(s)\left.\left(\nabla_{a} q_{\mu}(s, a)\right)\right|_{a=\mu(s)} \\
&= \mathbb{E}_{S \sim \rho_{\mu}}\left[\left.\nabla_{\theta} \mu(S)\left(\nabla_{a} q_{\mu}(S, a)\right)\right|_{a=\mu(S)}\right]
\end{aligned}
$$

Based on the deterministic policy gradient, the gradient ascent algorithm to maximize $J(\theta)$ is:

$$
\theta_{t+1} = \theta_t + \alpha_\theta \mathbb{E}_{S \sim \rho_\mu} \left[ \nabla_\theta \mu(S) \left( \nabla_a q_\mu(S, a) \right) |_{a=\mu(S)} \right]
$$

The corresponding single-step stochastic gradient ascent update is:

$$
\theta_{t+1} = \theta_t + \alpha_\theta \nabla_\theta \mu(s_t) \left( \nabla_a q_\mu(s_t, a) \right) |_{a=\mu(s_t)}
$$

The overall architecture update logic is as follows:

**TD error**:

$$
\delta_t = r_{t+1} + \gamma q(s_{t+1}, \mu(s_{t+1}, \theta_t), w_t) - q(s_t, a_t, w_t)
$$

**Critic (value update)**:

$$
w_{t+1} = w_t + \alpha_w \delta_t \nabla_w q(s_t, a_t, w_t)
$$

**Actor (policy update)**:

$$
\theta_{t+1} = \theta_t + \alpha_\theta \nabla_\theta \mu(s_t, \theta_t) \left( \nabla_a q(s_t, a, w_{t+1}) \right) |_{a=\mu(s_t)}
$$

* This is an **off-policy** implementation. The data collection policy (behavior policy $\beta$) is usually different from the target policy $\mu$ being optimized.
* To ensure exploration, the behavior policy $\beta$ is typically set to the target policy plus noise, i.e., $\beta = \mu + \text{noise}$.
* **Regarding the function approximation choice for $q(s, a, w)$**:
  * **Linear function**: $q(s, a, w) = \phi^T(s, a)w$, where $\phi(s, a)$ is a manually designed feature vector.
  * **Neural network**: When using deep neural networks to approximate value and policy, it evolves into the **Deep Deterministic Policy Gradient (DDPG)** algorithm.