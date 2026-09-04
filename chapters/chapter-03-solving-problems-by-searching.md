# Chapter 3 — Solving Problems by Searching

**Course:** CAI 4002 — Introduction to Artificial Intelligence (USF Fall 2026)
**Sections:** 3.1 Problem-Solving Agents (pp. 82–86) · 3.3 Search Algorithms (pp. 89–94).

When no single action is obviously right, an agent looks ahead for a *sequence* of actions that leads to a goal — **search**.

---

## 1. Problem-Solving Agents and Search

> **Definition (Problem-solving agent).** An agent that plans ahead by considering a sequence of actions that form a path to a goal state is called a *problem-solving agent*.
>
> **Definition (Search).** The computational process a problem-solving agent undertakes to find such a sequence is called *search*.

Two framing points:

- Problem-solving agents use **atomic representations** (§2.4.7): states are treated as wholes with no internal structure visible to search. Agents using factored/structured state descriptions instead are *planning agents* (Chapters 7, 11).
- The chapter starts in the simplest environments: **episodic, single agent, fully observable, deterministic, static, discrete, known** (§2.3); later chapters relax these constraints.

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

**Paths and solutions.** A sequence of actions forms a **path**; a **solution** is a path from the initial state to a goal state. Action costs are assumed **additive**: the total cost of a path is the sum of its individual action costs. An **optimal solution** has the lowest path cost among all solutions. This chapter assumes all action costs are positive — with a net-negative-cost cycle, looping it forever would be "cost-optimal"; zero-cost actions are allowed as long as no path contains unboundedly many of them.

**Graph view.** The state space can be represented as a **graph**: vertices are states, directed edges are actions. Figure 3.1's Romania map *is* such a graph — each road indicates two actions, one in each direction.

---

## 4. §3.1.2 — Formulating Problems and Abstraction

The Bucharest formulation is a **model** — an abstract mathematical description — not the real thing. Compare the atomic state "Arad" to an actual cross-country trip: traveling companions, the radio program, scenery, proximity of law enforcement, distance to the next rest stop, road condition, weather, traffic… All left out because they're irrelevant to finding a route.

> **Definition (Abstraction).** The process of removing detail from a representation is called *abstraction*. A good problem formulation has the right level of detail — if actions were "move the right foot forward a centimeter" or "turn the steering wheel one degree left," the agent would never get out of the parking lot, let alone to Bucharest.

**Level of abstraction.** The abstract states and actions correspond to *large sets* of detailed world states and detailed action sequences. An abstract solution (Arad → Sibiu → Rimnicu Vilcea → Pitești → Bucharest) corresponds to many more-detailed paths — radio on between Sibiu and Rimnicu Vilcea, off for the rest of the trip, etc. Two criteria:

- **Valid** if any abstract solution can be *elaborated* into a solution in the more detailed world. A sufficient condition: for every detailed state that is "in Arad," there is a detailed path to some state that is "in Sibiu," and so on.
- **Useful** if carrying out each action of the abstract solution is easier than the original problem — "drive from Arad to Sibiu" needs no further search or planning by an average driver.

So: remove as much detail as possible while retaining validity and keeping abstract actions easy to carry out — without useful abstraction, agents are swamped by the real world's detail.

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
- Different choices of $f(n)$ give different specific algorithms: breadth-first, uniform-cost, depth-first, greedy best-first, and A* are all best-first search with a particular evaluation function.

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

# Day 2 — Search Algorithms (Lecture)

## 6. Planning Agents and Environment Assumptions

**Planning involves:**

1. decisions based on **hypothesized (modeled) consequences** of actions → a planning agent can't be model-free;
2. having a **goal to plan for** → it can't be a reflex agent either;
3. **searching** for a good plan.

For this chapter the lecture restricts attention to environments that are:

- **single-agent**,
- **fully observable**,
- **deterministic**,
- **static** (the world doesn't change while the agent thinks),
- **discrete**.

(No ghosts, no hidden information.) These assumptions make search well-behaved; relaxing them is what later chapters are about.

## 7. State-Space Sizes: World States vs. Search States

A model is an **abstraction**: a *precise* (not necessarily accurate) description of the real world. Too much detail → the problem becomes too hard to solve; too little detail → it becomes unsolvable. The right level keeps only what the plan depends on.

**Worked example — Pac-Man.** State components: agent position ($120$ squares), food (which $30$ dots remain, i.e. $2^{30}$ configurations), ghost positions ($12^2$ for two ghosts), and facing direction ($4$: N/S/E/W).

| Question | States needed | Count |
|---|---|---|
| Full world states? | all components | $120 \cdot 2^{30} \cdot 12^2 \cdot 4 \approx 7.42 \times 10^{13}$ |
| Pathfinding (get to a goal)? | agent position only | $120$ |
| Eat all dots? | agent position + food | $120 \cdot 2^{30} \approx 1.29 \times 10^{11}$ |

Two ways to choose the state space:

- **Option 1 — world state:** include every detail of the environment given our model (for pathfinding: current $(x, y)$, actions N/S/E/W, wall locations, goal location; for eat-all-dots: same but with dot locations instead of a goal).
- **Option 2 — search state:** keep only the planning-specific details. Pathfinding needs just the agent's position; ghosts and facing are irrelevant to *where* it can go.

## 8. State-Space Graphs vs. Search Trees, and the Frontier

**State-space graph.** A complete model of the search problem: nodes = abstracted world configurations (states), arcs = transitions between states (the successor function), goals = a set of nodes (or a goal test). Each state occurs **exactly once**. The full graph is typically too big to build, but it's useful for conceptual understanding and theoretical analysis.

**Search tree.** A branching series of modeled decisions: the start state is the root node, children are successors, plans are top-down paths from root to a goal. Crucially, **states are not unique in a search tree** — the same state can appear under several nodes (once per path that reaches it). So any cycle in the state-space graph makes the search tree infinite:

> How big is the search tree from $S$ when the graph has a cycle? $\infty$. States get repeated.

**How to perform a search.** Maintain the set of most-complete partial plans (tree nodes) — called the **frontier** or **fringe**. Expand out partial plans until a goal is reached. The whole art of search is choosing *which* node to expand next so as to try out as few as possible:

> All the algorithms in this chapter are the same procedure with different expansion strategies.

## 9. Uninformed Search Strategies (§3.4)

An **uninformed** algorithm has no clue how close a state is to the goal — an agent in Arad with no knowledge of Romanian geography can't tell whether Zerind or Sibiu is the better first step. The lecture analyzes each strategy against the four criteria from §3.3.4, for a tree with branching factor $b$ and maximum depth $m$.

### 9.1 Breadth-First Search (BFS)

**Strategy: expand the shallowest node first.** Expand the root, then all its successors, then their successors — level by level. Implemented as best-first search with $f(n)$ = depth of the node; a FIFO queue gives the right order for free (new nodes are always deeper than their parents). Two efficiency tricks from §3.4.1:

- `reached` can be a plain **set** of states, not a state→node map — once BFS reaches a state it can never find a better path to it;
- an **early goal test**: check IS-GOAL as soon as a node is *generated*, rather than when it's expanded.

BFS always finds a solution with the minimal number of actions: while generating nodes at depth $d$ it has already generated every node at depth $d - 1$, so any shallower solution would have been found first.

| Criterion | Result |
|---|---|
| Completeness | **Yes** — systematic; won't get stuck in a cycle like DFS (complete even on infinite state spaces) |
| Optimality | **Only if all actions cost the same**; with varying costs, no |
| Time | $O(b^m)$ worst case; $O(b^s)$ if the solution is known to be at depth $s < m$ |
| Space | $O(b^m)$ — every node of the last tier stays in memory; $O(b^s)$ with a known solution depth |

**Why exponential space hurts.** With $b = 10$, processing 1 million nodes/second, and 1 Kbyte per node: searching to depth $d = 10$ takes under 3 hours but needs **10 terabytes** of memory; at depth $d = 14$ it would take 3.5 years even with infinite memory. Memory is the bigger problem for BFS than time — exponential-complexity search problems can't be solved by uninformed search except in their smallest instances.

### 9.2 Depth-First Search (DFS)

**Strategy: expand the deepest node first.** Dive to the deepest level, then "back up" to the next-deepest node with unexpanded successors. Implemented as a tree-like search using a **stack**, usually *without* a reached table — which is exactly why it's memory-cheap.

| Criterion | Result |
|---|---|
| Completeness | **No** — in cyclic state spaces it can loop forever; in infinite spaces it can get stuck on an infinite path even without cycles (incomplete). For finite tree-like state spaces it *is* complete and efficient |
| Optimality | **No** — returns the first ("leftmost") solution it finds, ignoring action costs entirely |
| Time | $O(b^m)$ without cycles; unbounded if a cycle exists (might try every path) |
| Space | $O(b \cdot m)$ — only one expansion per tier is kept in memory at once |

**The two practical problems with DFS and their fixes:**

- **Cycles** → pick a random child instead of always the first, or track visited states;
- **Revisiting states** → keep a table of visited (reached) states and ignore them.

### 9.3 Iterative Deepening

Use DFS but set a **maximum depth**, then repeat with an increasing limit — "the best of BFS & DFS": BFS's completeness and shallow-solution-first ordering, at DFS's memory cost. The depth limit needs an upper bound on path length; the number of possible states is one safe choice (for social networks, ~6 hops covers essentially everyone).

### 9.4 Uniform-Cost Search / Dijkstra's Algorithm (§3.4.2)

Both DFS and BFS ignore action costs. **Uniform-cost search (UCS)** — called **Dijkstra's algorithm** in theoretical CS — expands the **cheapest partial path first**: best-first search with $f(n)$ = path cost from the root, i.e. a priority queue ordered by $g(n)$.

Where BFS spreads out in waves of uniform *depth*, UCS spreads out in waves of uniform *path-cost*. The goal test happens at **expansion**, not generation — this matters:

> **Worked example — Sibiu → Bucharest (§3.4.2, Figure 3.10).** Sibiu's successors are Rimnicu Vilcea (cost 80) and Fagaras (99). UCS expands Rimnicu Vilcea first, adding Pitești at $80 + 97 = 177$. Next it expands Fagaras, which generates Bucharest at $99 + 211 = 310$ — but that node is only *generated*, not expanded, so the goal isn't detected yet. Expanding Pitești next yields a second path to Bucharest at $80 + 97 + 101 = 278$, which replaces the 310 path in `reached`. That node now has the lowest cost, gets expanded, and is returned as the goal. Had we tested for goals on *generation*, we would have returned the more expensive 310 path through Fagaras.

| Criterion | Result |
|---|---|
| Completeness | **Yes**, if all action costs are $> \epsilon > 0$ and a solution exists |
| Optimality | **Yes** — the first solution found has cost at least as low as any other node still in the frontier |
| Time | $O\left(b^{1 + \lfloor C^*/\epsilon \rfloor}\right)$, where $C^*$ = optimal solution cost and $\epsilon$ = lower bound on action cost — can be much worse than $b^d$, because UCS may explore large trees of cheap actions before taking one expensive but useful step |
| Space | $O\left(b^{1 + \lfloor C^*/\epsilon \rfloor}\right)$ — every node in the last tier of "effective depth" is stored |

When all action costs are equal, $b^{1+\lfloor C^*/\epsilon \rfloor}$ collapses to $b^{d+1}$ and UCS behaves like BFS. The weakness: **no information about which direction to go** — it explores in every direction at once. That's what informed search fixes.

### 9.5 One Search Algorithm for All Cases

DFS, BFS, and UCS are the *same* algorithm except for how the frontier is ordered:

| Strategy | Frontier data structure |
|---|---|
| DFS | stack (LIFO) |
| BFS | FIFO queue |
| Dijkstra's / UCS | priority queue ordered by path cost $g(n)$ |

So one implementation takes a strategy object with `.add()` and `.remove()` — swap the container, get a different algorithm.

## 10. Informed (Heuristic) Search Strategies (§3.5)

Uninformed algorithms search blindly in every direction until the goal happens to be found. An **informed** strategy uses domain-specific hints about where goals are:

> A **heuristic** $h(n)$ estimates the cost of the cheapest path from state $n$ to a goal — "how far do I still have to go?" For route-finding, the straight-line distance on the map is the classic example.

### 10.1 Greedy Best-First Search

**Strategy: expand the node with the lowest heuristic value.** Best-first search with $f(n) = h(n)$ — a priority queue keyed by $h$:

```python
from queue import PriorityQueue

class GreedySearchStrategyQueue:
    def __init__(self, heuristic):
        self.q = PriorityQueue()
        self.h = heuristic

    def add(self, state):
        self.q.put((self.h(state), state))

    def remove(self):
        cost, state = self.q.get()
        return state
```

It drops straight into the general tree-search framework — only the queue changes. On Romania with $h_{\text{SLD}}$ (straight-line distance to Bucharest), greedy search expands Arad → Sibiu → Fagaras → Bucharest and never touches a node off the solution path. But the found path is **not optimal**: via Sibiu–Fagaras it's 32 miles longer than through Rimnicu Vilcea–Pitești. "Greedy" because each step gets as close to a goal as possible, ignoring how much was already spent getting there — greediness can lead to worse results than being careful. Complete in finite state spaces (worst case $O(|V|)$), but not cost-optimal.

### 10.2 A* Search: Looking Both Backwards and Forwards

UCS orders by the **backwards** path cost $g(n)$ (how far I've come); greedy search orders by the **forward** goal estimate $h(n)$ (how far is left). **A\*** combines them — it orders by the sum:

$$
f_{\text{A}^*}(n) = g(n) + h(n) \;=\; \text{estimated cost of the best path that goes through } n
$$

**Termination subtlety.** A goal node may appear on the frontier before it's optimal. In the lecture's example, both $S \to B \to G$ ($f = 5$) and $S \to A \to G$ ($f = 4$) are generated; you must **wait until a goal is dequeued** (expanded), not just generated — otherwise you'd stop at the cost-5 path.

> In Romania, Bucharest first appears on the frontier at $f = 450$, but A* doesn't take it: Pitești sits at $f = 417$, so a solution as cheap as 417 might still exist through there. Only when a different path to Bucharest reaches $f = 418$ and becomes the frontier minimum is it expanded — and that's the optimal solution.

**What else do we need for optimality?** Consider: start $S$, goal $G$ reachable directly at cost 5, but $h(S) = 7$. Then $f(S \to G) = 5 + 0 = 5 < f(S) = 7$ — the estimate says going *toward* the goal is more expensive than being there. If we stopped on that, we'd stop too early. The fix:

> **Admissibility.** A heuristic $h$ is **admissible** if $h(n) \le h^*(n)$ for every node, where $h^*$ is the *true* cost to the nearest goal. In plain words: an admissible heuristic is **optimistic** — it never overestimates how far a state is from a goal.

**Why A* with an admissible heuristic is optimal (proof sketch from the slides).** Let $A$ be an optimal goal node and $B$ a suboptimal one; assume $h$ is admissible. Claim: $A$ exits the fringe before $B$. Suppose $B$ is on the fringe — then some ancestor $n$ of $A$ (on the optimal path) must also still be on the fringe, since A* only stops when it dequeues a goal. Because $h$ never overestimates,

$$
f(n) = g(n) + h(n) \le g(n) + h^*(n) = C^* < f(B),
$$

so $n$ has strictly lower $f$ than $B$ and is expanded first. Recursively, every ancestor of $A$ expands before $B$, so $A$ itself dequeues before $B$. A* therefore returns the optimal goal. (The textbook's version is a proof by contradiction: if A* returned a suboptimal path of cost $C > C^*$, some unexpanded node $n$ on the optimal path would have to satisfy both $f(n) > C^*$ and — by admissibility — $f(n) \le g^*(n) + h^*(n) = C^*$, a contradiction.)

**Consistency (textbook §3.5.3).** A stronger property: $h$ is **consistent** if for every node $n$ and successor $n'$ via action $a$,

$$
h(n) \le c(n, a, n') + h(n')
$$

— the triangle inequality (a side of a triangle can't exceed the sum of the other two). Every consistent heuristic is admissible (not vice versa), so A* with one is cost-optimal. Consistency also makes $f = g + h$ **monotonic** along any path, which means the first time we reach a state it's on an optimal path — no re-adding states to the frontier, no updating `reached`. With a consistent heuristic A* expands exactly the nodes with $f(n) < C^*$ (plus possibly some at $f = C^*$), and is **optimally efficient**: no other algorithm using the same heuristic can expand fewer.

**Beyond the lecture.** The textbook continues §3.5 with variants for when A*'s guarantees or memory use don't fit: **weighted A\*** ($f(n) = g(n) + W \cdot h(n)$, $W > 1$ — finds a solution within $W \times C^*$ of optimal while exploring far fewer states), **beam search** (keep only the $k$ best frontier nodes), and **iterative-deepening A\*** ($\text{IDA}^*$: iterative deepening with an $f$-cost cutoff instead of a depth cutoff — A* without storing all reached states).

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
| **Uninformed search** | no clue how close a state is to the goal — DFS, BFS, UCS (§3.4) |
| **BFS (breadth-first)** | expand shallowest node first; FIFO queue; complete; optimal only if all actions cost the same; time/space $O(b^m)$ ($O(b^s)$ with known solution depth $s$); memory is its killer |
| **DFS (depth-first)** | expand deepest node first; stack, no reached table; incomplete in cyclic/infinite spaces; not cost-optimal ("leftmost" solution); space only $O(b \cdot m)$ |
| **Iterative deepening** | DFS with an increasing max-depth limit — BFS's completeness at DFS's memory cost |
| **UCS / Dijkstra's algorithm** | expand cheapest partial path first (priority queue on $g(n)$); complete and optimal for positive costs; time/space $O(b^{1+\lfloor C^*/\epsilon \rfloor})$ |
| **Heuristic $h(n)$** | estimate of the cost from state $n$ to a goal — straight-line distance is the classic example (§3.5) |
| **Greedy best-first search** | expand node with lowest $h(n)$; fast but not optimal (Romania: 32 miles longer than A*) |
| **A\* search** | $f(n) = g(n) + h(n)$ — combines UCS's backwards cost with greedy's forward estimate; complete and optimal when $h$ is admissible |
| **Admissible heuristic** | $h(n) \le h^*(n)$ for all nodes — never overestimates the true cost to a goal (optimistic) |
| **Consistent heuristic** | $h(n) \le c(n, a, n') + h(n')$ (triangle inequality); implies admissibility; makes $f$ monotonic → no re-expansion of states |
| **Weighted A\*** | $f(n) = g(n) + W \cdot h(n)$ with $W > 1$: solution within $W \times C^*$ of optimal, explores far fewer states |
