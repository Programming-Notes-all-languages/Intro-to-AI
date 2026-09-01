# Chapter 3 — Solving Problems by Searching

**Course:** CAI 4002 — Introduction to Artificial Intelligence (USF Fall 2026)
**Section 3.1 · Problem-Solving Agents** — covered in Week 2 (Friday lecture). Textbook pp. 82–86.

Chapter 2 asked *what makes an agent rational*; Chapter 3 answers a more concrete question: what does the agent actually **do** when no single action is obviously right? It looks ahead and tries to find a *sequence* of actions that leads to a goal — that process is called **search**. This file covers §3.1 (the problem-solving framework); later sections (§3.2 example problems, §3.4 uninformed search, §3.5 informed search / A*) will be added here as they are covered in class.

---

## 1. Problem-Solving Agents and Search

> **Definition (Problem-solving agent).** An agent that plans ahead by considering a sequence of actions that form a path to a goal state is called a *problem-solving agent*.
>
> **Definition (Search).** The computational process a problem-solving agent undertakes to find such a sequence is called *search*.

Two framing notes from the chapter introduction:

- Problem-solving agents use **atomic representations** (§2.4.7): states of the world are treated as wholes, with no internal structure visible to the search algorithms. Agents that use factored/structured state descriptions instead are *planning agents* (Chapters 7 and 11).
- This chapter deliberately starts in the simplest possible environments: **episodic, single agent, fully observable, deterministic, static, discrete, and known** — exactly the properties catalogued in §2.3. Chapter 4 relaxes those constraints; Chapter 6 adds multiple agents.

The running example for the whole chapter: an agent on a touring vacation in Romania is currently in **Arad** and has a nonrefundable ticket out of **Bucharest** the next day. Street signs show three roads leaving Arad — toward Sibiu, Timișoara, or Zerind — none of which is the goal. With no map (an *unknown* environment), the agent can do no better than pick randomly; with a map (Figure 3.1, a simplified road map with distances in miles), it has real information to search over.

---

## 2. The Four-Phase Problem-Solving Process

With world information available, the agent follows four phases:

> **Goal formulation.** The agent adopts the goal of reaching Bucharest. Goals organize behavior by limiting the objectives and hence the actions to be considered.
>
> **Problem formulation.** The agent devises a description of the states and actions necessary to reach the goal — an *abstract model* of the relevant part of the world. For our agent, one good model is: actions travel from one city to an adjacent city, so the only fact that changes due to an action is the current city.
>
> **Search.** Before taking any action in the real world, the agent simulates sequences of actions *in its model*, searching until it finds a sequence that reaches the goal — such a sequence is called a **solution**. It may have to simulate many sequences that don't reach the goal before finding one (e.g., Arad → Sibiu → Făgăraș → Bucharest), or concluding no solution exists.
>
> **Execution.** The agent executes the actions in the solution, one at a time.

### 2.1 Open-Loop vs. Closed-Loop Execution

An important property: in a fully observable, deterministic, known environment, the solution to any problem is a **fixed sequence of actions** — drive to Sibiu, then Făgăraș, then Bucharest. If the model is correct, the agent can ignore its percepts while executing (closing its eyes, so to speak), because the solution is guaranteed to reach the goal:

- **Open-loop system** — ignoring percepts breaks the loop between agent and environment. Fine when the model is trusted; unsafe if the model might be wrong or the environment nondeterministic.
- **Closed-loop approach** — monitors percepts while executing (Section 4.4). Safer under uncertainty.

In partially observable or nondeterministic environments, a solution isn't a fixed sequence at all but a **branching strategy**: different future actions depending on what percepts arrive. The agent might plan Arad → Sibiu but keep a contingency for arriving in Zerind by accident, or finding a sign reading "Drum Închis" (Road Closed).

---

## 3. §3.1.1 — Search Problems and Solutions (Formal Definition)

> **Definition (Search problem).** A search problem is defined formally by five components:
>
> - **States** — the set of possible states that the environment can be in, called the *state space*.
> - **Initial state** — the state the agent starts in. Example: Arad.
> - **Goal states** — one or more goal states. Sometimes a single state (Bucharest), sometimes a small set of alternatives, sometimes a *property* that applies to many (even infinitely many) states — e.g., "no dirt in any location" in the vacuum world. All three cases are handled by specifying an **IS-GOAL** method.
> - **Actions** — given state $s$, $\text{ACTIONS}(s)$ returns a finite set of actions that can be executed in $s$; each is *applicable* in $s$.
> - **Transition model** — describes what each action does: $\text{RESULT}(s, a)$ returns the state resulting from doing action $a$ in state $s$.
>
> Plus an **action cost function**, written $\text{ACTION-COST}(s, a, s')$ when programming or $c(s, a, s')$ in math: the numeric cost of applying action $a$ in state $s$ to reach state $s'$. A good agent uses a cost function that reflects its own performance measure — for route-finding, road length in miles (Figure 3.1) or travel time.

**Worked example — Romania.** The two functions on the map:

$$
\text{ACTIONS}(\text{Arad}) = \lbrace ToSibiu,\ ToTimisoara,\ ToZerind \rbrace
$$

$$
\text{RESULT}(\text{Arad},\ ToZerind) = Zerind
$$

**Paths and solutions.** A sequence of actions forms a **path**; a **solution** is a path from the initial state to a goal state. Action costs are assumed **additive**: the total cost of a path is the sum of its individual action costs. An **optimal solution** has the lowest path cost among all solutions. This chapter assumes all action costs are positive (footnote 3: with a net-negative-cost cycle, the "cost-optimal" solution would be to loop around it forever; zero-cost actions are fine if consecutive ones are bounded).

**Graph view.** The state space can be represented as a **graph**: vertices are states, directed edges are actions. Figure 3.1's Romania map *is* such a graph — each road indicates two actions, one in each direction.

---

## 4. §3.1.2 — Formulating Problems and Abstraction

The Bucharest formulation is a **model** — an abstract mathematical description — not the real thing. Compare the atomic state "Arad" to an actual cross-country trip: traveling companions, the radio program, scenery, proximity of law enforcement, distance to the next rest stop, road condition, weather, traffic… All left out because they're irrelevant to finding a route.

> **Definition (Abstraction).** The process of removing detail from a representation is called *abstraction*. A good problem formulation has the right level of detail — if actions were "move the right foot forward a centimeter" or "turn the steering wheel one degree left," the agent would never get out of the parking lot, let alone to Bucharest.

**Level of abstraction.** The abstract states and actions correspond to *large sets* of detailed world states and detailed action sequences. An abstract solution (Arad → Sibiu → Rimnicu Vilcea → Pitești → Bucharest) corresponds to many more-detailed paths — radio on between Sibiu and Rimnicu Vilcea, off for the rest of the trip, etc. Two criteria:

- **Valid** if any abstract solution can be *elaborated* into a solution in the more detailed world. A sufficient condition: for every detailed state that is "in Arad," there is a detailed path to some state that is "in Sibiu," and so on.
- **Useful** if carrying out each action of the abstract solution is easier than the original problem — "drive from Arad to Sibiu" needs no further search or planning by an average driver.

So: remove as much detail as possible while retaining validity and keeping abstract actions easy to carry out. The book's closing line for the section: were it not for the ability to construct useful abstractions, intelligent agents would be completely swamped by the real world.

---

## 5. Check Your Understanding

1. Why does *goal formulation* come before *problem formulation*? What goes wrong if an agent formulates a problem without first limiting its objectives?
2. In the Romania example, why can the agent ignore percepts while executing the solution (open-loop)? Name one real-world event that would make closed-loop execution necessary.
3. Write out $\text{ACTIONS}(\text{Sibiu})$ and $\text{RESULT}(\text{Sibiu},\ ToArad)$ for the Romania map.
4. Why does this chapter assume all action costs are positive? What happens to "optimal" if a negative-cost cycle exists?
5. Give an example of a goal defined by a *property* rather than a single state, and explain how IS-GOAL handles it.
6. An abstraction is valid but not useful — what does that mean concretely, and why would you still reject the formulation?

---

## Quick Reference

| Term | Meaning |
|---|---|
| **Problem-solving agent** | plans ahead by considering sequences of actions forming a path to a goal state |
| **Search** | the computational process of finding such an action sequence |
| Four phases | goal formulation → problem formulation → search → execution |
| **States / state space** | set of all possible states the environment can be in |
| **Initial state** | where the agent starts (e.g., Arad) |
| **Goal states / IS-GOAL** | one state, a set of alternatives, or a property over many states; IS-GOAL tests membership |
| $\text{ACTIONS}(s)$ | finite set of actions applicable in state $s$ |
| **Transition model** $\text{RESULT}(s, a)$ | the state resulting from action $a$ in state $s$ |
| Action cost $c(s, a, s')$ | numeric cost of an action; path cost = sum (additive); assumed positive here |
| **Path / solution** | sequence of actions / path from initial state to a goal state |
| **Optimal solution** | lowest-cost path among all solutions |
| **State-space graph** | vertices = states, directed edges = actions (the Romania map is one) |
| **Abstraction** | removing detail; valid if elaborable into detailed solutions, useful if abstract actions are easier to execute |
| Open-loop / closed-loop | ignore percepts while executing fixed plan / monitor percepts and adapt |

---

## Where This Goes Next

- **§3.2 Example Problems** — grid worlds (the vacuum world reformulated with 8 states), the 8-puzzle, the traveling-salesman problem; standardized vs. real-world problems.
- **§3.3 Search Algorithms** — the general search framework: expanding nodes, frontier, and how to evaluate algorithms (completeness, optimality, time/space complexity).
- **§3.4 Uninformed Search Strategies** — breadth-first, depth-first, uniform-cost, iterative deepening; no estimate of goal distance available.
- **§3.5 Informed (Heuristic) Search Strategies** — greedy best-first and A* search using heuristics like straight-line distance to Bucharest (the Week 2 focus).
