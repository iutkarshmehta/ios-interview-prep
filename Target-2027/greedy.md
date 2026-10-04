# Greedy Algorithms: Theory, Identification, & LeetCode Patterns
1. What is a Greedy Algorithm?
A Greedy Algorithm builds a solution piece by piece, at each step choosing the immediate best option (locally optimal choice) with the expectation that this strategy leads to a globally optimal solution.

Unlike Dynamic Programming (DP) or Backtracking, a greedy algorithm makes its choice and never backtracks or reconsiders past choices.

2. Theoretical Foundations: When Does Greedy Work?
Greedy is mathematically sound only when a problem satisfies two properties:

1. Greedy Choice Property
A globally optimal solution can be reached by selecting the locally optimal choice at each step without needing to look ahead or revisit earlier decisions.

2. Optimal Substructure
An optimal solution to the overall problem contains within it the optimal solutions to its subproblems.

3. How to Know It's a Greedy Problem
A. Diagnostic Checklist
1. No Regret Property:
2. 
  - Does choosing item $A$ now eliminate an even better combination later?
  - 
  - If yes $\rightarrow$ Not Greedy (use DP or Backtracking).
  - 
  - If no $\rightarrow$ Greedy candidate.
  - 
3. Can sorting simplify decisions?
4. 
  - If sorting by start time, end time, ratio, magnitude, or deadline establishes a clear, deterministic processing order, the problem is very often greedy.
  - 
5. Continuous vs. Discrete:
6. 
  - Fractional / continuous resources (e.g., Fractional Knapsack) are almost always greedy.
  - 
  - 0/1 discrete selections with strict capacities (e.g., 0/1 Knapsack) usually fail greedy and require DP.
  - 
B. Proof Techniques (How to Validate It)
When analyzing whether a greedy approach is correct, use one of two proof paradigms:

- Exchange Argument:
- 
  1. Assume there exists an arbitrary optimal solution $O$ different from greedy solution $G$.
  2. 
  3. Find the first choice where $O$ differs from $G$.
  4. 
  5. Swap $O$'s choice with $G$'s choice.
  6. 
  7. Show that the new solution is just as good as (or better than) $O$, without violating any constraints.
  8. 
  9. By induction, $G$ can be transformed into an optimal solution without loss of quality.
  10. 
- Stay-Ahead Argument:
- 
  - Show that after each step $k$, the greedy algorithm's partial metric is at least as good as any other algorithm's metric at step $k$.
  - 
4. Greedy vs. Dynamic Programming
FeatureGreedy AlgorithmsDynamic ProgrammingDecision Point

Makes choice first, then solves subproblem

Solves all subproblems first, then picks best choice

Backtracking

Never revisits past choices

Explores and balances multiple subproblem outcomes

Overlapping Subproblems

Not strictly required

Mandatory

Time Complexity

Usually $O(N)$ or $O(N \log N)$ (dominated by sorting/heaps)

Usually $O(N^2)$, $O(N \times W)$, or higher

                 Does a local choice depend on subproblem results?
                                  /            \
                                 /              \
                               No               Yes
                              /                  \
                    [Greedy Algorithm]     [Dynamic Programming]
              Make choice -> Solve subproblem   Solve subproblems -> Make choice
5. LeetCode Greedy Patterns & Problem Map
Pattern 1: Interval Scheduling & Sweep Line
- Strategy: Sort intervals by end_time (to minimize footprint and leave room for subsequent intervals) or by start_time (to merge overlaps).
- 
- Key Problems:
- 
  - LeetCode 435: Non-overlapping Intervals
  - 
  - LeetCode 452: Minimum Number of Arrows to Burst Balloons
  - 
  - LeetCode 56: Merge Intervals
  - 
  - LeetCode 252: Meeting Rooms
  - 
Pattern 2: Farthest Reach / Horizon Tracking
- Strategy: Track the maximum boundary reachable with current resources/jumps, updating the horizon dynamically in a single linear pass.
- 
- Key Problems:
- 
  - LeetCode 55: Jump Game
  - 
  - LeetCode 45: Jump Game II
  - 
  - LeetCode 1024: Video Stitching
  - 
Pattern 3: Two-Pointer Extremes (Inward Squeeze)
- Strategy: Sort data and pair the smallest with the largest element, or move the boundary that forms the current bottleneck.
- 
- Key Problems:
- 
  - LeetCode 881: Boats to Save People
  - 
  - LeetCode 11: Container With Most Water
  - 
Pattern 4: Heap-Driven Dynamic Greedy
- Strategy: Use a Max-Heap or Min-Heap when the set of candidate choices updates dynamically after every step.
- 
- Key Problems:
- 
  - LeetCode 621: Task Scheduler
  - 
  - LeetCode 767: Reorganize String
  - 
  - LeetCode 502: IPO
  - 
  - LeetCode 1353: Maximum Number of Events That Can Be Attended
  - 
Pattern 5: Monotonic Stack (Lexicographical Greedy)
- Strategy: Greedily keep the smallest possible digit or character at higher-significance positions, popping worse previous choices if they can be replaced safely.
- 
- Key Problems:
- 
  - LeetCode 402: Remove K Digits
  - 
  - LeetCode 316: Remove Duplicate Letters
  - 
  - LeetCode 321: Create Maximum Number
  - 
Pattern 6: Two-Pass Bidirectional Greedy
- Strategy: When an element's choice depends on constraints from both left and right neighbors, run a forward pass to satisfy left-side conditions, then a backward pass to satisfy right-side conditions.
- 
- Key Problems:
- 
  - LeetCode 135: Candy
  - 
Pattern 7: Accumulate Every Local Delta
- Strategy: Instead of trying to find the single global minimum and maximum, greedily add every positive transition $(A[i] - A[i-1] > 0)$.
- 
- Key Problems:
- 
  - LeetCode 122: Best Time to Buy and Sell Stock II
  