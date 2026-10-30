# Chapter 23 — Reinforcement Learning

**Course:** CAI 4002 — Introduction to Artificial Intelligence (USF Fall 2026)

**Sections:** 23.1 Learning from Rewards (pp. 840–842) · 23.2 Passive Reinforcement Learning (pp. 842–848) · 23.3 Active Reinforcement Learning, introduction, §23.3.1 and §23.3.3 (pp. 848–850, 852–854), selections.

**Class notes:** *Day 10 — MDP & Value Iteration*, slides 59–82. See [Chapter 16](chapter-16-making-complex-decisions.md) for MDPs, Bellman equations, and planning.

## Day 10 — Learning from Experience

### 1. Planning versus Reinforcement Learning

> **Definition (Reinforcement learning).** An agent learns from interaction with an environment and reward feedback, aiming to maximize expected cumulative reward.

In the lecture's planning setting, the transition and reward models are given. In its reinforcement-learning setting, they are initially unknown, so the agent takes actions and observes transitions. Model-based RL can also use a model that is already known; what makes it model-based is **using a model**, not necessarily learning one.

Each experience can be recorded as $(s,a,r,s')$: current state, action, observed reward, and next state. The reward is feedback about the outcome, not necessarily a label telling the agent the correct action.

| Distinction | Meaning |
|---|---|
| Passive learning | follow a fixed policy and estimate its values |
| Active learning | choose actions and learn which behavior is good |
| Model-based | use a transition model, possibly learned from experience |
| Model-free | learn values or a policy without learning a transition model |

Passive versus active describes the **learning task**; model-based versus model-free describes the **method**. They are different distinctions.

### 2. Model-Based Learning

Estimate each transition probability from observed frequencies, then use the learned model for planning or policy evaluation. Let $N(s,a,s')$ count transitions to $s'$ and let $N(s,a)$ count all observed transitions from $(s,a)$:

$$
\widehat{T}(s,a,s')=\frac{N(s,a,s')}{N(s,a)},\qquad N(s,a)>0.
$$

Estimate transition rewards from observations as well; if rewards are stochastic, their sample average estimates the expected reward. An unvisited state–action pair has no empirical frequency estimate yet. A model learned from limited experience can be wrong, so solving it exactly does not guarantee an optimal policy in the real environment.

For passive learning, evaluate the given policy using the learned model. For active learning, plan over available actions using the learned model and continue gathering experience.

### 3. Direct Policy Evaluation

> **Definition (Direct evaluation).** Follow a fixed policy, observe the discounted return after visits to a state, and average those observed returns to estimate the state's policy value.

A return is the reward **from that visit onward**, not the entire episode's reward indiscriminately. With completed episodes and enough representative visits, sample averages estimate expected return. Direct evaluation does not need a transition model, but it waits for observed returns and does not directly exploit the value relationships between neighboring states.

Different sample averages can arise for two states that both lead to the same successor: their observed episodes may contain different outcomes. A finite sample estimate is not necessarily the true expected value.

### 4. Temporal-Difference Value Learning

> **Definition (Temporal-difference learning).** Update a value estimate using a sampled reward and the current estimate of a successor's value.

For a transition $(s,a,r,s')$ generated while following a fixed policy:

$$
\begin{aligned}
\text{TD target} &= r+\gamma V(s'),\\
\text{TD error} &= r+\gamma V(s')-V(s),\\
V(s) &\leftarrow V(s)+\alpha\left[r+\gamma V(s')-V(s)\right].
\end{aligned}
$$

Here $\alpha$ is the learning rate: the update moves the old estimate partway toward the target. For a terminal successor, use zero continuation value under the transition-reward convention. TD can update after one transition rather than waiting for the episode's complete return. It **bootstraps** because the target uses another current estimate, which can itself be inaccurate.

A fixed positive learning rate gives greater weight to more recent observations and may leave values fluctuating. For standard tabular convergence, suitably decreasing learning rates and sufficient visits are needed; merely making the rate small is not enough.

### 5. Q-Values and Q-Learning

A state value alone does not tell a model-free agent which action to choose: comparing actions by one-step look-ahead requires transition and reward information. A Q-value scores a **state–action pair**, so actions can be compared directly.

> **Definition (Q-learning).** A model-free, off-policy method that updates action values from sampled transitions toward the immediate reward plus the best estimated successor action value.

$$
Q(s,a)\leftarrow Q(s,a)+\alpha\left[r+\gamma\max_{a'\in A(s')}Q(s',a')-Q(s,a)\right].
$$

For a terminal successor, the future-value term is zero. The maximum is part of the **learning target**: it does not require the agent to have actually chosen that best successor action. Thus the behavior used to collect samples may differ from the greedy policy being learned; this is **off-policy** learning.

The model-based counterpart averages over all successor states using known or estimated transition probabilities. Q-learning uses an observed successor instead; frequent outcomes influence updates more often through sampling.

For tabular Q-learning in a finite discounted MDP with bounded rewards, standard convergence conditions include visiting every relevant state–action pair infinitely often and suitable per-pair learning rates: their sums diverge while their squared sums converge. These are asymptotic conditions, not a guarantee that a short training run or arbitrary exploration schedule has found the optimal policy.

### 6. Exploration and Exploitation

> **Exploration** tries actions to gain information. **Exploitation** chooses an action currently estimated to be best.

An **epsilon-greedy** behavior policy chooses a random available action with probability $\epsilon$ and a maximizing action otherwise. Random choice can also select a maximizing action. Reducing $\epsilon$ can reduce exploration later, but reducing it too quickly can prevent adequate coverage; not every decreasing schedule satisfies the convergence conditions.

| Parameter | Controls | Not the same as |
|---|---|---|
| $\epsilon$ | how often epsilon-greedy behavior explores | the update size or reward discount |
| $\gamma$ | weight given to future rewards | the exploration probability |
| $\alpha$ | how strongly a new target changes an estimate | the discount factor |

## Quick Reference

| Term | Meaning |
|---|---|
| Passive RL | learn values while following a fixed policy |
| Active RL | learn while selecting actions |
| Model-based RL | use a model of transitions for learning or decisions |
| Model-free RL | learn without a transition model |
| Direct evaluation | average observed returns after state visits |
| TD target | sampled immediate reward plus discounted estimated successor value |
| Bootstrapping | updating an estimate using another estimate |
| Q-learning | model-free, off-policy action-value learning using a greedy successor target |
| Epsilon-greedy | sometimes act randomly; otherwise choose a current best action |
| $\epsilon$, $\gamma$, $\alpha$ | exploration probability, discount factor, learning rate, respectively |
