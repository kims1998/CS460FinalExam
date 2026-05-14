# The Torchbearer

**Student Name:** Charles Kim
**Student ID:** 133161532
**Course:** CS 460 – Algorithms | Spring 2026

> This README is your project documentation. Write it the way a developer would document
> their design decisions , bullet points, brief justifications, and concrete examples where
> required. You are not writing an essay. You are explaining what you built and why you built
> it that way. Delete all blockquotes like this one before submitting.

---

## Part 1: Problem Analysis

> Document why this problem is not just a shortest-path problem. Three bullet points, one
> per question. Each bullet should be 1-2 sentences max.

- **Why a single shortest-path run from S is not enough:** \
  _Dijkstra from S yields the cheapest path to every individual node, but cannot decide the order in which to visit the relic chambers. Different orderings produce different total costs and Dijkstra has no way to compare them._

- **What decision remains after all inter-location costs are known:** \
  _The ordering decision remains after all inter-location costs are known. Which sequence to visit the relic chambers so that the sum of the pairwise inter-location costs is minimized._

- **Why this requires a search over orders (one sentence):** \
  _The total fuel depends on the chosen sequence of relics, so the engine must search the space of possible orderings and cannot compute the answer with a single closed-form calculation._

---

## Part 2: Precomputation Design

### Part 2a: Source Selection

> List the source node types as a bullet list. For each, one-line reason.

| Source Node Type | Why it is a source |
|---|---|
| _Spawn (S)_ | _The Torchbearer departs from S first, so we need cheapest distance from S to every relic and to the exit._ |
| _Each Relic Node (R<sub>1</sub>, ..., R<sub>k</sub>)_ | _After collecting a relic, the Torchbearer departs from it toward the next relic or the exit, so we need outgoing distances from every relic._ |

### Part 2b: Distance Storage

> Fill in the table. No prose required.

| Property | Your answer |
|---|---|
| Data structure name | Nested dictionary (`dict[node, dict[node, float]]`). |
| What the keys represent | Source node (outer key) and destination node (inner key). |
| What the values represent | Minimum fuel cost (shortest-path distance) from the source to the destination. `float('inf')` if unreachable. |
| Lookup time complexity | O(1) average |
| Why O(1) lookup is possible | Python dictionaries are hash maps, so key hashing gives constant-time average access for both the outer and inner lookups. |

### Part 2c: Precomputation Complexity

> State the total complexity and show the arithmetic. Two to three lines max.

- **Number of Dijkstra runs:** _k + 1 (one run per relic plus one run from spawn)._
- **Cost per run:** _O(m logn) where n=| V |, m=| E |._
- **Total complexity:** _O((k + 1)⋅ m logn) = O(k ⋅ m logn)._
- **Justification (one line):** _We run one independent Dijkstra from each of the k + 1 source nodes; each run processes every edge at most once with a log-n priority queue operation, giving O(m logn) per run._

---

## Part 3: Algorithm Correctness

> Document your understanding of why Dijkstra produces correct distances.
> Bullet points and short sentences throughout. No paragraphs.

### Part 3a: What the Invariant Means

> Two bullets: one for finalized nodes, one for non-finalized nodes.
> Do not copy the invariant text from the spec.

- **For nodes already finalized (in S):**
  _Every node that has been extracted from the priority queue has its distance premanently set to the true shortest-path cost from the soruce, this means that nothing in the remaining graph can produce a cheaper route to it._

- **For nodes not yet finalized (not in S):**
  _`dist[u]` holds the length of the best path discovered SO FAR, whose intermediate vertices all belong to S, so it is a valid upper bound that may be still be improved as more nodes are finalized._

### Part 3b: Why Each Phase Holds

> One to two bullets per phase. Maintenance must mention nonnegative edge weights.

- **Initialization : why the invariant holds before iteration 1:**
  _S is empty and `dist[source] = 0` is the correct zero-length path with no internal nodes. Every other node starts at `float('inf')`, correctly reflecting that no path through S has been found yet._

- **Maintenance : why finalizing the min-dist node is always correct:**
  _The node u, extracted next has the smallest `dist[u]` among all non-finalized nodes. This is because all edge weights are nonnegative and any alternative path to u that routes through a non-finalized node v must cost at least `dist[v] ≥ dist[u]`, so it cannot be cheaper. Therefore, `dist[u]` is already optimal and adding u to S maintains the invariant._

- **Termination : what the invariant guarantees when the algorithm ends:**
  _When the heap is empty, every reachable node has been added to S, so the invariant guarantees that `dist[v]` equals the true shortest-path distance from the source to every reachable node v. Any unreachable nodes retain `float('inf')`._

### Part 3c: Why This Matters for the Route Planner

> One sentence connecting correct distances to correct routing decisions.

_If any distance in `dist_table` were incorrect, the route planner might choose a suboptimal ordering or wrongly declare a reachable exit unreachable, which would make the Torchbearer waste fuel or fail entirely._

---

## Part 4: Search Design

### Why Greedy Fails

> State the failure mode. Then give a concrete counter-example using specific node names
> or costs (you may use the illustration example from the spec). Three to five bullets.

- **The failure mode:** _Always advancing to the nearest unvisited relic ignores the downstream cost of reaching subsequent relics and the exit from that chosen relic._
- **Counter-example setup:** _Using the spec illustration:_ \
  _S → B costs 1, S → C costs 2, S → D costs 2;_ \
  _B → D costs 1, D → C costs 1, C → B costs 1;_ \
  _B → T and C → T costs 1;_ \
  _T costs 100._
- **What greedy picks:** _From S greedy picks B (nearest, cost 1). From B it picks D (cost 1). From D the only unvisited relic is C (cost 1). Then C → T costs 1. Total = 4. (Greedy is lucky here; swap D → T to 1 and C → T to 100 and greedy's last stop would be C, forcing C → T = 100)._
- **What optimal picks:** _The optimal algorithm evaluates all 6 orderings of {B, C, D} and selects the one that minimizes total fuel, even if the first step is not the cheapest available._
- **Why greedy loses:** _A cheap first move can strand the Torchbearer at a node with expensive onward edges, inflating the total cost beyond what a slightly more expensive first move would have produced._

### What the Algorithm Must Explore

> One bullet point. Must use the word "order."

- _The algorithm must explore every possible order in which the relic chambers can be visited, using branch-and-bound pruning to abandon any partial order whose optimisitc lower bound cannot beat the best complete order found so far._

---

## Part 5: State and Search Space

### Part 5a: State Representation

> Document the three components of your search state as a table.
> Variable names here must match exactly what you use in torchbearer.py.

| Component | Variable name in code | Data type | Description |
|---|---|---|---|
| Current location | | | |
| Relics already collected | | | |
| Fuel cost so far | | | |

### Part 5b: Data Structure for Visited Relics

> Fill in the table.

| Property | Your answer |
|---|---|
| Data structure chosen | |
| Operation: check if relic already collected | Time complexity: |
| Operation: mark a relic as collected | Time complexity: |
| Operation: unmark a relic (backtrack) | Time complexity: |
| Why this structure fits | |

### Part 5c: Worst-Case Search Space

> Two bullet points.

- **Worst-case number of orders considered:** _Your answer (in terms of k)._
- **Why:** _One-line justification._

---

## Part 6: Pruning

### Part 6a: Best-So-Far Tracking

> Three bullet points.

- **What is tracked:** _Your answer here._
- **When it is used:** _Your answer here._
- **What it allows the algorithm to skip:** _Your answer here._

### Part 6b: Lower Bound Estimation

> Three bullet points.

- **What information is available at the current state:** _Your answer here._
- **What the lower bound accounts for:** _Your answer here._
- **Why it never overestimates:** _Your answer here._

### Part 6c: Pruning Correctness

> One to two bullet points. Explain why pruning is safe.

- _Your answer here._

---

## References

> Bullet list. If none beyond lecture notes, write that.

- _Your references here._
