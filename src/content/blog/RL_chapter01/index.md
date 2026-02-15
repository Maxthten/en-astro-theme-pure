---
title: "RL Study Notes: Basic Concepts"
publishDate: 2026-02-15 19:45:00
description: "A summary of core definitions in Reinforcement Learning (State, Action, Reward) and the elements of Markov Decision Processes (MDP)."
tags: ["Reinforcement Learning", "MDP", "Study Notes", "Math Basics"]
language: "en"
---

# Basic Concepts

## I. Core Definitions

1. **State**
   The status of the Agent relative to the environment. In a grid world, this is typically considered the Agent's coordinate location. For example, $S_1$ can be represented as a vector:
   $$S_1 = \begin{pmatrix} x \\ y \end{pmatrix}$$

2. **State space**
   The set of all possible states, denoted as $\mathcal{S}$. For example: $S=\left\{s_{i}\right\}_{i=1}^{9}$. It is essentially just a Set.

3. **Action**
   The moves an Agent can take for every State. For example, in a grid world, there might be five: up, down, left, right, and stay.

4. **Action space**
   The set of all possible actions for a specific state $s_i$, denoted as $\mathcal{A}(s_{i}) = \left\{a_{i}\right\}_{i=1}^{5}$.
   *Note: Action often depends on the State, meaning $\mathcal{A}$ is a function of $s$.*

5. **State transition**
   The process where the Agent transfers from one state to another after taking an action, denoted as $S_{1}\stackrel{a_2}{\longrightarrow}S_{2}$.
   This defines the mechanism of interaction with the environment. In a virtual world, this can be defined arbitrarily; in the real world, it must obey objective physical laws.

6. **State transition probability**
   Uses probability to describe the uncertainty of state transitions. For example, if we choose $a_2$ at $S_1$, the probability of moving to $S_2$ is:
   
   $$
   p(s'|s, a) \Rightarrow \begin{cases} p(s_{2}|s_{1},a_{2})=1 \\ p(s_{i}|s_{1},a_{2})=0, & \forall i\neq2 \end{cases}
   $$
   
   The example above represents a deterministic environment, but it can also be stochastic (random).

7. **Policy**
   The rule or function $\pi$ that describes what action the Agent should take in a given State.
   For example, a **Deterministic Policy**:
   
   $$
   \pi(a|s) \Rightarrow \begin{cases} \pi(a_{2}|s_{1})=1 \\ \pi(a_{i}|s_{1})=0, & \forall i\neq2 \end{cases}
   $$
   
   **Stochastic Policies** follow the same logic, where $\pi$ represents the probability of selecting an action.

8. **Reward**
   A **scalar real number** received after the Agent takes a specific action.
   
   * Positive numbers typically represent rewards (encouraging behavior);
   * Negative numbers typically represent punishment (suppressing behavior).
   
   Reward is a key method of **Human-Machine Interface**, used to guide the Agent to exhibit the behavior we expect. Mathematical expression:
   
   $$
   p(r=-1|s_{1},a_{1})=1 \quad \text{and} \quad p(r\neq-1|s_{1},a_{1})=0
   $$

9. **Trajectory**
   A complete State-Action-Reward chain. Specifically: taking an Action in a State, receiving a Reward, and transferring to the next State, repeated in a loop.
   
   $$
   S_{1} \xrightarrow[r=0]{a_3} S_{4} \xrightarrow[r=-1]{a_3} S_{7} \xrightarrow[r=0]{a_2} S_{8} \xrightarrow[r=+1]{a_2} S_{9}
   $$

10. **Return**
    The sum of all Rewards in a Trajectory. Different Policies will lead to different Returns.

11. **Discounted Return**
    For a Trajectory that runs infinitely, a simple sum would result in an infinite Return (divergence). We introduce a discount factor $\gamma \in [0,1)$:
    
    $$
    \text{Return} = 0+0+1+1+\dots = \infty \quad (\text{Divergent})
    $$
    
    With $\gamma$:
    
    $$
    G_t = R_{t+1} + \gamma R_{t+2} + \gamma^2 R_{t+3} + \dots = \sum_{k=0}^{\infty} \gamma^k R_{t+k+1}
    $$
    
    Example:
    
    $$
    \text{Discounted Return} = \gamma^{3}(1+\gamma+\gamma^{2}+\ldots) = \frac{\gamma^{3}}{1-\gamma} \quad (\text{Convergent})
    $$
    
    * **Role of $\gamma$**: Determines the Agent's "vision." A smaller $\gamma$ makes the Agent "near-sighted" (focusing on immediate rewards), while a larger $\gamma$ makes it "far-sighted" (focusing on long-term benefits).

12. **Episode**
    When interacting with the environment following a Policy, if the Agent stops at a **Terminal State**, the resulting trajectory is called an Episode (or Trial).
    
    * **Episodic Tasks**: Tasks with terminal states (finite steps).
    * **Continuing Tasks**: Tasks without terminal states (infinite steps).

---

## II. MDP (Markov Decision Process) Elements

### 1. Sets

* **State**: The set of States $\mathcal{S}$
* **Action**: The set of Actions $\mathcal{A}(s)$, where $s \in \mathcal{S}$
* **Reward**: The set of Rewards $\mathcal{R}(s,a)$

### 2. Probability Distribution (Dynamics)

* **State transition probability**: $p(s'|s,a)$
* **Reward probability**: $p(r|s,a)$

### 3. Policy

* The Agent's decision mechanism: $\pi(a|s)$

### 4. MDP Property

**Memoryless (Markov Property)**:
The probability of the next state and reward depends *only* on the current state and action, and is independent of all prior history.

$$
p(s_{t+1}, r_{t+1} | s_t, a_t, s_{t-1}, \dots) = p(s_{t+1}, r_{t+1} | s_t, a_t)
$$

### 5. MDP vs Markov Process

* **Markov Process**: Contains only States and Transition Probabilities. The observer can only passively accept the environment's evolution based on probability and cannot intervene.
* **MDP (Markov Decision Process)**: Adds **Decision (Action)**.
    State transitions depend not only on the current state but also on the **Action** taken. The Agent can actively influence the outcome probabilities by choosing different actions, rather than just passively accepting a fixed distribution.
