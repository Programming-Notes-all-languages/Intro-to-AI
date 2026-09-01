# Chapter 3 — Solving Problems by Searching

**Course:** CAI 4002 — Introduction to Artificial Intelligence (USF Fall 2026)
**Section 3.1 · Problem-Solving Agents** — covered in Week 2 (Friday lecture). Textbook pp. 82–86.
**Section 3.3 · Search Algorithms** — covered in Week 2. Textbook pp. 89–94.

Chapter 2 asked *what makes an agent rational*; Chapter 3 answers a more concrete question: what does the agent actually **do** when no single action is obviously right? It looks ahead and tries to find a *sequence* of actions that leads to a goal — that process is called **search**. This file covers §3.1 (the problem-solving framework) and §3.3 (the general search-algorithm machinery); later sections (§3.2 example problems, §3.4 uninformed search, §3.5 informed search / A*) will be added here as they are covered in class.

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

## 5. §3.3 — Search Algorithms

> **Definition (Search algorithm).** A search algorithm takes a search problem as input and returns either a solution or an indication of failure.

The algorithms in this chapter superimpose a **search tree** over the state-space graph, forming paths from the initial state until one reaches a goal:

- Each node in the search tree corresponds to a *state*; each edge corresponds to an *action*; the root is the initial state (Arad).
- **State space vs. search tree.** The state space describes the (possibly infinite) set of states and the transitions between them. The search tree describes *paths* between those states: it may contain several nodes for one state (one per path), but each node has exactly one unique path back to the root — as in any tree.

**Worked example — Figure 3.4.** Three partial search trees for Arad → Bucharest:

1. Expand the root Arad using $\text{ACTIONS}(\text{Arad})$ and $\text{RESULT}$, generating three children — Sibiu, Timișoara, Zerind — all on the frontier (green in the figure).
2. Choose one child to expand next. This is "the essence of search—following up one option now and putting the others aside for later."
3. Expand Sibiu: its successors are generated as new nodes, leaving 6 unexpanded nodes on the frontier. One of them is Arad again (Arad → Sibiu → Arad) — a cycle; since it can't be part of an optimal path, search should not continue from there.

**Frontier and separation property.** The **frontier** is the set of nodes (and their states) that have been reached but not yet expanded. A state is said to be **reached** as soon as a node has been generated for it, whether or not that node has been expanded. The frontier separates two regions of the state-space graph: an **interior** where every state has been expanded and an **exterior** of states not yet reached (Figure 3.6).

> Some authors call the frontier the *open list* and the set of previously expanded nodes the *closed list*; in this book's terminology, closed = reached minus frontier.

### 5.1 §3.3.1 — Best-First Search

> **Definition (Best-first search).** A general approach to choosing which node on the frontier to expand next: pick the node $n$ with minimum value of some evaluation function $f(n)$.

The algorithm (Figure 3.7):

```text
function BEST-FIRST-SEARCH(problem, f) returns a solution node or failure
    node ← NODE(STATE = problem.INITIAL)
    frontier ← priority queue ordered by f, containing node
    reached ← lookup table with one entry: problem.INITIAL → node
    while not IS-EMPTY(frontier) do
        node ← POP(frontier)
        if problem.IS-GOAL(node.STATE) then return node
        for each child in EXPAND(problem, node) do
            s ← child.STATE
            if s is not in reached or child.PATH-COST < reached[s].PATH-COST then
                reached[s] ← child
                add child to frontier
    return failure

function EXPAND(problem, node) yields nodes
    s ← node.STATE
    for each action in problem.ACTIONS(s) do
        s′ ← RESULT(s, action)
        cost ← node.PATH-COST + ACTION-COST(s, action, s′)
        yield NODE(STATE = s′, PARENT = node, ACTION = action, PATH-COST = cost)
```

Key behaviors:

- Each iteration pops the frontier node with minimum $f(n)$; if its state is a goal, return it — a **late goal test**, checked when a node is *expanded*, not when it is generated.
- A child enters the frontier if its state has never been reached before, or if it is now being reached by a path cheaper than any previous one (the `reached` table keeps only the best path to each state).
- Different choices of $f(n)$ give different specific algorithms — that is how this chapter organizes itself: breadth-first, uniform-cost, depth-first, greedy best-first, and A* are all instances of best-first search with a particular evaluation function.

### 5.2 §3.3.2 — Search Data Structures

A node is a data structure with four components:

- `node.STATE` — the state to which the node corresponds;
- `node.PARENT` — the node in the tree that generated this node;
- `node.ACTION` — the action applied to the parent's state to generate this node;
- `node.PATH-COST` — total cost of the path from the initial state to this node (written $g(\text{node})$ in mathematical formulas).

Following PARENT pointers back from a goal node recovers the states and actions along the solution.

The frontier is stored as some kind of **queue**, because its operations are: IS-EMPTY, POP (remove and return the top node), TOP (peek without removing), and ADD (insert into its proper place). Three kinds of queues appear in this chapter:

| Queue | Pops first… | Used by |
|---|---|---|
| Priority queue | node with minimum $f(n)$ | best-first search |
| FIFO queue | oldest added node | breadth-first search (§3.4) |
| LIFO queue (stack) | most recently added node | depth-first search (§3.4) |

Reached states are stored in a **lookup table** (e.g., a hash table): key = state, value = the best node for that state.

### 5.3 §3.3.3 — Redundant Paths

> **Definition (Repeated state / cycle).** A state appearing more than once on a path is a *repeated state*; a loopy path containing one is called a *cycle*. Romania has only 20 states, yet its complete search tree is infinite because there is no limit to how often one can traverse a loop.

A **redundant path** is any worse way of reaching the same state — cycles are just a special case. Example: Sibiu via Arad–Sibiu (140 miles) versus Arad–Zerind–Oradea–Sibiu (297 miles); the second need not be considered in the quest for optimal paths.

**Why it matters — 10×10 grid.** An agent that can move to any of 8 adjacent squares reaches every square in 9 moves or fewer, but there are almost $8^9$ paths of length 9 (over 100 million) — the average cell is reachable by over a million redundant paths. Eliminating redundancy completes the search roughly a million times faster:

> "Algorithms that cannot remember the past are doomed to repeat it."

Three approaches:

1. **Remember all reached states** (what best-first search does): detect every redundant path and keep only the best one per state. Preferred when there is much redundancy and the table fits in memory.
2. **Don't track at all**: some formulations rarely or never reach the same state twice — e.g., assembly problems where each action adds a part and ordering constraints (A before B, but not B before A) make revisits impossible. Skipping the reached table saves memory. An algorithm that checks for redundant paths is a **graph search**; one that does not is a **tree-like search**. BEST-FIRST-SEARCH as written is a graph search; removing all references to `reached` gives a tree-like version — less memory, but it examines redundant paths and runs slower.
3. **Compromise — check for cycles only**: each node has a chain of parent pointers, so cycles can be detected with no extra memory by walking up the chain looking for an earlier occurrence of the state. Some implementations walk all the way to the root (eliminating every cycle); others check just a few links (parent, grandparent, great-grandparent) — constant time, eliminating short cycles and relying on other mechanisms for long ones.

### 5.4 §3.3.4 — Measuring Problem-Solving Performance

Four criteria for comparing search algorithms:

- **Completeness** — is the algorithm guaranteed to find a solution when one exists, and to correctly report failure when there isn't?
- **Cost optimality** — does it find the lowest-cost solution among all solutions? (Some authors call this *admissibility* or just *optimality*.)
- **Time complexity** — how long until a solution is found; measured in seconds, or more abstractly by the number of states and actions considered.
- **Space complexity** — how much memory the search needs.

**Completeness, finite vs. infinite.** With a single goal that could be anywhere, a complete algorithm must systematically explore every state reachable from the initial state. In *finite* state spaces this is straightforward: track paths and cut off cycles (Arad → Sibiu → Arad), and eventually every reachable state will have been reached.

In *infinite* state spaces more care is needed. An algorithm that repeatedly applies "factorial" in Knuth's 4 problem follows the infinite path $4 \to 4! \to (4!)! \to \cdots$; on an obstacle-free infinite grid, moving straight forward forever also traces a path of new states only. Neither ever revisits a state, yet both are incomplete because wide expanses of the state space are never reached. A complete algorithm must be **systematic** — e.g., on the infinite grid, spiral out: cover all cells $s$ steps from the origin before moving to cells $s+1$ steps away. And in an infinite state space with no solution, a sound algorithm has to search forever — it can't terminate because it can't know whether the next state will be a goal.

**Complexity measures.** For explicit graphs (like Romania's map), complexity is measured against graph size $|V| + |E|$ ($|V|$ = number of states, $|E|$ = number of distinct state/action pairs). Most AI problems have only an *implicit* state space (initial state + actions + transition model), so complexity is expressed in terms of:

- $d$ — depth of the optimal solution (number of actions);
- $m$ — maximum number of actions in any path;
- $b$ — branching factor (number of successors to consider per node).

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
| **Search tree** | superimposed over the state-space graph; nodes ↔ states, edges ↔ actions, root = initial state; several nodes per state possible, each node has a unique path back to the root |
| Expand / generate | apply ACTIONS + RESULT to a node → child (successor) nodes with PARENT pointers |
| **Frontier** | reached but not yet expanded (a.k.a. open list); separates interior from exterior |
| **Reached** | states that have a generated node, whether or not it has been expanded |
| **Best-first search** | expand the frontier node with minimum $f(n)$; different $f$ → different algorithms |
| Node structure | STATE, PARENT, ACTION, PATH-COST ($g(\text{node})$) |
| Frontier queues | priority queue (best-first), FIFO (BFS), LIFO/stack (DFS) |
| **Graph search / tree-like** | checks for redundant paths via a reached table / does not check (less memory, slower) |
| **Redundant path / cycle** | worse way to reach the same state / loopy path; 20 states → infinite tree |
| Four performance criteria | completeness, cost optimality, time complexity, space complexity |
| $d$, $m$, $b$ | depth of optimal solution, max path length, branching factor (implicit state spaces) |
