# The Torchbearer

**Student Name:** Charles Kim
**Student ID:** 133161532
**Course:** CS 460 – Algorithms | Spring 2026

---

## Part 1: Problem Analysis

- **Why a single shortest-path run from S is not enough:** \
  _Dijkstra from S yields the cheapest path to every individual node, but cannot decide the order in which to visit the relic chambers. Different orderings produce different total costs and Dijkstra has no way to compare them._

- **What decision remains after all inter-location costs are known:** \
  _The ordering decision remains after all inter-location costs are known. Which sequence to visit the relic chambers so that the sum of the pairwise inter-location costs is minimized._

- **Why this requires a search over orders (one sentence):** \
  _The total fuel depends on the chosen sequence of relics, so the engine must search the space of possible orderings and cannot compute the answer with a single closed-form calculation._

---

## Part 2: Precomputation Design

### Part 2a: Source Selection

| Source Node Type | Why it is a source |
|---|---|
| _Spawn (S)_ | _The Torchbearer departs from S first, so we need cheapest distance from S to every relic and to the exit._ |
| _Each Relic Node (R<sub>1</sub>, ..., R<sub>k</sub>)_ | _After collecting a relic, the Torchbearer departs from it toward the next relic or the exit, so we need outgoing distances from every relic._ |

### Part 2b: Distance Storage

| Property | Your answer |
|---|---|
| Data structure name | Nested dictionary (`dict[node, dict[node, float]]`). |
| What the keys represent | Source node (outer key) and destination node (inner key). |
| What the values represent | Minimum fuel cost (shortest-path distance) from the source to the destination. `float('inf')` if unreachable. |
| Lookup time complexity | O(1) average |
| Why O(1) lookup is possible | Python dictionaries are hash maps, so key hashing gives constant-time average access for both the outer and inner lookups. |

### Part 2c: Precomputation Complexity

- **Number of Dijkstra runs:** _k + 1 (one run per relic plus one run from spawn)._
- **Cost per run:** _O(m logn) where n=| V |, m=| E |._
- **Total complexity:** _O((k + 1)⋅ m logn) = O(k ⋅ m logn)._
- **Justification (one line):** _We run one independent Dijkstra from each of the k + 1 source nodes; each run processes every edge at most once with a log-n priority queue operation, giving O(m logn) per run._

---

## Part 3: Algorithm Correctness


### Part 3a: What the Invariant Means

- **For nodes already finalized (in S):**
  _Every node that has been extracted from the priority queue has its distance premanently set to the true shortest-path cost from the soruce, this means that nothing in the remaining graph can produce a cheaper route to it._

- **For nodes not yet finalized (not in S):**
  _`dist[u]` holds the length of the best path discovered SO FAR, whose intermediate vertices all belong to S, so it is a valid upper bound that may be still be improved as more nodes are finalized._

### Part 3b: Why Each Phase Holds

- **Initialization : why the invariant holds before iteration 1:**
  _S is empty and `dist[source] = 0` is the correct zero-length path with no internal nodes. Every other node starts at `float('inf')`, correctly reflecting that no path through S has been found yet._

- **Maintenance : why finalizing the min-dist node is always correct:**
  _The node u, extracted next has the smallest `dist[u]` among all non-finalized nodes. This is because all edge weights are nonnegative and any alternative path to u that routes through a non-finalized node v must cost at least `dist[v] ≥ dist[u]`, so it cannot be cheaper. Therefore, `dist[u]` is already optimal and adding u to S maintains the invariant._

- **Termination : what the invariant guarantees when the algorithm ends:**
  _When the heap is empty, every reachable node has been added to S, so the invariant guarantees that `dist[v]` equals the true shortest-path distance from the source to every reachable node v. Any unreachable nodes retain `float('inf')`._

### Part 3c: Why This Matters for the Route Planner

_If any distance in `dist_table` were incorrect, the route planner might choose a suboptimal ordering or wrongly declare a reachable exit unreachable, which would make the Torchbearer waste fuel or fail entirely._

---

## Part 4: Search Design

### Why Greedy Fails

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

- _The algorithm must explore every possible order in which the relic chambers can be visited, using branch-and-bound pruning to abandon any partial order whose optimisitc lower bound cannot beat the best complete order found so far._

---

## Part 5: State and Search Space

### Part 5a: State Representation

| Component | Variable name in code | Data type | Description |
|---|---|---|---|
| Current location | `current_loc` | node (any hashable type) | The dungeon node where the Torchbearer currently stands. |
| Relics already collected | `relics_visited_order` / `relics_remaining` | `list[node]` / `set[node]` | `relic_visited_order` records collection order and `relics_remaining` tracks what is still needed. |
| Fuel cost so far | `cost_so_far` | `float` | Accumulated torch fuel burned to reach `current_loc` on the current partial route. |

### Part 5b: Data Structure for Visited Relics

| Property | Your answer |
|---|---|
| Data structure chosen | `set` (Python built-in hash set), used as `relics_remaining`. |
| Operation: check if relic already collected | Time complexity: O(1) average, relic not in `relics_remaining`. |
| Operation: mark a relic as collected | Time complexity: O(1) average, `relics_remaining.remove(relic)`. |
| Operation: unmark a relic (backtrack) | Time complexity: O(1) average, `relics_remaining.add(relic)`. |
| Why this structure fits | All three critical operations (check, add, remove) are O(1) average in a hash set, keeping the overhead per recursive call constant regardless of how many relics there are.|

### Part 5c: Worst-Case Search Space

- **Worst-case number of orders considered:** _O(k!) where k = |M|._
- **Why:** _In the worst case, the algorithm must try all permutations of k relics. The number of permutations of k items is k!, so the search tree has at most k! leaves._

---

## Part 6: Pruning

### Part 6a: Best-So-Far Tracking

- **What is tracked:** _A mutable list `best = [best_cost, best_order]` shared across all recursive calls. `best[0]` is the total fuel of the cheapest complete valid route found so far, and `best[1]` is the corresponding relic ordering._
- **When it is used:** _At the start of every call to `_explore`, before expanding any child, the algorithm computes a lower bound on the cost of completing the current partial route and compares it to `best[0]`._
- **What it allows the algorithm to skip:** _Any partial route whose optimisitc lower bound is ≥ `best[0]` is abandoned immediately. The entire subtree of completions rooted at that state is never generated._

### Part 6b: Lower Bound Estimation

- **What information is available at the current state:** _The algorithm knows `cost_so_far`, the `current_loc`, and the set of `relics_remaining`, plus the full `dist_table` of precomputed pairwise shortest-path costs._
- **What the lower bound accounts for:** _When relics remain, the bound adds the minimum travel cost from `current_loc` to any remaining relic. When no relics remain, it adds the cost from `current_loc` directly to the exit._
- **Why it never overestimates:** _The minimum-next-hop distance is a lower bound because the Torchbearer must travel at least that far before it can make any further progress. The actual remaining cost (visiting all relics plus reaching the exit) can only be greater than or equal._

### Part 6c: Pruning Correctness

- _Pruning is safe because the lower bound never overestimates: if `lower_bound ≥ best[0]`, then every possible completion of the current partial route costs at least `best[0]`, so none of them can strictly improve the best solution. The optimal route, if it passes through this state, would have a lower bound strictly less than its own total cost, which in turn would be strictly less than `best[0]`, so a contradiction. Therefore, the optimal route is never in a subtree that gets pruned._

---

## References

- _Lecture Notes Only._
