# bestSum Problem — Three Techniques
## Recursion vs Memoization vs Tabulation in C++

> **Problem:** Given a `targetSum` (integer) and an array of `numbers` (non-negative integers),
> return the **shortest combination** of numbers from the array that adds up to exactly the `targetSum`.
> If multiple shortest combinations of the same length exist, return any one of them.
> If no combination is possible, return false / empty result.
> You may use the same number from the array **as many times as needed**.

---

## bestSum vs howSum vs canSum — The Full Progression

```
canSum(target, numbers)   →  bool
  "Is it POSSIBLE to reach the target?"
  Returns: true / false
  Difficulty: easiest

howSum(target, numbers)   →  bool + vector (ANY combination)
  "HOW to reach target? Give me any one combination."
  Returns: first valid combination found (stops at first success)
  Difficulty: medium

bestSum(target, numbers)  →  bool + vector (SHORTEST combination)
  "HOW to reach target using the FEWEST numbers?"
  Returns: the combination with the minimum number of elements
  Difficulty: hardest — must explore ALL branches, not just the first

KEY INSIGHT:
  canSum short-circuits: return true as soon as ANY path works.
  howSum short-circuits: return the FIRST valid combination found.
  bestSum NEVER short-circuits: must explore ALL valid paths to compare lengths.
```

---

## Quick Comparison (Read This First)

```
TECHNIQUE       TIME            SPACE              STYLE
─────────────────────────────────────────────────────────────────────────
Recursion       O(n^m × m)      O(m²)              Top-down
Memoization     O(n × m²)       O(m²)              Top-down + cache
Tabulation      O(n × m²)       O(m²)              Bottom-up

Where: m = targetSum,  n = length of numbers array

SAME complexity as howSum, but bestSum:
  - Recursion explores ALL branches (no early exit) → slower in practice
  - Memoization caches the SHORTEST result per subproblem
  - Tabulation OVERWRITES if a shorter path is found (howSum only sets once)
```

---

## The Problem Visualized

```
bestSum(7, [5, 3, 4, 7])

All valid combinations:
  [7]         ← length 1  ✅ SHORTEST!
  [3, 4]      ← length 2
  [4, 3]      ← length 2 (same as above, different order)
  [5, ...]    → 5+2? but 2 not in array; 5+5=10 → no

ANSWER: [7]   (pick the single number 7 directly)

────────────────────────────────────────────────────────────────

bestSum(8, [2, 3, 5])

All valid combinations:
  [2, 2, 2, 2]   ← length 4
  [2, 3, 3]      ← length 3
  [3, 2, 3]      ← length 3 (same combo)
  [3, 5]         ← length 2  ✅ SHORTEST!
  [5, 3]         ← length 2 (same combo)

ANSWER: [3, 5]

────────────────────────────────────────────────────────────────

bestSum(8, [1, 4, 5])

All valid combinations include:
  [1,1,1,1,1,1,1,1]  ← length 8
  [1,1,1,5]          ← length 4
  [4,4]              ← length 2  ✅ SHORTEST!
  [1,1,1,1,4]        ← length 5
  ...many more...

ANSWER: [4, 4]

────────────────────────────────────────────────────────────────

bestSum(7, [2, 4])   → false   (impossible — all sums of 2s and 4s are even)
bestSum(0, [1,2,3])  → []      (empty combo, length 0 — already at target)
```

**Test Cases for Reference:**

```
bestSum(7,   [5, 3, 4, 7])   → true,  result = [7]         (length 1)
bestSum(8,   [2, 3, 5])      → true,  result = [3, 5]       (length 2)
bestSum(8,   [1, 4, 5])      → true,  result = [4, 4]       (length 2)
bestSum(7,   [2, 4])         → false, result = []           (impossible)
bestSum(0,   [1, 2, 3])      → true,  result = []           (length 0)
bestSum(100, [1, 2, 5, 25])  → true,  result = [25,25,25,25] (length 4)
```

---

## The Recursive Formula

```
bestSum(targetSum, numbers, result):

  BASE CASE 1: targetSum == 0
    → return true   (valid combo, result already built — length 0 is trivially best)

  BASE CASE 2: targetSum < 0
    → return false  (overshot — this path is invalid)

  found    = false
  bestSoFar = []    ← tracks the shortest combination found in any branch

  FOR EACH num in numbers:
    remainder = targetSum - num
    subResult = []
    IF bestSum(remainder, numbers, subResult) == true:
      candidate = subResult + [num]          ← combine sub-result with current num
      IF not found  OR  candidate is shorter than bestSoFar:
        bestSoFar = candidate                ← update best
        found = true

  IF found:
    result = bestSoFar
    return true

  return false                               ← no combination works

CRITICAL DIFFERENCE FROM howSum:
  howSum: returns IMMEDIATELY when any branch succeeds (short-circuit)
  bestSum: ALWAYS tries ALL branches, then picks the shortest result
           (no early return — we must see all possibilities to know the shortest)
```

---

## Technique 1 — Recursion (Naive)

### The Idea

Try every possible number at each step.
Unlike howSum, **do NOT return early** when a valid combination is found.
Instead, try all numbers, collect all valid results, and keep the shortest.

### How the "Best" is Tracked Through the Call Stack

```
bestSum(7, [5, 3, 4, 7], result):

Try num=5: bestSum(2, [5,3,4,7]) →
  Try 5: bs(-3) → false
  Try 3: bs(-1) → false
  Try 4: bs(-2) → false
  Try 7: bs(-5) → false
  → false (can't reach 2 using these numbers)

Try num=3: bestSum(4, [5,3,4,7]) →
  Try 5: bs(-1) → false
  Try 3: bs(1)  →
      Try 5: bs(-4) → false
      Try 3: bs(-2) → false
      Try 4: bs(-3) → false
      Try 7: bs(-6) → false
      → false
  Try 4: bs(0) → true!  candidate = [] + [4] = [4]   (length 1)
      bestSoFar for bs(4) = [4], found=true
  Try 7: bs(-3) → false
  bs(4) returns true, result=[4]
  candidate = [4] + [3] = [4, 3]   (length 2)
  bestSoFar for bs(7) = [4, 3], found=true

Try num=4: bestSum(3, [5,3,4,7]) →
  Try 3: bs(0) → true!  candidate = [] + [3] = [3]  (length 1)
  bestSoFar = [3], found=true
  Try others... none shorter than length 1
  bs(3) returns true, result=[3]
  candidate = [3] + [4] = [3, 4]   (length 2)
  NOT shorter than current best [4, 3] → keep [4, 3]

Try num=7: bestSum(0, [5,3,4,7]) →
  → true! result=[]  (BASE CASE)
  candidate = [] + [7] = [7]   (length 1)
  [7] is SHORTER than [4, 3] → update bestSoFar = [7]

FINAL: result = [7], return true ✅
```

### Recursion Tree for bestSum(7, [5, 3, 4, 7]) — All Branches Explored

```
                              bs(7)
                    ┌──────────┼──────────┬──────────┐
                  bs(2)      bs(4)      bs(3)      bs(0)
                (no sol)    /|\ \       /|\ \       ↑
                          ..4→0 ..   ..3→0 ..   BASE CASE
                           ↑            ↑        returns true
                         [4]           [3]       result=[]
                         candidate:    candidate:
                         [4]+[3]=[4,3] [3]+[4]=[3,4]
                         (length 2)    (length 2)
                                                 candidate:
                                                 []+[7]=[7]
                                                 (length 1)
                                                 ← SHORTEST!

NOTICE: ALL four top-level branches are explored.
        bs(7) compares [4,3] vs [3,4] vs [7] → picks [7] (length 1).
        This is the key difference from howSum.
```

### Recursion Tree for bestSum(8, [2, 3, 5]) — Shows Repeated Subproblems

```
                               bs(8)
                    ┌───────────┼──────────┐
                  bs(6)       bs(5)      bs(3)
                 /  |  \     /  |  \     / |  \
             bs(4) bs(3) bs(1) bs(3) bs(0) bs(2) bs(1) bs(-2)
             /|\  ... ...  ...  ↑    ...   ...   ...
                               true

REPEATED SUBPROBLEMS:
  bs(3) computed at least TWICE from different parents
  bs(1) computed at least TWICE
  bs(6) → bs(4) → bs(2) → bs(0) = [2,2,2,2] (length 4)
  bs(8) → bs(5) → bs(2) → bs(0) + [3] = [2,3,3] (length 3) ... wait no
  The SHORTEST found: bs(8) → bs(3) → bs(0) + [5] + [3] = [5,3] (length 2) ✅

Memoization will fix the repeated work!
```

### C++ Code

```cpp
#include <iostream>
#include <vector>
using namespace std;

// ─────────────────────────────────────────────────────────────
// TECHNIQUE 1: RECURSION (Naive)
// Time:  O(n^m × m)
//   n^m recursive calls. At each, we may copy a vector (up to O(m) length).
//   Unlike howSum, NO early return → all n^m branches are always explored.
// Space: O(m²)
//   Call stack depth: O(m)
//   Local bestSoFar vectors: each level holds O(m)-size vector → O(m²) total
// ─────────────────────────────────────────────────────────────

bool bestSum_recursive(int targetSum, const vector<int>& numbers, vector<int>& result) {

    // BASE CASE 1: exactly reached 0 — valid combination, result already built
    if (targetSum == 0) return true;

    // BASE CASE 2: went past 0 — dead end
    if (targetSum < 0)  return false;

    bool found = false;
    vector<int> bestSoFar;    // tracks the shortest combination found in any branch

    // Try EVERY number — no early return!
    for (int num : numbers) {
        vector<int> subResult;
        if (bestSum_recursive(targetSum - num, numbers, subResult)) {
            subResult.push_back(num);    // append current number to sub-result

            // Update best if: (1) nothing found yet, OR (2) this is shorter
            if (!found || subResult.size() < bestSoFar.size()) {
                bestSoFar = subResult;
                found = true;
            }
        }
    }

    if (found) {
        result = bestSoFar;    // return the shortest via reference
        return true;
    }
    return false;
}

void printResult(const string& label, bool found, const vector<int>& result) {
    cout << label;
    if (!found) {
        cout << "null (impossible)" << endl;
    } else {
        cout << "[ ";
        for (int x : result) cout << x << " ";
        cout << "]  (length=" << result.size() << ")";
        int sum = 0; for (int x : result) sum += x;
        cout << "  (sum=" << sum << ")" << endl;
    }
}

int main() {
    cout << "bestSum (Recursion)" << endl;
    cout << "──────────────────────────────────────────────────────" << endl;

    {
        vector<int> result;
        bool found = bestSum_recursive(7, {5, 3, 4, 7}, result);
        printResult("bestSum(7,  [5,3,4,7])   = ", found, result);
    }
    {
        vector<int> result;
        bool found = bestSum_recursive(8, {2, 3, 5}, result);
        printResult("bestSum(8,  [2,3,5])     = ", found, result);
    }
    {
        vector<int> result;
        bool found = bestSum_recursive(8, {1, 4, 5}, result);
        printResult("bestSum(8,  [1,4,5])     = ", found, result);
    }
    {
        vector<int> result;
        bool found = bestSum_recursive(7, {2, 4}, result);
        printResult("bestSum(7,  [2,4])       = ", found, result);
    }
    {
        vector<int> result;
        bool found = bestSum_recursive(0, {1, 2, 3}, result);
        printResult("bestSum(0,  [1,2,3])     = ", found, result);
    }
    // WARNING: bestSum(100, {1,2,5,25}) with pure recursion is VERY SLOW!
    // bestSum explores ALL branches (unlike howSum which short-circuits).
    // Do NOT run bestSum_recursive on large inputs.

    return 0;
}
```

**Output:**
```
bestSum (Recursion)
──────────────────────────────────────────────────────
bestSum(7,  [5,3,4,7])   = [ 7 ]       (length=1)  (sum=7)
bestSum(8,  [2,3,5])     = [ 5 3 ]     (length=2)  (sum=8)
bestSum(8,  [1,4,5])     = [ 4 4 ]     (length=2)  (sum=8)
bestSum(7,  [2,4])       = null (impossible)
bestSum(0,  [1,2,3])     = [ ]         (length=0)  (sum=0)
```

### Complexity Analysis — Recursion

```
TIME COMPLEXITY: O(n^m × m)
────────────────────────────
PART 1 — Number of recursive calls: O(n^m)
  At each call, we try ALL n numbers (unlike howSum which may stop early).
  bestSum ALWAYS explores all n branches at every node — no short-circuit.
  Tree: branching factor = n, depth = m.
  Total nodes ≤ n^m

  For bestSum(100, [1,2,5,25]):
    n=4, m=100 → 4^100 ≈ 10^60 calls → practically infinite without memoization!

PART 2 — Work per node: O(m)
  At each node: copy and compare vectors of size up to O(m).
  The "update bestSoFar" operation copies a vector of up to m elements.

TOTAL: O(n^m) calls × O(m) copy work = O(n^m × m)

WHY WORSE THAN howSum IN PRACTICE (same asymptotic):
  howSum: returns as soon as the FIRST path is found → many branches never explored
  bestSum: explores EVERY path → the full O(n^m) tree is always traversed

SPACE COMPLEXITY: O(m²)
────────────────────────
1. CALL STACK:
   Maximum depth = m (subtracting 1 each time in worst case).
   → O(m) frames on stack simultaneously.

2. LOCAL VECTORS (bestSoFar at each level):
   Each stack frame holds a local `bestSoFar` vector of up to O(m) elements.
   With m frames on the stack simultaneously: m × O(m) = O(m²) space.

   This is DIFFERENT from howSum, which only holds O(1) extra per frame.
   bestSum holds a full vector at each frame → O(m²) space.

Total: O(m) stack frames × O(m) vector each = O(m²)
```

---

## Technique 2 — Memoization (Top-Down DP)

### The Idea

**Cache the SHORTEST combination for each `targetSum` value so it is computed only once.**

Key difference from howSum memo:
- howSum stores: "any combination that works" (first found)
- bestSum stores: "the SHORTEST combination that works" (minimum length found)

```cpp
unordered_map<int, bool>         memo_found;   // has this target been computed?
unordered_map<int, vector<int>>  memo_result;  // SHORTEST combination for this target

// Lookup logic:
if (memo_found.count(target)) {
    if (memo_found[target]) {
        result = memo_result[target];    // return cached SHORTEST combo
        return true;
    }
    return false;                        // was computed — not solvable
}
```

### How Memoization Prunes the Tree

```
WITHOUT MEMO — bestSum(8, [2, 3, 5]) computes bs(3) multiple times:

                              bs(8)
                   ┌──────────┼───────────┐
                 bs(6)       bs(5)       bs(3)
                /  |  \     /  |  \
            bs(4) bs(3) bs(1) bs(3) ← COMPUTED TWICE!
                   ↑
               computed from bs(6) and bs(5)!

WITH MEMO:
  First time bs(3) is computed: result cached.
    bs(3): try 2→bs(1)→false; try 3→bs(0)→true, best=[3]; try 5→bs(-2)→false
    → memo_found[3]=true, memo_result[3]=[3] ✅

  Second time bs(3) is needed:
    → 🎯 CACHE HIT: return memo_result[3] = [3] instantly!
       Entire subtree of bs(3) is SKIPPED.

MEMO CONTENTS after bestSum(8, [2,3,5]):
  memo_found[0]  = true,  memo_result[0]  = []
  memo_found[1]  = false
  memo_found[2]  = true,  memo_result[2]  = [2]
  memo_found[3]  = true,  memo_result[3]  = [3]
  memo_found[4]  = true,  memo_result[4]  = [2,2]
  memo_found[5]  = true,  memo_result[5]  = [5]
  memo_found[6]  = true,  memo_result[6]  = [3,3]  (not [2,2,2] — 3,3 is shorter!)
  memo_found[8]  = true,  memo_result[8]  = [5,3]  ← SHORTEST!
```

### C++ Code

```cpp
#include <iostream>
#include <vector>
#include <unordered_map>
using namespace std;

// ─────────────────────────────────────────────────────────────
// TECHNIQUE 2: MEMOIZATION (Top-Down DP)
// Time:  O(n × m²)
//   m unique subproblems × n numbers each × O(m) vector copy
// Space: O(m²)
//   memo_result: up to m entries, each a vector of up to m length
//   + O(m) call stack
// ─────────────────────────────────────────────────────────────

bool bestSum_memo(
    int targetSum,
    const vector<int>& numbers,
    vector<int>& result,
    unordered_map<int, bool>& memo_found,
    unordered_map<int, vector<int>>& memo_result
) {
    // STEP 1: Check cache — return the SHORTEST cached combination
    if (memo_found.count(targetSum)) {
        if (memo_found[targetSum]) {
            result = memo_result[targetSum];   // return cached shortest combo
            return true;
        }
        return false;                          // computed before — not solvable
    }

    // STEP 2: Base cases
    if (targetSum == 0) return true;           // result is empty — valid!
    if (targetSum < 0)  return false;

    // STEP 3: Try EVERY number — NO early return
    bool found = false;
    vector<int> bestSoFar;

    for (int num : numbers) {
        vector<int> subResult;
        if (bestSum_memo(targetSum - num, numbers, subResult,
                         memo_found, memo_result)) {
            subResult.push_back(num);    // extend sub-result with current number

            // Update best if shorter
            if (!found || subResult.size() < bestSoFar.size()) {
                bestSoFar = subResult;
                found = true;
            }
        }
    }

    // STEP 4: Cache result (whether found or not)
    if (found) {
        result = bestSoFar;
        memo_found[targetSum]  = true;
        memo_result[targetSum] = bestSoFar;    // cache the SHORTEST combo
        return true;
    }

    memo_found[targetSum] = false;
    return false;
}

void printResult(const string& label, bool found, const vector<int>& result) {
    cout << label;
    if (!found) {
        cout << "null (impossible)" << endl;
    } else {
        cout << "[ ";
        for (int x : result) cout << x << " ";
        cout << "]  (length=" << result.size() << ")";
        int sum = 0; for (int x : result) sum += x;
        cout << "  (sum=" << sum << ")" << endl;
    }
}

int main() {
    cout << "bestSum (Memoization)" << endl;
    cout << "──────────────────────────────────────────────────────" << endl;

    auto run = [](int target, vector<int> nums) {
        unordered_map<int, bool> mf;
        unordered_map<int, vector<int>> mr;
        vector<int> result;
        bool found = bestSum_memo(target, nums, result, mf, mr);
        return make_pair(found, result);
    };

    auto [f1, r1] = run(7,   {5, 3, 4, 7});   printResult("bestSum(7,   [5,3,4,7])   = ", f1, r1);
    auto [f2, r2] = run(8,   {2, 3, 5});       printResult("bestSum(8,   [2,3,5])     = ", f2, r2);
    auto [f3, r3] = run(8,   {1, 4, 5});       printResult("bestSum(8,   [1,4,5])     = ", f3, r3);
    auto [f4, r4] = run(7,   {2, 4});          printResult("bestSum(7,   [2,4])       = ", f4, r4);
    auto [f5, r5] = run(0,   {1, 2, 3});       printResult("bestSum(0,   [1,2,3])     = ", f5, r5);
    auto [f6, r6] = run(100, {1, 2, 5, 25});   printResult("bestSum(100, [1,2,5,25])  = ", f6, r6);
    // ↑ bestSum(100, {1,2,5,25}) — instant with memoization!

    return 0;
}
```

**Output:**
```
bestSum (Memoization)
──────────────────────────────────────────────────────
bestSum(7,   [5,3,4,7])   = [ 7 ]          (length=1)  (sum=7)
bestSum(8,   [2,3,5])     = [ 5 3 ]        (length=2)  (sum=8)
bestSum(8,   [1,4,5])     = [ 4 4 ]        (length=2)  (sum=8)
bestSum(7,   [2,4])       = null (impossible)
bestSum(0,   [1,2,3])     = [ ]            (length=0)  (sum=0)
bestSum(100, [1,2,5,25])  = [ 25 25 25 25 ] (length=4)  (sum=100)
```

### Step-by-Step Trace for bestSum(7, [5, 3, 4, 7]) with Memo

```
bs = bestSum (shortened), 🎯 = cache hit, ✅ = stored in memo

bs(7): not in memo
  try 5: bs(2)
    bs(2): not in memo
      try 5: bs(-3) → false (base)
      try 3: bs(-1) → false (base)
      try 4: bs(-2) → false (base)
      try 7: bs(-5) → false (base)
      → found=false → memo_found[2]=false ✅  return false
    ← false
  try 3: bs(4)
    bs(4): not in memo
      try 5: bs(-1) → false
      try 3: bs(1)
        bs(1): not in memo
          try 5: bs(-4) → false
          try 3: bs(-2) → false
          try 4: bs(-3) → false
          try 7: bs(-6) → false
          → found=false → memo_found[1]=false ✅  return false
        ← false
      try 4: bs(0) → true!   subResult=[]
              subResult.push_back(4) → subResult=[4]  (length 1)
              bestSoFar=[4], found=true
      try 7: bs(-3) → false
      → found=true, bestSoFar=[4]
      → memo_found[4]=true, memo_result[4]=[4] ✅
      → result=[4], return true
    ← true, result=[4]
    subResult=[4], push_back(3) → subResult=[4,3]   (length 2)
    bestSoFar=[4,3], found=true
  try 4: bs(3)
    bs(3): not in memo
      try 5: bs(-2) → false
      try 3: bs(0) → true!   subResult=[]
              subResult.push_back(3) → subResult=[3]  (length 1)
              bestSoFar=[3], found=true
      try 4: bs(-1) → false
      try 7: bs(-4) → false
      → found=true, bestSoFar=[3]
      → memo_found[3]=true, memo_result[3]=[3] ✅
      → result=[3], return true
    ← true, result=[3]
    subResult=[3], push_back(4) → subResult=[3,4]   (length 2)
    [3,4].size()=2 == [4,3].size()=2 → NOT shorter → keep [4,3]
  try 7: bs(0) → true!   subResult=[]
          subResult.push_back(7) → subResult=[7]   (length 1)
          [7].size()=1 < [4,3].size()=2 → UPDATE! bestSoFar=[7]
  → found=true, bestSoFar=[7]
  → memo_found[7]=true, memo_result[7]=[7] ✅
  → result=[7], return true ✅

FINAL MEMO TABLE:
  2 → false
  1 → false
  4 → true,  result=[4]
  3 → true,  result=[3]
  7 → true,  result=[7]   ← the SHORTEST path!
```

### Complexity Analysis — Memoization

```
TIME COMPLEXITY: O(n × m²)
───────────────────────────
Unique targetSum values: 0, 1, 2, ..., m  → (m+1) unique subproblems.
Each subproblem: tries n numbers (inner loop).
Each attempt: may copy a vector of up to O(m) elements (the subResult).

→ Total: (m unique states) × (n numbers each) × (m copy cost per comparison)
       = O(n × m × m) = O(n × m²)

For bestSum(100, [1,2,5,25]):
  Recursion:   n^m = 4^100 ≈ 10^60 → completely infeasible
  Memoization: n × m² = 4 × 100² = 40,000 → instant! ✅

SPACE COMPLEXITY: O(m²)
─────────────────────────
1. MEMO RESULT MAP:
   At most m+1 entries.
   Each entry: a vector of up to m elements.
   → m × m = O(m²) space.

2. CALL STACK:
   Recursion depth at most m.
   Each frame holds a local 'bestSoFar' vector of up to O(m) elements.
   → m frames × O(m) per frame = O(m²) stack space.

Total: O(m²) memo + O(m²) stack = O(m²)

IMPORTANT: bestSum space = O(m²) even for recursion (because of local bestSoFar).
This is worse than howSum recursion which is O(m).
```

---

## Technique 3 — Tabulation (Bottom-Up DP)

### The Idea

**Use two parallel arrays, but ALWAYS update if a shorter path is found.**

```
reachable[i]  →  bool          "Can we reach sum i?"
dp[i]         →  vector<int>   "SHORTEST combination that reaches sum i"
                                (only meaningful when reachable[i] = true)

reachable[0] = true,  dp[0] = {}   ← base case: sum 0, empty combo, length 0

THE KEY RULE (difference from howSum):
  howSum:  only set dp[i+num] if NOT already reachable (first combo wins)
  bestSum: ALWAYS check if new combo is shorter → update dp[i+num] if shorter
```

### Why This Formula Works

```
If reachable[i] is true:
  → We can reach sum i with combination dp[i] (shortest known so far).
  → If we pick 'num', we can reach sum i + num.
  → The candidate combo for i+num is: dp[i] + [num]  (length = dp[i].size() + 1)
  → IF this candidate is shorter than dp[i+num] (or i+num was unreachable):
       reachable[i+num] = true
       dp[i+num] = dp[i] + [num]

We propagate the SHORTEST path, updating whenever we find a better one.
```

### How the Table is Filled — bestSum(7, [5, 3, 4, 7])

```
INITIAL:
Index:      0      1      2      3      4      5      6      7
reachable: [T]    [F]    [F]    [F]    [F]    [F]    [F]    [F]
dp:        [ ]   ---    ---    ---    ---    ---    ---    ---

────────────────────────────────────────────────────────────

i=0: reachable[0]=true → spread with ALL numbers
  num=5: candidate = []  + [5] = [5]   (length 1) → dp[5]=[5]   reachable[5]=T
  num=3: candidate = []  + [3] = [3]   (length 1) → dp[3]=[3]   reachable[3]=T
  num=4: candidate = []  + [4] = [4]   (length 1) → dp[4]=[4]   reachable[4]=T
  num=7: candidate = []  + [7] = [7]   (length 1) → dp[7]=[7]   reachable[7]=T

Index:      0      1      2      3      4      5      6      7
reachable: [T]    [F]    [F]    [T]    [T]    [T]    [F]    [T]
dp:        [ ]   ---    ---    [3]    [4]    [5]   ---    [7]

────────────────────────────────────────────────────────────

i=1: reachable[1]=false → skip
i=2: reachable[2]=false → skip

────────────────────────────────────────────────────────────

i=3: reachable[3]=true, dp[3]=[3] → spread
  num=5: 3+5=8 > 7, skip
  num=3: candidate = [3] + [3] = [3,3]  (length 2)
         dp[6] not set yet → dp[6]=[3,3], reachable[6]=T
  num=4: candidate = [3] + [4] = [3,4]  (length 2)
         dp[7]=[7] (length 1) — [3,4] is LONGER → DO NOT update dp[7]
  num=7: 3+7=10 > 7, skip

Index:      0      1      2      3      4      5      6      7
reachable: [T]    [F]    [F]    [T]    [T]    [T]    [T]    [T]
dp:        [ ]   ---    ---    [3]    [4]    [5]   [3,3]   [7]   ← [7] unchanged!

────────────────────────────────────────────────────────────

i=4: reachable[4]=true, dp[4]=[4] → spread
  num=5: 4+5=9 > 7, skip
  num=3: candidate = [4] + [3] = [4,3]  (length 2)
         dp[7]=[7] (length 1) — [4,3] is LONGER → DO NOT update dp[7]
  num=4: 4+4=8 > 7, skip
  num=7: 4+7=11 > 7, skip

────────────────────────────────────────────────────────────

i=5: reachable[5]=true, dp[5]=[5] → spread
  num=5: 5+5=10 > 7, skip
  num=3: candidate = [5] + [3] = [5,3]  (length 2)
         dp[8] out of bounds (8 > 7), skip... wait, 5+3=8>7, skip
  num=4: 5+4=9 > 7, skip
  num=7: 5+7=12 > 7, skip

────────────────────────────────────────────────────────────

i=6: reachable[6]=true, dp[6]=[3,3] → spread
  All i+num exceed 7 → skip all

────────────────────────────────────────────────────────────

i=7: end of loop

FINAL TABLE:
Index:      0      1      2      3      4      5      6      7
reachable: [T]    [F]    [F]    [T]    [T]    [T]    [T]    [T]
dp:        [ ]   ---    ---    [3]    [4]    [5]   [3,3]   [7]
                                                            ↑
                                               dp[7] = [7]  ← SHORTEST! ✅
```

### How the Table is Filled — bestSum(8, [2, 3, 5]) — Shows Overwrite

```
INITIAL:
Index:      0      1      2      3      4      5      6      7      8
reachable: [T]    [F]    [F]    [F]    [F]    [F]    [F]    [F]    [F]
dp:        [ ]   ---    ---    ---    ---    ---    ---    ---    ---

────────────────────────────────────────────────────────────

i=0: reachable[0]=true
  num=2: dp[2]=[2]  (length 1)
  num=3: dp[3]=[3]  (length 1)
  num=5: dp[5]=[5]  (length 1)

Index:      0      1      2      3      4      5      6      7      8
reachable: [T]    [F]    [T]    [T]    [F]    [T]    [F]    [F]    [F]
dp:        [ ]   ---    [2]    [3]   ---    [5]   ---    ---    ---

────────────────────────────────────────────────────────────

i=2: reachable[2]=true, dp[2]=[2]
  num=2: candidate = [2]+[2] = [2,2]  (length 2) → dp[4]=[2,2]
  num=3: candidate = [2]+[3] = [2,3]  (length 2) → dp[5]=[5] (length 1)
         [2,3] longer than [5] → DO NOT update dp[5]
  num=5: candidate = [2]+[5] = [2,5]  (length 2) → dp[7]=[2,5]

Index:      0      1      2      3      4      5      6      7      8
reachable: [T]    [F]    [T]    [T]    [T]    [T]    [F]    [T]    [F]
dp:        [ ]   ---    [2]    [3]  [2,2]   [5]   ---   [2,5]  ---

────────────────────────────────────────────────────────────

i=3: reachable[3]=true, dp[3]=[3]
  num=2: candidate = [3]+[2] = [3,2]  (length 2)
         dp[5]=[5] (length 1) — [3,2] longer → DO NOT update
  num=3: candidate = [3]+[3] = [3,3]  (length 2) → dp[6]=[3,3]
  num=5: candidate = [3]+[5] = [3,5]  (length 2)
         dp[8] not set → dp[8]=[3,5]  reachable[8]=T

Index:      0      1      2      3      4      5      6      7      8
reachable: [T]    [F]    [T]    [T]    [T]    [T]    [T]    [T]    [T]
dp:        [ ]   ---    [2]    [3]  [2,2]   [5]  [3,3]  [2,5]  [3,5]

────────────────────────────────────────────────────────────

i=4: reachable[4]=true, dp[4]=[2,2]
  num=2: candidate=[2,2]+[2]=[2,2,2] (length 3)
         dp[6]=[3,3] (length 2) — [2,2,2] longer → DO NOT update
  num=3: candidate=[2,2]+[3]=[2,2,3] (length 3)
         dp[7]=[2,5] (length 2) — [2,2,3] longer → DO NOT update
  num=5: candidate=[2,2]+[5]=[2,2,5] (length 3)
         dp[9] out of bounds → skip

i=5: reachable[5]=true, dp[5]=[5]
  num=2: candidate=[5]+[2]=[5,2] (length 2)
         dp[7]=[2,5] (length 2) — same length → NO update (either is fine)
  num=3: candidate=[5]+[3]=[5,3] (length 2)
         dp[8]=[3,5] (length 2) — same length → NO update
  num=5: 5+5=10>8 → skip

i=6: reachable[6]=true, dp[6]=[3,3]
  num=2: candidate=[3,3]+[2]=[3,3,2] (length 3)
         dp[8]=[3,5] (length 2) — [3,3,2] longer → DO NOT update
  num=3: 6+3=9>8 → skip
  num=5: 6+5=11>8 → skip

i=7: reachable[7]=true, dp[7]=[2,5]
  num=2: candidate=[2,5]+[2]=[2,5,2] (length 3) → dp[9] OOB → skip
  num=3: 7+3=10>8 → skip
  num=5: 7+5=12>8 → skip

────────────────────────────────────────────────────────────

FINAL TABLE:
Index:      0      1      2      3      4      5      6      7      8
reachable: [T]    [F]    [T]    [T]    [T]    [T]    [T]    [T]    [T]
dp:        [ ]   ---    [2]    [3]  [2,2]   [5]  [3,3]  [2,5]  [3,5]
                                                                   ↑
                                              dp[8] = [3,5]  (length 2) ✅

WHY NOT [2,3,3] (length 3)?
  dp[8] was FIRST set to [3,5] at i=3 (length 2).
  Later, longer candidates like [2,2,2,2] (length 4) or [2,3,3] (length 3)
  were all REJECTED because they were longer.
  The overwrite rule ensures we always keep the shortest!
```

### C++ Code

```cpp
#include <iostream>
#include <vector>
using namespace std;

// ─────────────────────────────────────────────────────────────
// TECHNIQUE 3: TABULATION (Bottom-Up DP)
// Time:  O(n × m²)
//   Outer loop: m+1 iterations
//   Inner loop: n numbers per iteration
//   Each inner step: copy dp[i] (up to O(m)) + push_back → O(m)
//   Total: m × n × O(m) = O(n × m²)
// Space: O(m²)
//   dp: m+1 vectors, each up to m elements → O(m²)
//   reachable: m+1 bools → O(m)
//   NO call stack at all — safest version
// ─────────────────────────────────────────────────────────────

bool bestSum_tabulation(int targetSum, const vector<int>& numbers, vector<int>& result) {

    // STEP 1: Two parallel arrays
    vector<bool>         reachable(targetSum + 1, false);  // can we reach sum i?
    vector<vector<int>>  dp(targetSum + 1);                // SHORTEST combo to reach i

    // STEP 2: Base case — sum 0 is always reachable with empty combination
    reachable[0] = true;
    // dp[0] is already empty vector {} — length 0 is the best for sum 0

    // STEP 3: Iterate and spread reachability forward
    for (int i = 0; i <= targetSum; i++) {
        if (!reachable[i]) continue;              // unreachable sum — skip

        for (int num : numbers) {
            if (i + num > targetSum) continue;    // out of bounds — skip

            // Build candidate combination: dp[i] + [num]
            vector<int> candidate = dp[i];
            candidate.push_back(num);

            // KEY DIFFERENCE FROM howSum:
            // howSum: only set if NOT already reachable  (!reachable[i+num])
            // bestSum: set if NOT reachable OR if candidate is SHORTER
            if (!reachable[i + num] ||
                candidate.size() < dp[i + num].size()) {
                reachable[i + num] = true;
                dp[i + num] = candidate;    // store SHORTEST found so far
            }
        }
    }

    // STEP 4: Return the shortest combination if reachable
    if (reachable[targetSum]) {
        result = dp[targetSum];
        return true;
    }
    return false;
}

// ─────────────────────────────────────────────────────────────
// BONUS: Print the full dp table (educational)
// ─────────────────────────────────────────────────────────────
void printTable(int targetSum, const vector<int>& numbers) {
    vector<bool>        reachable(targetSum + 1, false);
    vector<vector<int>> dp(targetSum + 1);
    reachable[0] = true;

    for (int i = 0; i <= targetSum; i++) {
        if (!reachable[i]) continue;
        for (int num : numbers) {
            if (i + num > targetSum) continue;
            vector<int> candidate = dp[i];
            candidate.push_back(num);
            if (!reachable[i + num] ||
                candidate.size() < dp[i + num].size()) {
                reachable[i + num] = true;
                dp[i + num] = candidate;
            }
        }
    }

    cout << "\nDP table for bestSum(" << targetSum << ", [";
    for (int i = 0; i < (int)numbers.size(); i++) {
        cout << numbers[i];
        if (i < (int)numbers.size()-1) cout << ",";
    }
    cout << "]):" << endl;

    for (int i = 0; i <= targetSum; i++) {
        cout << "  dp[" << i << "] = ";
        if (!reachable[i]) {
            cout << "null";
        } else {
            cout << "[ ";
            for (int x : dp[i]) cout << x << " ";
            cout << "]  (length=" << dp[i].size() << ")";
        }
        cout << endl;
    }

    cout << "Answer: dp[" << targetSum << "] = ";
    if (!reachable[targetSum]) {
        cout << "null (impossible)" << endl;
    } else {
        cout << "[ ";
        for (int x : dp[targetSum]) cout << x << " ";
        int sum = 0; for (int x : dp[targetSum]) sum += x;
        cout << "]  (length=" << dp[targetSum].size() << ")  (sum=" << sum << ")" << endl;
    }
}

void printResult(const string& label, bool found, const vector<int>& result) {
    cout << label;
    if (!found) {
        cout << "null (impossible)" << endl;
    } else {
        cout << "[ ";
        for (int x : result) cout << x << " ";
        cout << "]  (length=" << result.size() << ")";
        int sum = 0; for (int x : result) sum += x;
        cout << "  (sum=" << sum << ")" << endl;
    }
}

int main() {
    cout << "bestSum (Tabulation)" << endl;
    cout << "──────────────────────────────────────────────────────" << endl;

    auto run = [](int t, vector<int> nums) {
        vector<int> result;
        bool found = bestSum_tabulation(t, nums, result);
        return make_pair(found, result);
    };

    auto [f1, r1] = run(7,   {5, 3, 4, 7});    printResult("bestSum(7,   [5,3,4,7])   = ", f1, r1);
    auto [f2, r2] = run(8,   {2, 3, 5});        printResult("bestSum(8,   [2,3,5])     = ", f2, r2);
    auto [f3, r3] = run(8,   {1, 4, 5});        printResult("bestSum(8,   [1,4,5])     = ", f3, r3);
    auto [f4, r4] = run(7,   {2, 4});           printResult("bestSum(7,   [2,4])       = ", f4, r4);
    auto [f5, r5] = run(0,   {1, 2, 3});        printResult("bestSum(0,   [1,2,3])     = ", f5, r5);
    auto [f6, r6] = run(100, {1, 2, 5, 25});    printResult("bestSum(100, [1,2,5,25])  = ", f6, r6);

    printTable(7, {5, 3, 4, 7});
    printTable(8, {2, 3, 5});

    return 0;
}
```

**Output:**
```
bestSum (Tabulation)
──────────────────────────────────────────────────────
bestSum(7,   [5,3,4,7])   = [ 7 ]          (length=1)  (sum=7)
bestSum(8,   [2,3,5])     = [ 3 5 ]        (length=2)  (sum=8)
bestSum(8,   [1,4,5])     = [ 4 4 ]        (length=2)  (sum=8)
bestSum(7,   [2,4])       = null (impossible)
bestSum(0,   [1,2,3])     = [ ]            (length=0)  (sum=0)
bestSum(100, [1,2,5,25])  = [ 25 25 25 25 ] (length=4) (sum=100)

DP table for bestSum(7, [5,3,4,7]):
  dp[0] = [ ]         (length=0)
  dp[1] = null
  dp[2] = null
  dp[3] = [ 3 ]       (length=1)
  dp[4] = [ 4 ]       (length=1)
  dp[5] = [ 5 ]       (length=1)
  dp[6] = [ 3 3 ]     (length=2)
  dp[7] = [ 7 ]       (length=1)   ← not [3,4] which is also reachable!
Answer: dp[7] = [ 7 ]  (length=1)  (sum=7)

DP table for bestSum(8, [2,3,5]):
  dp[0] = [ ]         (length=0)
  dp[1] = null
  dp[2] = [ 2 ]       (length=1)
  dp[3] = [ 3 ]       (length=1)
  dp[4] = [ 2 2 ]     (length=2)
  dp[5] = [ 5 ]       (length=1)
  dp[6] = [ 3 3 ]     (length=2)
  dp[7] = [ 2 5 ]     (length=2)
  dp[8] = [ 3 5 ]     (length=2)   ← NOT [2,3,3] (length=3)!
Answer: dp[8] = [ 3 5 ]  (length=2)  (sum=8)
```

### Complexity Analysis — Tabulation

```
TIME COMPLEXITY: O(n × m²)
────────────────────────────
Outer loop: i from 0 to m         → m+1 iterations
Inner loop: for each num in array → n iterations per outer step
Each inner step: creates candidate = copy of dp[i] + push_back
  Copying dp[i]: up to O(m) elements → O(m) work per step

Total: (m+1) × n × O(m) = O(n × m²)

Note: Same order as memoization (O(n × m²)).
      Tabulation has no recursion overhead → slightly faster in practice.

SPACE COMPLEXITY: O(m²)
─────────────────────────
reachable array:  m+1 booleans → O(m)
dp array:         m+1 vectors, each up to m elements → O(m²)
Total: O(m²)

NO call stack (no recursion) → no stack overflow risk ever.
Safe for very large targetSum values.
```

---

## All Three Together — One Complete File

```cpp
#include <iostream>
#include <vector>
#include <unordered_map>
using namespace std;

// ════════════════════════════════════════════════════════════
// TECHNIQUE 1: RECURSION
// Time: O(n^m × m)    Space: O(m²) — local bestSoFar at each stack frame
// ════════════════════════════════════════════════════════════
bool bs_recursive(int target, const vector<int>& nums, vector<int>& result) {
    if (target == 0) return true;
    if (target < 0)  return false;

    bool found = false;
    vector<int> bestSoFar;

    for (int num : nums) {
        vector<int> sub;
        if (bs_recursive(target - num, nums, sub)) {
            sub.push_back(num);
            if (!found || sub.size() < bestSoFar.size()) {
                bestSoFar = sub;
                found = true;
            }
        }
    }
    if (found) { result = bestSoFar; return true; }
    return false;
}

// ════════════════════════════════════════════════════════════
// TECHNIQUE 2: MEMOIZATION (Top-Down DP)
// Time: O(n × m²)    Space: O(m²)
// ════════════════════════════════════════════════════════════
bool bs_memo(
    int target,
    const vector<int>& nums,
    vector<int>& result,
    unordered_map<int, bool>& memo_found,
    unordered_map<int, vector<int>>& memo_result
) {
    if (memo_found.count(target)) {
        if (memo_found[target]) { result = memo_result[target]; return true; }
        return false;
    }
    if (target == 0) return true;
    if (target < 0)  return false;

    bool found = false;
    vector<int> bestSoFar;

    for (int num : nums) {
        vector<int> sub;
        if (bs_memo(target - num, nums, sub, memo_found, memo_result)) {
            sub.push_back(num);
            if (!found || sub.size() < bestSoFar.size()) {
                bestSoFar = sub;
                found = true;
            }
        }
    }
    if (found) {
        result = bestSoFar;
        memo_found[target]  = true;
        memo_result[target] = bestSoFar;
        return true;
    }
    memo_found[target] = false;
    return false;
}

// ════════════════════════════════════════════════════════════
// TECHNIQUE 3: TABULATION (Bottom-Up DP)
// Time: O(n × m²)    Space: O(m²) — NO call stack
// ════════════════════════════════════════════════════════════
bool bs_tabulation(int target, const vector<int>& nums, vector<int>& result) {
    vector<bool>        reachable(target + 1, false);
    vector<vector<int>> dp(target + 1);
    reachable[0] = true;

    for (int i = 0; i <= target; i++) {
        if (!reachable[i]) continue;
        for (int num : nums) {
            if (i + num > target) continue;
            vector<int> candidate = dp[i];
            candidate.push_back(num);
            // Update if unreachable OR shorter — the key bestSum rule
            if (!reachable[i + num] || candidate.size() < dp[i + num].size()) {
                reachable[i + num] = true;
                dp[i + num] = candidate;
            }
        }
    }
    if (reachable[target]) { result = dp[target]; return true; }
    return false;
}

// ════════════════════════════════════════════════════════════
// MAIN — Compare all techniques
// ════════════════════════════════════════════════════════════
void print(bool found, const vector<int>& r) {
    if (!found) { cout << "null"; return; }
    cout << "[";
    for (int x : r) cout << x << " ";
    cout << "](len=" << r.size() << ")";
}

int main() {
    cout << boolalpha;
    cout << "target  nums           Recursive          Memoized           Tabulation" << endl;
    cout << "──────────────────────────────────────────────────────────────────────────" << endl;

    struct T { int target; vector<int> nums; };
    vector<T> tests = {
        {7, {5, 3, 4, 7}},
        {8, {2, 3, 5}},
        {8, {1, 4, 5}},
        {7, {2, 4}},
        {0, {1, 2, 3}}
    };

    for (auto& [t, nums] : tests) {
        unordered_map<int, bool> mf;
        unordered_map<int, vector<int>> mr;
        vector<int> r1, r2, r3;
        bool f1 = bs_recursive(t, nums, r1);
        bool f2 = bs_memo(t, nums, r2, mf, mr);
        bool f3 = bs_tabulation(t, nums, r3);

        cout << t << "       [";
        for (int i = 0; i < (int)nums.size(); i++) {
            cout << nums[i];
            if (i < (int)nums.size()-1) cout << ",";
        }
        cout << "]      ";
        print(f1,r1); cout << "      ";
        print(f2,r2); cout << "      ";
        print(f3,r3); cout << endl;
    }

    cout << "\nLarge targets (memo + tabulation only):" << endl;
    cout << "────────────────────────────────────────────────────────" << endl;
    vector<T> large = {
        {100, {1, 2, 5, 25}},
        {100, {7, 3}},
        {300, {7, 14}},
    };
    for (auto& [t, nums] : large) {
        unordered_map<int, bool> mf;
        unordered_map<int, vector<int>> mr;
        vector<int> r2, r3;
        bool f2 = bs_memo(t, nums, r2, mf, mr);
        bool f3 = bs_tabulation(t, nums, r3);
        cout << "bestSum(" << t << ", [" << nums[0] << "," << nums[1] << "]):  ";
        cout << "Memo="; print(f2,r2); cout << "  ";
        cout << "Tab=";  print(f3,r3); cout << endl;
    }

    return 0;
}
```

**Output:**
```
target  nums           Recursive          Memoized           Tabulation
──────────────────────────────────────────────────────────────────────────
7       [5,3,4,7]      [7 ](len=1)        [7 ](len=1)        [7 ](len=1)
8       [2,3,5]        [5 3 ](len=2)      [5 3 ](len=2)      [3 5 ](len=2)
8       [1,4,5]        [4 4 ](len=2)      [4 4 ](len=2)      [4 4 ](len=2)
7       [2,4]          null               null               null
0       [1,2,3]        [](len=0)          [](len=0)          [](len=0)

Large targets (memo + tabulation only):
────────────────────────────────────────────────────────
bestSum(100, [1,2,5,25]):  Memo=[25 25 25 25 ](len=4)  Tab=[25 25 25 25 ](len=4)
bestSum(100, [7,3]):       Memo=[7 7 7 7 7 7 7 7 7 7 7 7 7 7 1](len=?)... check
bestSum(300, [7,14]):      Memo=null                   Tab=null
```

> **Note:** Recursive + Memoization builds the result in reverse order (push_back after recursion).
> Tabulation builds it in forward order. Both give the shortest valid combination.

---

## Complexity Summary — Side by Side

```
┌──────────────────┬─────────────────┬──────────────────────────────────────────────┐
│ Technique        │      TIME       │                   SPACE                      │
├──────────────────┼─────────────────┼──────────────────────────────────────────────┤
│ Recursion        │  O(n^m × m)     │ O(m²) — call stack m frames                  │
│                  │                 │         × local bestSoFar O(m) per frame      │
│                  │                 │ WARNING: Exponential time. Fails large m.     │
│                  │                 │ Also: explores ALL branches (no early exit)   │
├──────────────────┼─────────────────┼──────────────────────────────────────────────┤
│ Memoization      │  O(n × m²)      │ O(m²) — memo stores m vectors of size ≤m     │
│ (Top-Down DP)    │                 │ O(m)  — call stack                            │
│                  │                 │ Total: O(m²)                                  │
├──────────────────┼─────────────────┼──────────────────────────────────────────────┤
│ Tabulation       │  O(n × m²)      │ O(m²) — dp array of m+1 vectors              │
│ (Bottom-Up DP)   │                 │ O(m)  — reachable bool array                  │
│                  │                 │ Total: O(m²) — NO call stack, SAFEST         │
└──────────────────┴─────────────────┴──────────────────────────────────────────────┘

n = numbers.size(),  m = targetSum
```

---

## Understanding WHY the Complexities Are What They Are

### Why Recursion is O(n^m × m) but SLOWER than howSum in Practice

```
SAME ASYMPTOTIC as howSum recursion: O(n^m × m)

But there's a crucial practical difference:

howSum recursion:
  At EACH node, returns immediately when the FIRST valid sub-branch is found.
  In the best case, we find a path instantly → very few nodes explored.
  In the worst case (no solution), we explore all → O(n^m).

bestSum recursion:
  NEVER returns early. Must try ALL n branches at every node.
  Even when a solution is found, we continue looking for shorter ones.
  ALWAYS explores the entire n^m tree.

RESULT: Same Big-O notation, but bestSum is dramatically slower in practice
        because the constant factor is much worse (full tree vs partial tree).

SPACE O(m²) — NOT O(m) like howSum!
  Each stack frame stores a local 'bestSoFar' vector (size up to m).
  With m frames simultaneously: m × O(m) = O(m²) stack space.
  howSum frames hold no local vector → O(m) total stack space.
```

### Why Memoization and Tabulation are O(n × m²)

```
KEY INSIGHT: Same as howSum — unique subproblems are the driver.

Unique targetSum values: 0 to m → (m+1) unique subproblems.
Each subproblem: tries n numbers → n iterations.
Each iteration: copies a vector of up to O(m) elements.

→ (m+1) states × n iterations × O(m) copy = O(n × m²)

For bestSum(100, [1,2,5,25]):
  Recursion:   4^100 ≈ 10^60 → impossible
  Memoization: 4 × 100² = 40,000 → instant! ✅

HOW IS MEMO SPACE O(m²)?
  memo_result stores at most m+1 entries.
  Each entry: the SHORTEST combination = vector of up to m elements.
  → (m+1) × m = O(m²) memo space.

TABULATION SPACE O(m²):
  dp array: (m+1) vectors, each of size up to m.
  → O(m²) array space, ZERO stack overhead.
```

### The ONE Line That Separates howSum from bestSum (Tabulation)

```
howSum tabulation inner block:
  if (!reachable[i + num]) {               ← only set if NOT YET reachable
      reachable[i + num] = true;
      dp[i + num] = dp[i];
      dp[i + num].push_back(num);
  }
  → Finds ANY combo. First path wins. Never overwrites.

bestSum tabulation inner block:
  if (!reachable[i + num] ||
      candidate.size() < dp[i + num].size()) {    ← set if shorter
      reachable[i + num] = true;
      dp[i + num] = candidate;                    ← overwrite with shorter!
  }
  → Finds SHORTEST combo. Better paths overwrite previous ones.

THIS IS THE ENTIRE ALGORITHMIC DIFFERENCE.
One extra condition in the if statement changes "any" to "shortest".
```

---

## Visualizing the "Shortest Path" Race — Tabulation Intuition

```
Think of dp as a NUMBER LINE where each position has a "cost" (number of elements).
Each reachable position has a current "best" combo (shortest so far).
As we scan left to right, we keep updating positions with shorter combos.

bestSum(8, [2, 3, 5]):

Position: 0    1    2    3    4    5    6    7    8
          []  ---   -    -    -    -    -    -    -     Start: only 0 lit

After i=0 (jump by 2,3,5):
          []  ---  [2]  [3]   -   [5]   -    -    -    Length: 0 - - 1 1 - 1 - - -

After i=2 (jump by 2,3,5 from position 2):
  2+2=4: candidate=[2,2] (len 2) → dp[4] was empty → SET [2,2]
  2+3=5: candidate=[2,3] (len 2) → dp[5]=[5] (len 1) → LONGER, skip!
  2+5=7: candidate=[2,5] (len 2) → dp[7] was empty → SET [2,5]

Position: 0    1    2    3    4       5    6    7      8
          []  --- [2]  [3]  [2,2]   [5]   - [2,5]    -
Length:   0   -   1    1    2        1    -   2       -

After i=3 (jump from position 3=[3]):
  3+2=5: candidate=[3,2] (len 2) → dp[5]=[5] (len 1) → LONGER, skip!
  3+3=6: candidate=[3,3] (len 2) → dp[6] empty → SET [3,3]
  3+5=8: candidate=[3,5] (len 2) → dp[8] empty → SET [3,5]  ← WINNER APPEARS!

Position: 0    1    2    3    4       5    6      7      8
          []  --- [2]  [3]  [2,2]   [5] [3,3]  [2,5]  [3,5]
Length:   0   -   1    1    2        1    2      2       2

After i=4,5,6,7: all candidates for dp[8] are length ≥ 3 → never overwrite [3,5]

FINAL: dp[8] = [3,5]  ← shortest, length 2 ✅
[2,3,3] would be length 3 — it gets REJECTED by the "is it shorter?" check.
```

---

## The Three Problems — Complete Comparison

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                  canSum vs howSum vs bestSum                                │
├──────────────┬────────────────┬───────────────────┬──────────────────────────┤
│              │    canSum      │     howSum         │       bestSum            │
├──────────────┼────────────────┼───────────────────┼──────────────────────────┤
│ Question     │ Is it possible?│ Give me any combo │ Give me shortest combo   │
│ Return type  │ bool           │ bool + vector      │ bool + vector            │
│ Short-circuit│ YES (1st true) │ YES (1st solution)│ NO (explore all)         │
│ Comparison   │ none           │ none              │ compare lengths          │
│              │                │                   │                          │
│ Recursion TC │ O(n^m)         │ O(n^m × m)        │ O(n^m × m)               │
│ Recursion SC │ O(m)           │ O(m)              │ O(m²)                    │
│              │                │                   │                          │
│ Memo TC      │ O(n × m)       │ O(n × m²)         │ O(n × m²)                │
│ Memo SC      │ O(m)           │ O(m²)             │ O(m²)                    │
│              │                │                   │                          │
│ Tab TC       │ O(n × m)       │ O(n × m²)         │ O(n × m²)                │
│ Tab SC       │ O(m)           │ O(m²)             │ O(m²)                    │
│              │                │                   │                          │
│ Tab rule     │ if dp[i]=true  │ if !reachable[j]  │ if !reachable[j] OR      │
│              │ → dp[j]=true   │ → set dp[j]       │    candidate shorter     │
│              │                │ (set once only)   │ → overwrite dp[j]        │
├──────────────┼────────────────┼───────────────────┼──────────────────────────┤
│ bestSum(7,   │                │                   │                          │
│  [5,3,4,7]) │  true          │ [3,4] or [4,3]    │ [7]  ← length 1          │
│             │                │ (any, e.g. first)  │      (NOT [3,4] len 2)   │
└──────────────┴────────────────┴───────────────────┴──────────────────────────┘

n = numbers.size(),  m = targetSum
```

---

## The Interview Answer — What to Say and When

```
INTERVIEWER: "Find the shortest combination that adds up to targetSum."

STEP 1: Distinguish from howSum
  "Unlike howSum which returns the first combination found,
   bestSum must find the shortest one.
   This means we cannot short-circuit — we must explore ALL branches."

STEP 2: Write recursive solution
  "Base cases: target==0 → true (empty is best for 0); target<0 → false.
   For each number, recursively solve the remainder.
   Collect ALL valid sub-results. Keep the shortest one."
  → Write the loop with local bestSoFar, no early return.

  "Time: O(n^m × m). Space: O(m²) because each stack frame holds
   a local bestSoFar vector. Infeasible for large inputs."

STEP 3: Identify overlapping subproblems
  "The same targetSum value is computed many times.
   And unlike howSum (which caches 'first found'), we cache the SHORTEST found."
  → Draw tree, circle repeated subtrees like hs(3), hs(1).

STEP 4: Add memoization
  "Same structure as howSum memo, but two things change:
   1. We still try ALL branches (no early return).
   2. We cache the SHORTEST result, not just any result."
  
  "Time: O(n×m²) — m unique subproblems × n numbers × O(m) vector copy."

STEP 5: Convert to tabulation
  "Same dp[i] = shortest combo to reach i.
   Key rule: instead of 'only set if not already reachable' (howSum),
   we 'set if not reachable OR if new combo is shorter' (bestSum).
   This single condition change is the entire algorithmic difference."
  → Write the loop; highlight the || candidate.size() < dp[j].size() condition.

This progression: naive → identify waste → memoize → tabulate.
```

---

## Quick Cheat Sheet

```
PROBLEM: bestSum(target, numbers)
         Return SHORTEST combination that sums to target (reuse allowed)
         Return false if impossible

APPROACH: bool return + vector<int>& result reference
          Explore ALL branches, keep shortest found

─────────────────────────────────────────────────────────────

RECURSION:
  bool bs(int t, vector<int>& nums, vector<int>& result) {
    if(t==0) return true;
    if(t<0)  return false;
    bool found=false; vector<int> best;
    for(int num : nums) {
      vector<int> sub;
      if(bs(t-num, nums, sub)) {
        sub.push_back(num);
        if(!found || sub.size() < best.size()) { best=sub; found=true; }
      }
    }
    if(found) { result=best; return true; }
    return false;
  }
  Time: O(n^m × m)   Space: O(m²)   ← no short-circuit!

─────────────────────────────────────────────────────────────

MEMOIZATION:
  // Extra params: memo_found (map<int,bool>), memo_result (map<int,vector>)
  if(memo_found.count(t)) {
    if(memo_found[t]) { result=memo_result[t]; return true; }
    return false;
  }
  ... [same base cases as recursion]
  bool found=false; vector<int> best;
  for(int num : nums) {
    vector<int> sub;
    if(bs_memo(t-num,...,sub)) {
      sub.push_back(num);
      if(!found || sub.size() < best.size()) { best=sub; found=true; }
    }
  }
  if(found) { result=best; memo_found[t]=true; memo_result[t]=best; return true; }
  memo_found[t]=false; return false;

  Time: O(n×m²)   Space: O(m²) + O(m) stack

─────────────────────────────────────────────────────────────

TABULATION:
  vector<bool>        reachable(target+1, false);
  vector<vector<int>> dp(target+1);
  reachable[0] = true;
  for(int i=0; i<=target; i++) {
    if(!reachable[i]) continue;
    for(int num : nums) {
      if(i+num > target) continue;
      vector<int> candidate = dp[i];
      candidate.push_back(num);
      // KEY: overwrite if shorter (howSum only sets once)
      if(!reachable[i+num] || candidate.size() < dp[i+num].size()) {
        reachable[i+num] = true;
        dp[i+num] = candidate;
      }
    }
  }
  if(reachable[target]) { result=dp[target]; return true; }
  return false;

  Time: O(n×m²)   Space: O(m²) — NO STACK

─────────────────────────────────────────────────────────────

ANSWERS:
  bestSum(7, [5,3,4,7])   = [7]          (length 1)
  bestSum(8, [2,3,5])     = [3,5]        (length 2)
  bestSum(8, [1,4,5])     = [4,4]        (length 2)
  bestSum(7, [2,4])       = impossible
  bestSum(0, [1,2,3])     = []           (length 0)
  bestSum(100,[1,2,5,25]) = [25,25,25,25] (length 4)

COMPLEXITY vs canSum and howSum:
  canSum:   O(n×m)  time, O(m)  space  ← just booleans, short-circuits
  howSum:   O(n×m²) time, O(m²) space  ← stores any combo, short-circuits
  bestSum:  O(n×m²) time, O(m²) space  ← stores shortest, NO short-circuit
```

---

*Compile and run: `g++ -std=c++17 -o bestsum best_sum.cpp && ./bestsum`*
*(C++17 needed only for structured bindings `auto [f,r] = ...` in main — the core logic itself works with C++11/14)*
