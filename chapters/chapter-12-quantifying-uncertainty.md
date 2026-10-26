# Chapter 12 — Quantifying Uncertainty

**Course:** CAI 4002 — Introduction to Artificial Intelligence (USF Fall 2026)

**Sections:** 12.1 Acting under Uncertainty (pp. 403–406) · 12.2 Basic Probability Notation (pp. 406–413) · 12.3 Inference Using Full Joint Distributions (pp. 413–415) · 12.5 Bayes’ Rule and Its Use (pp. 417–420). Supporting distinction: 12.4 Independence (pp. 415–417).

**Class notes:** *Day 8 — Soundness & Completeness + Probability Theory*, slides 49–87 · *Day 9 — Probability Theory + Markov Decision Processes*, slides 5–43.

## 1. Why Agents Need Probability

Real agents must act without knowing every relevant fact or guaranteeing every outcome. Listing all possible exceptions to a logical rule can be impractical; a symptom does not logically prove its suspected cause.

| Source of uncertainty | Meaning |
|---|---|
| **Laziness** | listing every exception or qualification is too much work |
| **Theoretical ignorance** | the domain's rules are not completely known |
| **Practical ignorance** | relevant observations or tests are unavailable |

> **Definition (Probability).** A numerical degree of belief relative to the information available to the agent.

A proposition still holds or fails in the actual world; uncertainty concerns the agent's knowledge. Different probabilities under different evidence need not contradict each other.

**Probability describes uncertainty; utility describes preference.** Decision theory combines them: a rational agent chooses the action with maximum expected utility, not necessarily the action with the highest chance of one desired outcome.

## 2. Random Variables, Domains, and Assignments

> **Definition (Random variable).** A variable representing an uncertain aspect of the world; formally, a function from possible worlds to a range of values.

The **domain** or **range** lists the values the variable can take. Capital letters name variables; lowercase names identify particular values or propositions.

| Notation | Meaning |
|---|---|
| $X$ | a random variable |
| $x$ | one possible value of $X$ |
| $X=x$ | an assignment, asserting that $X$ has value $x$ |
| $P(X=x)$ or $P(x)$ | one probability; the shorthand needs an unambiguous variable |
| $\mathbf{P}(X)$ | the entire distribution over $X$, in a stated value order |
| $P(x,y)$ | probability of both assignments: the comma means AND |
| $P(x\mid y)$ | probability of $x$ given evidence $y$ |

For Boolean $X$, the lecture writes the domain as $\lbrace +x,-x \rbrace$. Here **$+x$ means true and $-x$ means false**; these are outcome labels, not positive and negative numerical magnitudes. The textbook also abbreviates these propositions as $x$ and $\neg x$.

A **complete assignment** gives a value to every variable in the model and identifies a possible world. A partial assignment generally describes a set of worlds.

## 3. Probability Distributions and Normalization

> **Definition (Probability distribution).** An assignment of probability to each possible value of a discrete random variable.

Every probability is nonnegative, and the probabilities over all mutually exclusive, exhaustive values sum to one:

$$
0\le P(X=x)\le 1,\qquad \sum_x P(X=x)=1.
$$

A single probability is a number; a distribution is a collection of numbers. A finite discrete distribution can be stored as a table, but **not every distribution is a finite table**.

| Variable type | Representation / caution |
|---|---|
| **Discrete** | probabilities for individual values; a countably infinite domain is possible |
| **Continuous** | a probability density; interval probabilities come from area under the density |

For a continuous variable with a density, an exact point has probability zero. **A density value is not a point probability** and need not be bounded above by one. The finite-table sums below apply to discrete variables.

To normalize nonnegative weights, add them and divide each by their total:

$$
Z=\sum_x w(x),\qquad P(X=x)=\frac{w(x)}{Z},\qquad Z>0.
$$

Normalization preserves relative weights. If the total is zero, division cannot produce a distribution.

## 4. Joint Distributions and Events

> **Definition (Joint distribution).** A distribution assigning probability to every combination of values of a set of variables. A full joint includes every variable in the model.

The lecture's temperature–weather model is:

| $T$ | $W$ | $P(T,W)$ |
|---|---|---:|
| hot | sun | 0.4 |
| hot | rain | 0.1 |
| cold | sun | 0.2 |
| cold | rain | 0.3 |

These entries sum to one. With $n$ variables having $d$ values each, the full joint has $d^n$ entries; Boolean variables give $2^n$. This exponential size is a representation problem as well as an inference problem.

> **Definition (Event).** A set of outcomes. Its probability is the sum of the probabilities of its member worlds.

**AND** selects worlds satisfying both conditions. **OR is inclusive:** select worlds satisfying either condition, including both, but count each world only once.

$$
P(a\lor b)=P(a)+P(b)-P(a\land b),\qquad P(\neg a)=1-P(a).
$$

In the table, hot AND sun has probability $0.4$; hot has probability $0.5$; hot OR sun has probability $0.7$. Adding the hot and sun marginals without subtracting their overlap would count the hot–sun row twice.

For mutually exclusive events the overlap is zero, so their probabilities can be added directly. **Mutual exclusivity is not independence:** excluding simultaneous occurrence is different from leaving another event's probability unchanged.

The axioms constrain related beliefs, not just individual numbers. Inconsistent probabilities can permit a combination of bets guaranteeing a loss, regardless of the outcome.

## 5. Marginalization: Sum Out What Is Not Needed

> **Definition (Marginalization).** Obtaining a distribution over fewer variables by summing over every value of the variables being removed.

$$
\mathbf{P}(X)=\sum_y \mathbf{P}(X,Y=y).
$$

When several variables are removed, the sum ranges over all combinations of their values. No evidence has been asserted merely by summing a variable out.

The lecture table gives $\mathbf{P}(T)=\langle 0.5,0.5\rangle$ in hot–cold order and $\mathbf{P}(W)=\langle 0.6,0.4\rangle$ in sun–rain order.

Expanding each joint term using the product rule gives the **law of total probability**, called a conditioning rule in the textbook:

$$
P(x)=\sum_y P(x\mid y)P(y).
$$

This averages conditional cases using their probabilities; it does not mean observing that one particular $y$ occurred.

## 6. Conditional Probability and Conditional Distributions

> **Definition (Conditional probability).** The fraction of the evidence-compatible probability mass that also satisfies the query.

$$
P(x\mid e)=\frac{P(x,e)}{P(e)},\qquad P(e)>0.
$$

The evidence restricts which worlds remain under consideration; division rescales their total probability to one. A zero-probability evidence event cannot be used in this finite-table definition.

| Operation | What happens to the other variable? | Result |
|---|---|---|
| **Marginalize** | include all values and add | $\mathbf{P}(X)$ |
| **Condition** | fix an observed value, retain matching mass, normalize | $\mathbf{P}(X\mid e)$ |

A **conditional distribution** contains probabilities for every query value at a fixed evidence assignment. Each such distribution sums to one; an entire table containing several different evidence cases does not have to sum to one globally.

For the lecture table, selecting cold leaves sun–rain weights $\langle 0.2,0.3\rangle$. Normalizing gives $\mathbf{P}(W\mid\text{cold})=\langle 0.4,0.6\rangle$; selecting hot instead gives $\langle 0.8,0.2\rangle$.

**New evidence changes the conditioning context**, not the meaning of the old probability. A prior remains a valid statement about its original information state, even when a posterior is now more useful. Conditioning is not logical implication.

*Slide correction:* Day 8 slide 68 / Day 9 slide 24 prints the wrong denominator for the final conditional. The evidence is $+x$, so the denominator must sum the two $+x$ rows:

$$
P(-y\mid +x)=\frac{0.3}{0.2+0.3}=\frac{3}{5}.
$$

<details>
<summary>Example — Rain overall versus rain given heavy traffic</summary>

*Original practice modeled on the Day 8–9 joint-table conditional-probability exercises.*

An agent uses this joint probability model:

| Rain | Traffic | Joint probability |
|---|---|---:|
| Yes | Heavy | 0.30 |
| Yes | Light | 0.10 |
| No | Heavy | 0.15 |
| No | Light | 0.45 |

The agent observes that **traffic is heavy**. What is the probability that it is raining, given that observation?

<details>
<summary>Solution</summary>

**Answer: $2/3 \approx 0.667$.**

The observation restricts the possibilities to the **heavy-traffic rows**. Ignore the light-traffic rows for this conditional calculation:

| Rain | Traffic | Retained probability mass |
|---|---|---:|
| Yes | Heavy | 0.30 |
| No | Heavy | 0.15 |

Ask: **Of the heavy-traffic probability mass, what fraction includes rain?** Apply the conditional-probability definition above: divide the probability of rain AND heavy traffic by the total probability of heavy traffic.

| Quantity | Calculation | Result |
|---|---|---|
| Heavy traffic, the evidence | 0.30 + 0.15 | 0.45 |
| Rain AND heavy traffic | the matching joint-table entry | 0.30 |
| Rain GIVEN heavy traffic | 0.30 / 0.45 | 2/3, approximately 0.667 |

**Why 0.4 answers a different question:** adding 0.30 + 0.10 gives the probability of **rain overall**, including light traffic. It does not use the observation that traffic is heavy.

- **Rain overall:** add every rain row.
- **Rain given heavy traffic:** retain only heavy-traffic rows, then normalize by their total.

</details>
</details>

## 7. Inference by Enumeration: Select → Sum → Normalize

> **Definition (Probabilistic inference).** Computing a desired probability from known probabilities, usually a posterior given observed evidence.

Partition the model's variables by their role in the particular question:

| Role | Symbol | Meaning |
|---|---|---|
| **Query** | $Q$ | variable whose distribution is requested |
| **Evidence** | $E=e$ | observed variables and their fixed values |
| **Hidden** | $H$ | remaining unobserved variables to sum out |

1. **Select** joint entries consistent with the evidence.
2. **Sum** over all hidden assignments, grouping by query value.
3. **Normalize** the resulting weights across query values.

$$
\mathbf{P}(Q\mid e)=\alpha\sum_h\mathbf{P}(Q,e,h),\qquad \alpha=\frac{1}{P(e)}.
$$

The same normalization factor applies to every query value for this fixed evidence. It can be obtained from the weights' total without separately assessing $P(e)$.

For $\mathbf{P}(W\mid\text{winter})$ in the lecture's season–temperature–weather model, weather is the query, season is evidence, and temperature is hidden. If hot is also observed, temperature becomes evidence rather than hidden.

A complete discrete joint can answer any such query in principle. Storing it and scanning it by enumeration take $O(d^n)$ space and worst-case time, respectively; estimating all its entries can also be impractical.

## 8. Product Rule and Chain Rule

The product rule builds a joint from a marginal and a conditional:

$$
P(x,y)=P(x\mid y)P(y)=P(y\mid x)P(x).
$$

Repeated application gives the chain rule, using any chosen ordering of the variables:

$$
\begin{aligned}
P(x_1,x_2,x_3)&=P(x_1)P(x_2\mid x_1)P(x_3\mid x_1,x_2),\\
P(x_1,\ldots,x_n)&=P(x_1)\prod_{i=2}^{n}P(x_i\mid x_1,\ldots,x_{i-1}).
\end{aligned}
$$

**Neither rule assumes independence.** The conditionals retain the dependencies. Writing $P(x,y)=P(x)P(y)$ is the extra independence assertion, not the general product rule. Conditional factors are used on contexts where they are defined.

Changing the ordering changes the required conditional tables but not the represented joint. The chain rule alone does not guarantee a compact model.

## 9. Bayes’ Rule: Reverse the Conditional

Bayes’ rule equates the two product-rule factorizations, allowing an easier conditional direction to support inference in the reverse direction:

$$
P(h\mid e)=\frac{P(e\mid h)P(h)}{P(e)},\qquad P(e)>0.
$$

| Term | Interpretation |
|---|---|
| **Prior** $P(h)$ | belief in the hypothesis before the current evidence |
| **Likelihood** $P(e\mid h)$ | how probable the evidence is if that hypothesis holds |
| **Evidence probability** $P(e)$ | probability of the observation across all hypotheses; normalizes the result |
| **Posterior** $P(h\mid e)$ | belief in the hypothesis after observing the evidence |

For mutually exclusive, exhaustive hypothesis values, the denominator is found by total probability:

$$
P(e)=\sum_h P(e\mid h)P(h),\qquad \mathbf{P}(H\mid e)=\alpha\mathbf{P}(e\mid H)\mathbf{P}(H).
$$

A likelihood is not a posterior, and likelihoods across hypothesis values need not sum to one. Normalization does not remove the need for a model: every hypothesis needs its likelihood and prior weight.

**Base rates matter.** In the textbook's illustrative meningitis model, $P(s\mid m)=0.7$, $P(m)=1/50{,}000$, and $P(s)=0.01$, yielding $P(m\mid s)=0.0014$, or **0.14%**. A symptom can be common among cases of a rare disease while most people with that symptom do not have the disease. These are the textbook's hypothetical values, not clinical estimates.

This is the distinction between the **causal direction**, evidence given cause, and the **diagnostic direction**, cause given evidence. Conditional probability alone does not establish a causal relationship. If the prior changes, the diagnostic probability must be reconsidered even if the likelihood model stays fixed.

## 10. Combining Evidence Requires the Right Independence Claim

> **Definition (Independence).** Learning one variable's value leaves the other's distribution unchanged; the joint factors into the two marginals.

> **Definition (Conditional independence).** Once specified background information is known, learning one variable adds no information about the other.

These are different claims. In the textbook's dental model, toothache and a probe catching are related through cavity, but are modeled as independent **given whether a cavity is present**.

$$
\mathbf{P}(T,K\mid C)=\mathbf{P}(T\mid C)\mathbf{P}(K\mid C).
$$

Here $T$ is Toothache, $K$ is Catch, and $C$ is Cavity. Only with that conditional-independence assumption may the combined likelihood be split:

$$
\mathbf{P}(C\mid t,k)=\alpha\mathbf{P}(t,k\mid C)\mathbf{P}(C)=\alpha\mathbf{P}(t\mid C)\mathbf{P}(k\mid C)\mathbf{P}(C).
$$

Without it, keep the joint likelihood or use the full chain rule. Conditional independence can reduce representation size substantially; it is an additional property of the model, not a consequence of merely observing multiple pieces of evidence.

## Quick Reference

| Term / symbol | Meaning |
|---|---|
| **Random variable / domain** | uncertain feature / its possible values |
| **$+x$, $-x$** | Boolean true and false labels, not arithmetic signs |
| **$P(x)$ / $\mathbf{P}(X)$** | one probability / a distribution over values |
| **Full joint** | probability for each complete assignment; $d^n$ entries |
| **Event** | set of worlds; sum their probabilities without double-counting |
| **Marginalization** | sum out unwanted variables; no observation required |
| **Conditional distribution** | distribution at fixed evidence; each evidence case normalizes separately |
| **Normalization** | divide nonnegative weights by their positive total |
| **Enumeration** | select evidence-compatible worlds → sum hidden variables → normalize query |
| **Product / chain rule** | exact joint factorizations; independence is not required |
| **Bayes’ rule** | posterior = likelihood × prior / evidence probability |
| **Independence / conditional independence** | no information gained / no information gained after specified evidence |
| **Maximum expected utility** | choose using both outcome probabilities and preferences |
