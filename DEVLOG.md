# Development Log – The Torchbearer

**Student Name:** Charles Kim
**Student ID:** 133161532

> Instructions: Write at least four dated entries. Required entry types are marked below.
> Two to five sentences per entry is sufficient. Write entries as you go, not all in one
> sitting. Graders check that entries reflect genuine work across multiple sessions.
> Delete all blockquotes before submitting.

---

## Entry 1 – [05/11/2026]: Initial Plan

> Required. Write this before writing any code. Describe your plan: what you will
> implement first, what parts you expect to be difficult, and how you plan to test.

_For this project, my plan is to tackle it in two phases: First I'm going to precompute all pairwise shortest-path costs between spawn, relics, and exit using Dijkstra, then use DFS branch-and-bound to search relic orderings. I expect the pruning logic in `_explore` to be the hardest part. This is because  getting the lower bound tight enough to cut branches without ever discarding the optimal route. I'll test incrementally: unit-test `run_dijkstra` on a small hand-traced graph first, then verify `find_optimal_route` against the spec illustration before moving to the provided test suite._

---

## Entry 2 – [Date]: [Short description]

> Required. At least one entry must describe a bug, wrong assumption, or design change
> you encountered. Describe what went wrong and how you resolved it.

_Your entry here._

---

## Entry 3 – [Date]: [Short description]

_Your entry here._

---

## Entry 4 – [Date]: Post-Implementation Reflection

> Required. Written after your implementation is complete. Describe what you would
> change or improve given more time.

_Your entry here._

---

## Final Entry – [Date]: Time Estimate

> Required. Estimate minutes spent per part. Honesty is expected; accuracy is not graded.

| Part | Estimated Hours |
|---|---|
| Part 1: Problem Analysis | |
| Part 2: Precomputation Design | |
| Part 3: Algorithm Correctness | |
| Part 4: Search Design | |
| Part 5: State and Search Space | |
| Part 6: Pruning | |
| Part 7: Implementation | |
| README and DEVLOG writing | 10 minutes working on initial set up |
| **Total** | |
