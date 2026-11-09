# Chapter 23 — Reinforcement Learning

**Course:** CAI 4002 — Introduction to Artificial Intelligence (USF Fall 2026)

**Sections:** 23.1 Learning from Rewards (pp. 840–842) · 23.2 Passive Reinforcement Learning (pp. 842–848) · 23.3 Active Reinforcement Learning, introduction, §23.3.1 and §23.3.3 (pp. 848–850, 852–854) · 23.4 Generalization in Reinforcement Learning, introduction and §§23.4.1–23.4.2 (pp. 854–857), selections.

**Class notes:** *Day 10 — MDP & Value Iteration*, slides 59–82 · *Day 11 — Reinforcement Learning*. See [Chapter 16](chapter-16-making-complex-decisions.md) for MDPs, Bellman equations, and planning, including the Day 11 review.

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

## Day 11 — Exploration and Approximate Q-Learning

### 7. Optimistic Exploration

A purely greedy agent always chooses a currently best estimated action. It can settle on a suboptimal policy because it never discovers that an untried action is better. This does not contradict Q-learning's convergence theorem: the theorem requires sufficient exploration of state–action pairs.

> **Definition (Exploration function).** A function combines a current value estimate with a measure of how often a choice has been tried, making uncertain choices appear more attractive.

The lecture illustrates an optimistic bonus with a value estimate $u$ and a visit count $n$:

$$
f(u,n)=\begin{cases}
u+K/n & \text{if } 0<n<N_e,\\
u & \text{if } n\ge N_e.
\end{cases}
$$

Here $K>0$ controls the bonus and $N_e$ is the exploration threshold. The expression is undefined at $n=0$: unvisited choices must be handled separately, for example by giving them an optimistic initial score. The count can be associated with the particular state–action choice being scored. As the count grows, the bonus shrinks; at the threshold this particular bonus is removed.

The lecture also shows a modified backup that applies optimism to successor choices, allowing a bonus to propagate backward toward actions leading to unexplored areas. This is **not** the ordinary Q-learning target in Section 5. Alternatively, the textbook's exploratory Q-learning agent uses an exploration function to select behavior while retaining the ordinary Q-learning update. A finite bonus threshold alone does not guarantee the infinite exploration required by standard tabular Q-learning convergence.

### 8. Regret

> **Definition (Regret).** The difference between a specified optimal benchmark's expected cumulative reward and the learning agent's expected cumulative reward over the same time horizon.

Regret accounts for rewards lost **while learning**, not just the quality of the final policy. Two methods may eventually learn good policies but incur different regret because one spends more time on poor choices. The precise benchmark depends on the problem; in an MDP, actions also affect the states reached. Eventual convergence does not by itself imply low finite-time regret.

### 9. Features and Generalization

Tabular Q-learning stores a separate estimate for every state–action pair. Large state spaces can make both storage and visiting every pair impractical.

> **Definition (Feature).** A numerical property of a state or state–action pair used to describe it for learning.

For a game, features might describe the distance to food, distance to a ghost, or whether an action moves toward food. A feature can be binary or numeric; it need not be a raw input variable. Shared features let experience from one situation affect predictions for other situations.

In a **linear Q-function**, weights are shared across state–action pairs:

$$
\widehat{Q}_w(s,a)=\sum_{i=1}^{n}w_i f_i(s,a).
$$

The function is linear **in its weights**; the features themselves can be nonlinear functions, such as inverse distance. A constant feature can provide a bias term. Good features enable generalization, but two genuinely different situations can share misleading features, and the chosen representation might not express the correct values.

### 10. Approximate Q-Learning Update

Instead of changing a single table entry, approximate Q-learning updates the shared feature weights. For a sampled transition $(s,a,r,s')$, use the current approximation to form the target and error:

$$
\begin{aligned}
y &= r+\gamma\max_{a'\in A(s')}\widehat{Q}_w(s',a'),\\
\delta &= y-\widehat{Q}_w(s,a),\\
w_i &\leftarrow w_i+\alpha\delta f_i(s,a).
\end{aligned}
$$

For a terminal successor, the future term is zero. A positive error increases the weights of positive-valued active features; a negative error decreases them. A zero feature causes no change to its corresponding weight on that update. Since the weights are shared, changing them can also change predictions for other state–action pairs.

The target itself uses current estimates, so it changes as learning proceeds. This is a **semi-gradient** update: it treats the current target as fixed while adjusting the prediction. Unlike standard tabular Q-learning, approximate Q-learning does not generally inherit the same convergence guarantee; approximation and bootstrapping can be unstable.

### 11. Connection to Least Squares

For a fixed target $y$ and prediction $\widehat{y}$, the residual is the target minus the prediction. One common single-example loss is half the squared residual:

$$
L(w)=\frac{1}{2}(y-\widehat{y})^2,
\qquad \widehat{y}=\sum_{i=1}^{n}w_i f_i.
$$

The gradient is the vector of partial derivatives and points toward the greatest local increase in loss. Gradient descent moves the weights in the opposite direction. With a linear prediction and fixed target, the update is:

$$
w_i\leftarrow w_i-\alpha\frac{\partial L}{\partial w_i}=
w_i+\alpha(y-\widehat{y})f_i.
$$

Approximate Q-learning has this same form when its sampled TD target is held fixed for the update. Unlike ordinary supervised regression with a known fixed label, its target includes an estimate of the future and can change after later updates.

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
| Exploration function | adds optimism for insufficiently tried choices |
| Regret | cumulative reward gap relative to a specified optimal benchmark |
| Feature | numerical property describing a state or state–action pair |
| Linear Q approximation | weighted sum of shared state–action features |
| Approximate Q update | adjust weights using TD error and feature values |
| Gradient descent | move parameters against the loss gradient |
