# Chapter 2 — Intelligent Agents

**Course:** CAI 4002 — Introduction to Artificial Intelligence (USF Fall 2026)
**Section 2.1 · Agents and Environments** — covered in Week 1 (Friday lecture). Textbook pp. 54–56.

Chapter 1 introduced the idea that AI is about building *rational agents* — systems that behave as well as possible. Chapter 2 makes this concrete: what an agent is, what environments look like, and how the two are coupled together. This file covers §2.1; later sections (§2.2 rationality, §2.3 environment properties, §2.4 agent architectures) will be added here as they are covered in class.

---

## 1. What Is an Agent?

### 1.1 Definition

> **Definition (Agent).** An *agent* is anything that can be viewed as perceiving its environment through sensors and acting upon that environment through actuators.

Intuition: an agent is anything that *senses* something and then *does* something in response. The definition is deliberately broad — it covers humans, robots, and software alike, so the same design principles apply to all of them (Figure 2.1).

The four building blocks:

| Term | Role |
|---|---|
| **Environment** | everything outside the agent that it can sense or affect |
| **Sensor** | the part of the agent that perceives the environment |
| **Actuator** | the part of the agent that acts on the environment |
| **Percept / Action** | what comes in through sensors / what goes out through actuators |

### 1.2 Examples: Humans, Robots, Software

The book's three canonical examples — notice how "sensors" and "actuators" change meaning across them:

| Agent | Sensors | Actuators |
|---|---|---|
| **Human** | eyes, ears, other organs | hands, legs, vocal tract |
| **Robot** | cameras, infrared range finders | various motors |
| **Software agent** | file contents, network packets, human input (keyboard / mouse / touchscreen / voice) | writing files, sending network packets, displaying information, generating sounds |

A software agent is just as much an "agent" in this sense as a person: it perceives (reads files, receives packets, gets user input) and acts (writes files, sends packets, displays output). This framing is what lets us apply one set of design principles to all three.

### 1.3 What Counts as the Environment?

The environment *could* be everything — the entire universe. In practice it is just that part of the universe whose state we care about when designing this agent: **the part that affects what the agent perceives and that is affected by the agent's actions.**

**Example.** For a vacuum robot, "the environment" is not the whole building or the weather — it is the set of squares (and their dirt states) plus whatever else can change what the robot sees or does.

---

## 2. Percepts and Percept Sequences

> **Definition (Percept).** The *percept* is the content an agent's sensors are perceiving at a given moment.
>
> **Definition (Percept sequence).** An agent's *percept sequence* is the complete history of everything the agent has ever perceived.

The key principle behind all of AI in this book:

> An agent's choice of action at any instant can depend on its **built-in knowledge** and on the **entire percept sequence observed to date**, but **not on anything it hasn't perceived**.

Two consequences worth internalizing:

1. The full history matters, not just "right now." A good decision may require remembering what happened earlier (e.g., where you last saw dirt).
2. No action can be based on hidden information. If the agent cannot perceive something, a sound design will never pretend to know it — uncertainty has to be handled explicitly (that becomes §2.3's "partially observable" environments and later chapters' probability machinery).

---

## 3. Agent Function vs. Agent Program

> **Definition (Agent function).** The *agent function* maps any given percept sequence to an action. Specifying it for every possible percept sequence describes the agent's behavior completely — it is an *external characterization* of the agent.
>
> **Definition (Agent program).** The *agent program* is a concrete implementation of the agent function, running within some physical system. It is the *internal* side: actual code on an actual machine.

**Why two terms?** Because they are genuinely different things:

| | Agent function | Agent program |
|---|---|---|
| What it is | abstract mathematical mapping (percept sequence → action) | concrete implementation in a physical system |
| Viewpoint | external — "what does the agent do?" | internal — "how does it compute that?" |
| Size | infinite table for most agents | finite code |

**Tabulating the function.** In principle, given an agent to experiment with, we could build its full table by trying out every possible percept sequence and recording which action it takes. For real agents this table is enormous — **infinite unless we bound the length of percept sequences** we consider. (If the agent randomizes its actions, each sequence would have to be tried many times to estimate the probability of each action; acting randomly turns out to be very intelligent in some settings, as we'll see later.)

The practical upshot: no one builds agents by filling in an infinite table. The job of AI is to write a *smallish program* that produces rational behavior — which is exactly what the rest of the book is about.

---

## 4. Worked Example — The Vacuum-Cleaner World

The running example of this chapter (and much of the course) is a robotic vacuum cleaner in a world of squares, each either **clean** or **dirty**. Figure 2.2 uses just two squares, A and B; the agent starts in square A.

### 4.1 Specifying the World

- **Percepts:** which square the agent is in + whether that square has dirt → e.g., `[A, Clean]`, `[B, Dirty]`.
- **Actions:** move right, move left, suck (clean the current square), do nothing.
- **States of the world:** location × dirt = $\lbrace A, B \rbrace \times \lbrace Clean, Dirty \rbrace$ — four possible state combinations.

(That's a Cartesian product in the COT 4210 sense: every ordered pair with one choice from each set.)

> Footnote detail worth remembering: a real robot would not have actions like "move right" — it would have things like "spin wheels forward." The book picks page-friendly actions, not implementation-realistic ones.

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

### 4.3 The Question This Section Leaves Open

Different vacuum-world agents are defined simply by filling in the right-hand column of that table differently. So: *what is the right way to fill out the table?* What makes an agent good or bad, intelligent or stupid? §2.2 answers with **rationality**.

---

## 5. "Agent" Is a Tool for Analysis, Not a Boundary

Before closing the section, the book emphasizes that the notion of an agent is meant to be a *tool for analyzing systems*, not an absolute line dividing the world into agents and non-agents:

- You *could* view a hand-held calculator as an agent that chooses the action "display 4" when given the percept sequence "2 + 2 =" — but such an analysis would hardly aid our understanding of the calculator.
- In a sense, all engineering designs artifacts that interact with the world. AI operates at (the authors consider) the most interesting end of the spectrum: where the artifact has **significant computational resources** and the task environment requires **nontrivial decision making**.

---

## 6. Check Your Understanding

1. Identify the sensors and actuators of a software agent you use daily (e.g., an email client or a web browser).
2. Why is the full agent-function table infinite in general? What single restriction would make it finite?
3. In the vacuum world, why does `[A, Clean], [A, Clean]` get the same action as `[A, Clean]` alone? What does that tell you about what this particular agent uses to decide?
4. Why do we bother distinguishing *agent function* from *agent program*? Give one reason the distinction matters when designing an AI system.
5. Is a calculator an "agent" under the definition in §1.1? If so, why is that analysis not useful?

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

---

## Where This Goes Next

- **§2.2 Good Behavior: The Concept of Rationality** — performance measures, what "rational" means, omniscience vs. rationality, learning and autonomy.
- **§2.3 The Nature of Environments** — PEAS descriptions; properties like observable/partially observable, deterministic/nondeterministic, episodic/sequential, static/dynamic, discrete/continuous, known/unknown.
- **§2.4 The Structure of Agents** — simple reflex, model-based, goal-based, utility-based, and learning agents.
