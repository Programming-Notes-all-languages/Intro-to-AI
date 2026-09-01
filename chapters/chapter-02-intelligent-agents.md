# Chapter 2 — Intelligent Agents

**Course:** CAI 4002 — Introduction to Artificial Intelligence (USF Fall 2026)
**Sections:** 2.1 Agents and Environments (pp. 54–56) · 2.3 Properties of Task Environments (pp. 60–65) · 2.4.2 Simple Reflex Agents (pp. 67–69).

Chapter 1 introduced AI as building *rational agents* — systems that behave as well as possible; this chapter makes it concrete: what an agent is, what environments look like, and how the two couple together.

---

## 1. What Is an Agent?

### 1.1 Definition

> **Definition (Agent).** An *agent* is anything that can be viewed as perceiving its environment through sensors and acting upon that environment through actuators.

The definition is deliberately broad — it covers humans, robots, and software alike, so the same design principles apply to all (Figure 2.1).

The four building blocks:

| Term | Role |
|---|---|
| **Environment** | everything outside the agent that it can sense or affect |
| **Sensor** | the part of the agent that perceives the environment |
| **Actuator** | the part of the agent that acts on the environment |
| **Percept / Action** | what comes in through sensors / what goes out through actuators |

### 1.2 Examples: Humans, Robots, Software

Three canonical examples — note how "sensors" and "actuators" shift meaning across them:

| Agent | Sensors | Actuators |
|---|---|---|
| **Human** | eyes, ears, other organs | hands, legs, vocal tract |
| **Robot** | cameras, infrared range finders | various motors |
| **Software agent** | file contents, network packets, human input (keyboard / mouse / touchscreen / voice) | writing files, sending network packets, displaying information, generating sounds |

A software agent is just as much an "agent": it perceives (reads files, receives packets, gets user input) and acts (writes files, sends packets, displays output), so one set of design principles applies to all three.

### 1.3 What Counts as the Environment?

The environment *could* be everything — the entire universe. In practice it is just that part of the universe whose state we care about when designing this agent: **the part that affects what the agent perceives and that is affected by the agent's actions.**

**Example.** For a vacuum robot, "the environment" is not the whole building or the weather — it is the set of squares (and their dirt states) plus whatever else can change what the robot sees or does.

---

## 2. Percepts and Percept Sequences

> **Definition (Percept).** The *percept* is the content an agent's sensors are perceiving at a given moment.
>
> **Definition (Percept sequence).** An agent's *percept sequence* is the complete history of everything the agent has ever perceived.

The key principle:

> An agent's choice of action at any instant can depend on its **built-in knowledge** and on the **entire percept sequence observed to date**, but **not on anything it hasn't perceived**.

Consequences:

1. The full history matters, not just "right now." A good decision may require remembering what happened earlier (e.g., where you last saw dirt).
2. No action can be based on hidden information. If the agent cannot perceive something, a sound design will never pretend to know it — uncertainty must be handled explicitly (§2.3 partially observable environments; later probability machinery).

---

## 3. Agent Function vs. Agent Program

> **Definition (Agent function).** The *agent function* maps any given percept sequence to an action. Specifying it for every possible percept sequence describes the agent's behavior completely — it is an *external characterization* of the agent.
>
> **Definition (Agent program).** The *agent program* is a concrete implementation of the agent function, running within some physical system. It is the *internal* side: actual code on an actual machine.

**Why two terms?** They are genuinely different things:

| | Agent function | Agent program |
|---|---|---|
| What it is | abstract mathematical mapping (percept sequence → action) | concrete implementation in a physical system |
| Viewpoint | external — "what does the agent do?" | internal — "how does it compute that?" |
| Size | infinite table for most agents | finite code |

**Tabulating the function.** In principle, given an agent to experiment with, we could build its full table by trying out every possible percept sequence and recording which action it takes. For real agents this table is enormous — **infinite unless we bound the length of percept sequences** we consider. (If the agent randomizes its actions, each sequence must be tried many times to estimate the probability of each action — and acting randomly can itself be rational in some settings.)

The practical upshot: no one builds agents by filling in an infinite table. The job of AI is to write a *smallish program* that produces rational behavior.

---

## 4. Worked Example — The Vacuum-Cleaner World

The running example is a robotic vacuum cleaner in a world of squares, each either **clean** or **dirty**. Figure 2.2 uses just two squares, A and B; the agent starts in square A.

### 4.1 Specifying the World

- **Percepts:** which square the agent is in + whether that square has dirt → e.g., `[A, Clean]`, `[B, Dirty]`.
- **Actions:** move right, move left, suck (clean the current square), do nothing.
- **States of the world:** location × dirt = $\lbrace A, B \rbrace \times \lbrace Clean, Dirty \rbrace$ — four possible state combinations.

A real robot's actions would be things like "spin wheels forward," not "move right"; the vacuum world uses page-friendly actions, not implementation-realistic ones.

### 4.2 A Simple Agent Function

One very simple rule: **if the current square is dirty, Suck; otherwise move to the other square.**

A partial tabulation of this agent function (Figure 2.3):

| Percept sequence | Action |
|---|---|
| `[A, Clean]` | Right |
| `[A, Dirty]` | Suck |
| `[B, Clean]` | Left |
| `[B, Dirty]` | Suck |
| `[A, Clean], [A, Clean]` | Right |
| `[A, Clean], [A, Dirty]` | Suck |
| ... | ... |

**Observation.** Look at the last two rows: the action for `[A, Clean], [A, Clean]` is the same as for `[A, Clean]` alone. In fact, this agent's action depends only on the *last* percept — it ignores all earlier history (and its built-in knowledge). That's why the table keeps repeating itself. Agents that decide purely from the current percept are called **simple reflex agents** (§2.4.2); they work well in fully observable worlds but can get stuck in infinite loops otherwise.

### 4.3 What Makes One Agent Better Than Another?

Different vacuum-world agents differ only in how the right-hand column of that table is filled in — what makes one better than another is answered by **rationality** (§2.2).

---

## 5. "Agent" Is a Tool for Analysis, Not a Boundary

The notion of an agent is a *tool for analyzing systems*, not an absolute line dividing the world into agents and non-agents:

- You *could* view a hand-held calculator as an agent that chooses the action "display 4" when given the percept sequence "2 + 2 =" — but such an analysis would hardly aid our understanding of the calculator.
- In a sense, all engineering designs artifacts that interact with the world. AI operates at (the authors consider) the most interesting end of the spectrum: where the artifact has **significant computational resources** and the task environment requires **nontrivial decision making**.

---

## 6. Properties of Task Environments (§2.3)

A **task environment** is the "problem" to which rational agents are the "solutions," and its nature *directly determines* the appropriate agent design — classifying an environment along a few dimensions tells you which techniques apply. The remaining §2.3 properties (PEAS, episodic/sequential, static/dynamic, known/unknown) follow in later sections.

### 6.1 Fully Observable vs. Partially Observable

> **Definition.** A task environment is **fully observable** if the agent's sensors give it access to the *complete state* of the environment at each point in time. It is **partially observable** otherwise — and **unobservable** if the agent has no sensors at all.

Two refinements worth knowing:

- **"Effectively fully observable."** The sensors don't have to detect *everything* — only every aspect that is *relevant to the choice of action*. Relevance depends on the performance measure, so observability is always relative to what the agent actually has to do.
- **Why partial observability happens:** noisy/inaccurate sensors, or parts of the state simply missing from the sensor data.

**Examples.** A vacuum agent with only a *local* dirt sensor cannot tell whether other squares have dirt → partially observable. An automated taxi cannot see what other drivers are thinking → partially observable. (Even an unobservable environment isn't hopeless — Chapter 4 shows goals can sometimes still be achieved, with certainty.)

### 6.2 Single-Agent vs. Multiagent

> **Definition.** A task environment is **single-agent** if only one agent matters; it is **multiagent** if other rational agents are present whose behavior the agent must account for.

The subtle part: *which* entities count as agents? The key distinction is whether entity B's behavior is best described as **maximizing a performance measure whose value depends on agent A's behavior**. If so, treat it as an agent; if not, it's just an object obeying the laws of physics (like waves at the beach or leaves in the wind).

- **Chess** → *competitive* multiagent: by the rules, B maximizing its own performance measure minimizes A's.
- **Taxi driving** → *partially cooperative* (avoiding collisions maximizes everyone's performance) and *partially competitive* (only one car can take a parking space).

Design consequences: communication often emerges as rational behavior in multiagent settings, and in some competitive environments **randomized behavior is rational** because it avoids the pitfalls of predictability.

### 6.3 Deterministic vs. Nondeterministic (and "Stochastic")

> **Definition.** An environment is **deterministic** if the next state is *completely determined* by the current state and the action executed; otherwise it is **nondeterministic**.

- In a fully observable, deterministic environment an agent need not worry about uncertainty. But in a partially observable one, hidden aspects make the world *appear* nondeterministic — and most real situations are too complex to track all unobserved aspects, so they must be treated as nondeterministic for practical purposes (taxi driving: traffic is unpredictable, tires blow out, engines seize).
- The vacuum world (§4) is deterministic; variations can add randomly appearing dirt or an unreliable suction mechanism.

**Terminology trap — stochastic ≠ nondeterministic.** Some authors use "stochastic" as a synonym for "nondeterministic," but the book draws a line:

| Term | Meaning | Example phrasing |
|---|---|---|
| **Stochastic** | model explicitly deals with *probabilities* | "there's a 25% chance of rain tomorrow" |
| **Nondeterministic** | possibilities are listed but *not quantified* | "there's a chance of rain tomorrow" |

### 6.4 Discrete vs. Continuous

> **Definition.** The discrete/continuous distinction applies to the **state of the environment**, to how **time** is handled, and to the agent's **percepts and actions**.

- **Chess:** finite number of distinct states (excluding the clock), plus a discrete set of percepts and actions → fully discrete.
- **Taxi driving:** continuous-state *and* continuous-time — speed and location sweep smoothly through ranges of values; actions are continuous too (steering angles).
- Gray area: digital camera input is technically discrete, but is typically treated as representing continuously varying intensities and locations.

### 6.5 The Property Table (Figure 2.6)

Worked classification of familiar environments (Figure 2.6), first four properties:

| Task environment | Observable | Agents | Deterministic | Discrete |
|---|---|---|---|---|
| Crossword puzzle | Fully | Single | Deterministic | Discrete |
| Chess with a clock | Fully | Multi | Deterministic | Discrete |
| Poker | Partially | Multi | Stochastic | Discrete |
| Backgammon | Fully | Multi | Stochastic | Discrete |
| Taxi driving | Partially | Multi | Stochastic | Continuous |
| Medical diagnosis | Partially | Single | Stochastic | Continuous |
| Image analysis | Fully | Single | Deterministic | Continuous |
| Part-picking robot | Partially | Single | Stochastic | Continuous |
| Refinery controller | Partially | Single | Stochastic | Continuous |
| English tutor | Partially | Multi | Stochastic | Discrete |

Properties are *not always cut and dried* — medical diagnosis is listed single-agent because a disease process isn't profitably modeled as an agent, but recalcitrant patients could make it multiagent. Figure 2.6 omits "known/unknown" because that distinction concerns the *agent's knowledge of the environment's laws*, not strictly a property of the environment itself.

---

## 7. Simple Reflex Agents (§2.4.2)

How does an agent program implement its function? The four basic architectures are simple reflex, model-based reflex, goal-based, and utility-based (§2.4), plus learning agents (§2.4.6). The first is the simplest:

> **Definition (Simple reflex agent).** An agent that selects actions on the basis of the **current percept only**, ignoring the rest of the percept history.

### 7.1 The Vacuum Agent Program (Figure 2.8)

The vacuum agent from section 4.2 *is* a simple reflex agent — its decision depends only on current location and dirt status:

```
function REFLEX-VACUUM-AGENT([location, status]) returns an action
    if status = Dirty then return Suck
    else if location = A then return Right
    else if location = B then return Left
```

Notice the size difference: this tiny program replaces a table with $4^T$ rows (all percept sequences of length up to $T$). Ignoring history cuts the relevant cases from $4^T$ down to just 4, and the fact that "dirty → Suck" doesn't depend on location trims it further. Simple enough that it could be implemented as a Boolean circuit.

### 7.2 Condition–Action Rules (Figures 2.9–2.10)

The general pattern behind simple reflex agents:

> **Definition (Condition–action rule).** A rule of the form *if* condition *then* action — also called situation–action rules, productions, or if–then rules. Example: `if car-in-front-is-braking then initiate-braking`.

The general-purpose structure: sensors → percept → **INTERPRET-INPUT** (abstracts the percept into a state description) → **RULE-MATCH** (finds the first rule whose condition matches) → execute that rule's action. Humans run on the same pattern — some rules are learned (driving), some innate reflexes (blinking when something approaches the eye).

### 7.3 The Fatal Flaw: Partial Observability

A simple reflex agent works **only if the correct decision can be made from the current percept alone** — i.e., only in fully observable environments. Even a little unobservability causes serious trouble:

- **Braking example:** `car-in-front-is-braking` may not be decidable from a single video frame (older cars have taillight/brake-light configurations that look alike). A simple reflex taxi would either brake continuously and unnecessarily, or — worse — never brake at all.
- **Vacuum example:** strip the location sensor; percepts are just `[Dirty]` / `[Clean]`. Suck on `[Dirty]`, but on `[Clean]`? Moving Left fails forever if it starts in A; moving Right fails forever if it starts in B. **Infinite loops are often unavoidable** for simple reflex agents in partially observable environments.

### 7.4 Randomization as a Partial Fix

If the agent can randomize, escape from infinite loops becomes possible: on `[Clean]`, flip a coin between Left and Right — it reaches the other square in an average of two steps, then cleans it if dirty. So a *randomized* simple reflex agent can outperform a deterministic one here. But randomization is usually **not** rational in single-agent environments — it's a useful trick for simple reflex agents, and more sophisticated deterministic designs do much better. The proper fix for partial observability is an **internal state** that tracks what can't be seen now — the *model-based* reflex agent (§2.4.3).

---

## Quick Reference

| Term | Meaning |
|---|---|
| **Agent** | anything perceiving its environment through sensors and acting on it through actuators |
| **Environment** | part of the universe whose state we care about: affects percepts, affected by actions |
| **Sensor / Actuator** | perceives the environment / acts on it |
| **Percept** | content the sensors are perceiving right now |
| **Percept sequence** | complete history of everything ever perceived |
| **Agent function** | mapping from any percept sequence to an action (external characterization) |
| **Agent program** | concrete implementation of the agent function in a physical system |
| Vacuum world | squares A, B; each clean/dirty; actions: right, left, suck, do nothing |
| **Fully / partially observable** | sensors give access to the complete state (or at least all action-relevant parts) / some of the state is hidden or noisy |
| **Single- vs. multiagent** | only one agent matters / other entities maximize performance measures whose value depends on yours; competitive vs. cooperative |
| **Deterministic / nondeterministic** | next state completely determined by current state + action / not; *stochastic* = probabilities explicit, *nondeterministic* = possibilities unquantified |
| **Discrete / continuous** | finite distinct states, percepts, actions vs. smoothly varying values — applies to state, time, percepts, and actions |
| **Simple reflex agent** | acts on the current percept only via condition–action rules; works only in fully observable worlds (infinite loops otherwise); randomization is a partial fix |
