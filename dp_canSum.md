# canSum Problem — Three Techniques
## Recursion vs Memoization vs Tabulation in C++

> **Problem:** Given a `targetSum` (integer) and an array of `numbers` (non-negative integers),
> return `true` if it is possible to generate the `targetSum` using numbers from the array.
> You may use the same number from the array **as many times as needed**.
> All numbers are non-negative.

---

## Quick Comparison (Read This First)

```
TECHNIQUE       TIME          SPACE         STYLE       BEST FOR
─────────────────────────────────────────────────────────────────────
Recursion       O(n^m)        O(m)          Top-down    Understanding the problem
Memoization     O(n * m)      O(m)          Top-down    Recursive + cache
Tabulation      O(n * m)      O(m)          Bottom-up   Most DP problems

Where: m = targetSum,  n = length of numbers array
```

---

## The Problem Visualized

```
CAN WE MAKE targetSum = 7 from numbers = [2, 3]?

Try all combinations (repetition allowed):
  2          → 2
  2+2        → 4
  2+2+2      → 6
  2+2+2+2    → 8   ✗ (overshot)
  2+2+3      → 7   ✅ YES!

Other valid combos: 3+2+2=7, 3+4... (4 not in array)

CAN WE MAKE targetSum = 7 from numbers = [2, 4]?
  2          → 2
  2+2        → 4
  2+2+2      → 6
  2+2+2+2    → 8   ✗
  4+4        → 8   ✗
  All sums of 2s and 4s are EVEN. 7 is ODD. → false ✗

CAN WE MAKE targetSum = 7 from numbers = [5, 3, 4]?
  3+4 = 7   ✅ YES!
```

**Test Cases for Reference:**

```
canSum(7,  [2, 3])       = true   (2+2+3 = 7)
canSum(7,  [5, 3, 4])    = true   (3+4 = 7)
canSum(7,  [2, 4])       = false  (all combos are even, can't make odd 7)
canSum(8,  [2, 3, 5])    = true   (3+5 = 8)
canSum(300,[7, 14])      = false  (7 and 14 both divisible by 7, 300 is not)
canSum(0,  [1, 2, 3])    = true   (choose nothing — sum is already 0)
```

---

## The Recursive Formula

```
canSum(targetSum, numbers):

  BASE CASE 1: targetSum == 0  → return true
               (We've whittled the target down to 0 — valid combination found!)

  BASE CASE 2: targetSum < 0  → return false
               (We overshot — this path is invalid)

  FOR EACH num in numbers:
    remainder = targetSum - num
    IF canSum(remainder, numbers) == true:
      return true          ← this branch works, no need to try others

  return false             ← no number worked from here

INTUITION:
  "Can I make 7 from [2,3]?"
  →  "If I pick 2, can I make 5 from [2,3]?"
       → "If I pick 2, can I make 3 from [2,3]?"
            → "If I pick 3, can I make 0 from [2,3]?"
                 → YES! (targetSum == 0)  ← BASE CASE
```

---

## Technique 1 — Recursion (Naive)

### The Idea

For each call, try subtracting every number in the array.
If any resulting remainder leads to 0, return true.
If all choices lead to negative or dead ends, return false.

### Recursion Tree for canSum(7, [5, 3, 4])

```
                        canSum(7)
                       /    |    \
             canSum(2)  canSum(4)  canSum(3)
             [7-5]      [7-3]      [7-4]
            / | \       / | \      / | \
          (-3)(-1)(-2) (-1)(1)(0) (-2)(0)(-1)
           ✗   ✗   ✗   ✗  ...  ✅  ✗  ✅
                           |
                        canSum(0) = TRUE ← 7-3-4 = 0 ✅

Shortcut: as soon as one branch returns true, we stop.
The right branch canSum(3) found 3-3=0 → true → propagates up.
```

### Recursion Tree for canSum(7, [2, 3]) — Shows Repeated Work

```
                            canSum(7)
                           /         \
                     canSum(5)      canSum(4)
                    /       \      /       \
              canSum(3)   canSum(2) canSum(2) canSum(1)
              /     \     /    \    /    \    /    \
          canSum(1)  (0) (-1) (0) (-1)  (0) (-1) (-2)
            ✅  ✅   ✅   ✗   ✅   ✗   ✅   ✗   ✗    ✗

REPEATED SUBPROBLEMS:
  canSum(4) computed twice   ← at canSum(7) and canSum(5) branches
  canSum(3) computed twice   ← multiple paths
  canSum(2) computed THREE times!
  canSum(1) computed multiple times

Each unique targetSum should only need to be computed ONCE.
```

### C++ Code

```cpp
#include <iostream>
#include <vector>
using namespace std;

// ─────────────────────────────────────────────────────────────
// TECHNIQUE 1: RECURSION (Naive)
// Time:  O(n^m)   where n = numbers.size(), m = targetSum
// Space: O(m)     call stack depth
// ─────────────────────────────────────────────────────────────

bool canSum_recursive(int targetSum, const vector<int>& numbers) {

    // BASE CASE 1: reached exactly 0 — valid combination found
    if (targetSum == 0) return true;

    // BASE CASE 2: went below 0 — this path is invalid
    if (targetSum < 0) return false;

    // Try subtracting each number
    for (int num : numbers) {
        int remainder = targetSum - num;
        if (canSum_recursive(remainder, numbers)) {
            return true;     // found a valid path — stop searching
        }
    }

    // No number led to a solution from here
    return false;
}

int main() {
    cout << "canSum (Recursion)" << endl;
    cout << "──────────────────────────────────────────────────" << endl;

    cout << boolalpha;    // print true/false instead of 1/0

    cout << "canSum(7,  [2,3])     = " << canSum_recursive(7,  {2, 3})       << endl;
    cout << "canSum(7,  [5,3,4])   = " << canSum_recursive(7,  {5, 3, 4})    << endl;
    cout << "canSum(7,  [2,4])     = " << canSum_recursive(7,  {2, 4})       << endl;
    cout << "canSum(8,  [2,3,5])   = " << canSum_recursive(8,  {2, 3, 5})    << endl;
    cout << "canSum(0,  [1,2,3])   = " << canSum_recursive(0,  {1, 2, 3})    << endl;

    // WARNING: canSum(300, [7,14]) with pure recursion is VERY SLOW
    // It explores 2^300 / similar branches — do not run naive recursion on this!

    return 0;
}
```

**Output:**
```
canSum (Recursion)
──────────────────────────────────────────────────
canSum(7,  [2,3])     = true
canSum(7,  [5,3,4])   = true
canSum(7,  [2,4])     = false
canSum(8,  [2,3,5])   = true
canSum(0,  [1,2,3])   = true
```

### Complexity Analysis — Recursion

```
TIME COMPLEXITY: O(n^m)
────────────────────────
m = targetSum, n = numbers.size()

At each recursive call, we branch into n possibilities (one per number).
The maximum depth of the recursion tree is m:
  Worst case: numbers = [1]
  canSum(m) → canSum(m-1) → canSum(m-2) → ... → canSum(0)
  But at each level we try n numbers.

It's like a tree with branching factor n and depth m:
  Total nodes = n^m

For canSum(300, [7, 14]):
  n=2, m=300 → 2^300 ≈ 10^90 calls — effectively infinite!

SPACE COMPLEXITY: O(m)
────────────────────────
The call stack depth = the longest chain of calls.
In the worst case (subtracting 1 each time), the stack grows to depth m.
→ At most m frames on the call stack at once → O(m) space.

Call stack for canSum(7, [2,3]) at deepest:
  [canSum(7)]
  [canSum(5)]     ← 7-2
  [canSum(3)]     ← 5-2
  [canSum(1)]     ← 3-2
  [canSum(-1)]    ← base case: return false
  ← unwind and try next branch
```

---

## Technique 2 — Memoization (Top-Down DP)

### The Idea

**Same recursion, but CACHE the result for each `targetSum` value.**

Key insight: `canSum(5, [2,3])` will always give the same answer no matter how we arrived at 5.
So once computed, store it — never recompute it.

Use a `map<int, bool>` or a `vector<int>` with a "visited" flag as the cache.

### How Memoization Prunes the Tree

```
WITHOUT MEMO:
  canSum(7, [2,3]) computes canSum(3) multiple times:
    canSum(7)
    ├── canSum(5)
    │   ├── canSum(3) ← computed here
    │   └── canSum(2)
    └── canSum(4)
        ├── canSum(2)
        └── canSum(1)

Wait — what if canSum(3) was already computed?
  Both canSum(5) and canSum(4) eventually reach canSum(3).
  Without memo: computed twice.
  With memo: computed ONCE, result (true) returned immediately.

WITH MEMO:
  canSum(7, [2,3]):
    canSum(5):
      canSum(3):
        canSum(1):
          canSum(-1): return false
          canSum(-2): return false
          → return false  ← memo[1] = false ✅ STORED
        canSum(0): return true ← BASE CASE
        → return true   ← memo[3] = true ✅ STORED
      ← memo[3] = true → CACHE HIT → return true immediately
    → return true   ← memo[5] = true ✅ STORED
  canSum(4): memo[4] not set yet, compute:
    canSum(2): ... (computed, memo[2] = true)
    → return true   ← memo[4] = true ✅ STORED
  → return true   ← memo[7] = true ✅ STORED

Each unique targetSum value computed EXACTLY ONCE.
```

### C++ Code

```cpp
#include <iostream>
#include <vector>
#include <map>
using namespace std;

// ─────────────────────────────────────────────────────────────
// TECHNIQUE 2: MEMOIZATION (Top-Down DP)
// Time:  O(n * m)   — each targetSum computed once, tries n numbers
// Space: O(m)       — memo map stores m entries + O(m) call stack
// ─────────────────────────────────────────────────────────────

bool canSum_memo(int targetSum, const vector<int>& numbers, map<int, bool>& memo) {

    // STEP 1: Check cache — if this targetSum was already solved, return it
    if (memo.count(targetSum)) {
        return memo[targetSum];    // CACHE HIT — instant return
    }

    // STEP 2: Base cases
    if (targetSum == 0) return true;
    if (targetSum < 0)  return false;

    // STEP 3: Try each number
    for (int num : numbers) {
        int remainder = targetSum - num;
        if (canSum_memo(remainder, numbers, memo)) {
            memo[targetSum] = true;    // cache before returning
            return true;
        }
    }

    // STEP 4: No number worked — cache false and return
    memo[targetSum] = false;
    return false;
}

int main() {
    cout << "canSum (Memoization)" << endl;
    cout << "──────────────────────────────────────────────────" << endl;

    cout << boolalpha;

    map<int, bool> memo;

    cout << "canSum(7,   [2,3])    = " << canSum_memo(7,   {2, 3},    memo) << endl;  memo.clear();
    cout << "canSum(7,   [5,3,4])  = " << canSum_memo(7,   {5, 3, 4}, memo) << endl;  memo.clear();
    cout << "canSum(7,   [2,4])    = " << canSum_memo(7,   {2, 4},    memo) << endl;  memo.clear();
    cout << "canSum(8,   [2,3,5])  = " << canSum_memo(8,   {2, 3, 5}, memo) << endl;  memo.clear();
    cout << "canSum(0,   [1,2,3])  = " << canSum_memo(0,   {1, 2, 3}, memo) << endl;  memo.clear();
    cout << "canSum(300, [7,14])   = " << canSum_memo(300, {7, 14},   memo) << endl;  memo.clear();
    // ↑ This finishes instantly! (impossible without memoization)

    return 0;
}
```

**Output:**
```
canSum (Memoization)
──────────────────────────────────────────────────
canSum(7,   [2,3])    = true
canSum(7,   [5,3,4])  = true
canSum(7,   [2,4])    = false
canSum(8,   [2,3,5])  = true
canSum(0,   [1,2,3])  = true
canSum(300, [7,14])   = false
```

### Step-by-Step Trace for canSum(7, [2, 3]) with Memo

```
Calls made (→ = "calls", ✅=stored, 🎯=cache hit):

canSum(7):  not in memo
  try 2: canSum(5)
    canSum(5): not in memo
      try 2: canSum(3)
        canSum(3): not in memo
          try 2: canSum(1)
            canSum(1): not in memo
              try 2: canSum(-1) → return false (base)
              try 3: canSum(-2) → return false (base)
              all failed → memo[1] = false ✅ return false
          try 3: canSum(0) → return true (base case!)
          memo[3] = true ✅ return true
        ← RETURNED TRUE → memo[5] = true ✅ return true immediately
      ← RETURNED TRUE → memo[7] = true ✅ return true immediately

Final memo table:
  1 → false
  3 → true
  5 → true
  7 → true

Total unique canSum() calls: 5    (was many more without memo)
Cache hits: 0 in this trace (small example — hits matter more for larger inputs)

For canSum(7, [2,4]) — false case, memo fills up:
  canSum(7): try 2→canSum(5): try 2→canSum(3): try 2→canSum(1): try 2→canSum(-1)=false
                                                                  try 4→canSum(-3)=false
                                                  memo[1]=false
                                                try 4→canSum(-1)=false
                                  memo[3]=false
                              try 4→canSum(1) 🎯 CACHE HIT → false
             memo[5]=false
         try 4→canSum(3) 🎯 CACHE HIT → false
  memo[7]=false
  try 4→canSum(5) 🎯 CACHE HIT → false
  return false

CACHE HITS save us from re-exploring entire subtrees.
```

### Complexity Analysis — Memoization

```
TIME COMPLEXITY: O(n * m)
──────────────────────────
Unique values of targetSum: 0, 1, 2, ..., m  → (m+1) values
Each targetSum is computed exactly ONCE.
For each targetSum, we try n numbers (loop over array).
→ Total work = (m+1) × n = O(n * m)

Compare:
  Recursion:   O(n^m) → billions of calls for canSum(300, [7,14])
  Memoization: O(n*m) = O(2*300) = 600 operations  → instant!

SPACE COMPLEXITY: O(m)
────────────────────────
TWO sources:

1. MEMO MAP:
   Stores one entry per unique targetSum.
   targetSum ranges from 0 to m → at most m+1 entries.
   → O(m) space.

2. CALL STACK:
   Recursion can go m deep (subtracting 1 each time).
   → O(m) stack space.

Total: O(m) + O(m) = O(m)
Much better than O(n^m) time of plain recursion!
```

---

## Technique 3 — Tabulation (Bottom-Up DP)

### The Idea

**Build a boolean array `dp` of size `m+1` from the bottom up.**

- `dp[i]` = "Can I generate sum `i` using numbers from the array?"
- Start with `dp[0] = true` (sum 0 is always achievable — pick nothing)
- For each index where `dp[i] = true`, mark `dp[i + num] = true` for every num in array
- The answer is `dp[targetSum]`

### Why This Formula Works

```
dp[0] = true  → we can make sum 0 (trivially)

If dp[i] = true, it means we CAN make sum i.
  → So from sum i, if we pick 'num', we can make sum i+num.
  → Therefore dp[i + num] = true.

We push reachability FORWARD (from smaller sums to larger sums).
Each reachable sum "infects" further sums as reachable.
```

### How the Table is Filled for canSum(7, [2, 3])

```
INITIAL TABLE: all false, except dp[0] = true

Index:  0     1     2     3     4     5     6     7
dp:    [T]   [F]   [F]   [F]   [F]   [F]   [F]   [F]
        ↑
    Start: can make 0

──────────────────────────────────────────────────────

i=0: dp[0] = true → spread reachability
  num=2: dp[0+2] = dp[2] = true  ← can reach 2 by picking 2 from 0
  num=3: dp[0+3] = dp[3] = true  ← can reach 3 by picking 3 from 0

Index:  0     1     2     3     4     5     6     7
dp:    [T]   [F]  [T]   [T]   [F]   [F]   [F]   [F]

──────────────────────────────────────────────────────

i=1: dp[1] = false → skip (can't make 1, so can't extend from here)

──────────────────────────────────────────────────────

i=2: dp[2] = true → spread
  num=2: dp[2+2] = dp[4] = true
  num=3: dp[2+3] = dp[5] = true

Index:  0     1     2     3     4     5     6     7
dp:    [T]   [F]  [T]   [T]   [T]   [T]   [F]   [F]

──────────────────────────────────────────────────────

i=3: dp[3] = true → spread
  num=2: dp[3+2] = dp[5] = true  (already true)
  num=3: dp[3+3] = dp[6] = true

Index:  0     1     2     3     4     5     6     7
dp:    [T]   [F]  [T]   [T]   [T]   [T]   [T]   [F]

──────────────────────────────────────────────────────

i=4: dp[4] = true → spread
  num=2: dp[4+2] = dp[6] = true  (already true)
  num=3: dp[4+3] = dp[7] = true  ← TARGET REACHED!

Index:  0     1     2     3     4     5     6     7
dp:    [T]   [F]  [T]   [T]   [T]   [T]   [T]  [T]
                                                  ↑
                                          Answer: dp[7] = true ✅

──────────────────────────────────────────────────────

i=5: dp[5]=true: dp[7]=true (already), dp[8] out of bounds
i=6: dp[6]=true: dp[8] out of bounds, dp[9] out of bounds
i=7: done → return dp[7] = true ✅
```

### Table for canSum(7, [2, 4]) — The FALSE Case

```
INITIAL:
Index:  0     1     2     3     4     5     6     7
dp:    [T]   [F]   [F]   [F]   [F]   [F]   [F]   [F]

i=0: dp[2]=true, dp[4]=true
Index:  0     1     2     3     4     5     6     7
dp:    [T]   [F]  [T]   [F]   [T]   [F]   [F]   [F]

i=2: dp[4]=true(already), dp[6]=true
Index:  0     1     2     3     4     5     6     7
dp:    [T]   [F]  [T]   [F]   [T]   [F]   [T]   [F]

i=4: dp[6]=true(already), dp[8] OOB

i=6: dp[8] OOB, dp[10] OOB

FINAL:
Index:  0     1     2     3     4     5     6     7
dp:    [T]   [F]  [T]   [F]   [T]   [F]   [T]  [F]
                                                  ↑
                                         dp[7] = false ✗

OBSERVATION: Only even indices are true (2+2+2...) — 7 is odd, unreachable!
```

### C++ Code

```cpp
#include <iostream>
#include <vector>
using namespace std;

// ─────────────────────────────────────────────────────────────
// TECHNIQUE 3: TABULATION (Bottom-Up DP)
// Time:  O(n * m)
// Space: O(m)    — just the dp array, NO call stack
// ─────────────────────────────────────────────────────────────

bool canSum_tabulation(int targetSum, const vector<int>& numbers) {

    // STEP 1: Create dp array of size (targetSum+1), all false
    vector<bool> dp(targetSum + 1, false);

    // STEP 2: Base case — sum 0 is always reachable
    dp[0] = true;

    // STEP 3: For every reachable sum, mark sums reachable from it
    for (int i = 0; i <= targetSum; i++) {
        if (dp[i] == false) continue;    // can't make i, skip

        for (int num : numbers) {
            if (i + num <= targetSum) {
                dp[i + num] = true;      // we can also make i+num
            }
        }
    }

    // STEP 4: Answer is at the target index
    return dp[targetSum];
}

// ─────────────────────────────────────────────────────────────
// BONUS: Print the dp table (for learning purposes)
// ─────────────────────────────────────────────────────────────
void printTable(int targetSum, const vector<int>& numbers) {
    vector<bool> dp(targetSum + 1, false);
    dp[0] = true;

    cout << "\nBuilding dp table for canSum(" << targetSum << ", [";
    for (int i = 0; i < (int)numbers.size(); i++) {
        cout << numbers[i];
        if (i < (int)numbers.size() - 1) cout << ",";
    }
    cout << "]):" << endl;

    cout << "Index: ";
    for (int i = 0; i <= targetSum; i++) cout << "[" << i << "] ";
    cout << endl;

    cout << "Start: ";
    for (int i = 0; i <= targetSum; i++) cout << (dp[i] ? " T   " : " F   ");
    cout << endl;

    for (int i = 0; i <= targetSum; i++) {
        if (!dp[i]) continue;
        for (int num : numbers) {
            if (i + num <= targetSum) dp[i + num] = true;
        }
        cout << "i=" << i << ":   ";
        for (int j = 0; j <= targetSum; j++) cout << (dp[j] ? " T   " : " F   ");
        cout << endl;
    }

    cout << "Answer: dp[" << targetSum << "] = " << (dp[targetSum] ? "true" : "false") << endl;
}

int main() {
    cout << "canSum (Tabulation)" << endl;
    cout << "──────────────────────────────────────────────────" << endl;

    cout << boolalpha;

    cout << "canSum(7,   [2,3])    = " << canSum_tabulation(7,   {2, 3})       << endl;
    cout << "canSum(7,   [5,3,4])  = " << canSum_tabulation(7,   {5, 3, 4})    << endl;
    cout << "canSum(7,   [2,4])    = " << canSum_tabulation(7,   {2, 4})       << endl;
    cout << "canSum(8,   [2,3,5])  = " << canSum_tabulation(8,   {2, 3, 5})    << endl;
    cout << "canSum(0,   [1,2,3])  = " << canSum_tabulation(0,   {1, 2, 3})    << endl;
    cout << "canSum(300, [7,14])   = " << canSum_tabulation(300, {7, 14})      << endl;

    // Print tables for small examples
    printTable(7, {2, 3});
    printTable(7, {2, 4});

    return 0;
}
```

**Output:**
```
canSum (Tabulation)
──────────────────────────────────────────────────
canSum(7,   [2,3])    = true
canSum(7,   [5,3,4])  = true
canSum(7,   [2,4])    = false
canSum(8,   [2,3,5])  = true
canSum(0,   [1,2,3])  = true
canSum(300, [7,14])   = false

Building dp table for canSum(7, [2,3]):
Index: [0] [1] [2] [3] [4] [5] [6] [7]
Start:  T    F    F    F    F    F    F    F
i=0:    T    F    T    T    F    F    F    F
i=2:    T    F    T    T    T    T    F    F
i=3:    T    F    T    T    T    T    T    F
i=4:    T    F    T    T    T    T    T    T   ← dp[7] flipped!
Answer: dp[7] = true

Building dp table for canSum(7, [2,4]):
Index: [0] [1] [2] [3] [4] [5] [6] [7]
Start:  T    F    F    F    F    F    F    F
i=0:    T    F    T    F    T    F    F    F
i=2:    T    F    T    F    T    F    T    F
i=4:    T    F    T    F    T    F    T    F
i=6:    T    F    T    F    T    F    T    F
Answer: dp[7] = false
```

### Complexity Analysis — Tabulation

```
TIME COMPLEXITY: O(n * m)
──────────────────────────
Outer loop: i from 0 to m  → m+1 iterations
Inner loop: for each num in numbers → n iterations per outer step
Each inner step: O(1) array write.

Total: (m+1) × n ≈ O(n * m)

Same as memoization! But with no recursion overhead.

SPACE COMPLEXITY: O(m)
────────────────────────
dp array: m+1 booleans → O(m)
NO call stack at all (no recursion).

For canSum(300, [7,14]):
  dp has 301 entries (one bool per sum 0..300) → ~301 bytes
  vs recursion which would stack overflow or take forever.

TABULATION is the SAFEST version — no risk of stack overflow.
```

---

## All Three Together — One Complete File

```cpp
#include <iostream>
#include <vector>
#include <map>
using namespace std;

// ════════════════════════════════════════════════════════════
// TECHNIQUE 1: RECURSION
// Time: O(n^m)   Space: O(m) call stack
// ════════════════════════════════════════════════════════════
bool cs_recursive(int target, const vector<int>& nums) {
    if (target == 0) return true;
    if (target < 0)  return false;
    for (int num : nums)
        if (cs_recursive(target - num, nums)) return true;
    return false;
}

// ════════════════════════════════════════════════════════════
// TECHNIQUE 2: MEMOIZATION (Top-Down DP)
// Time: O(n*m)   Space: O(m) memo + O(m) stack
// ════════════════════════════════════════════════════════════
bool cs_memo(int target, const vector<int>& nums, map<int, bool>& memo) {
    if (memo.count(target)) return memo[target];  // cache hit
    if (target == 0) return true;
    if (target < 0)  return false;
    for (int num : nums) {
        if (cs_memo(target - num, nums, memo)) {
            return memo[target] = true;   // cache true and return
        }
    }
    return memo[target] = false;          // cache false and return
}

// ════════════════════════════════════════════════════════════
// TECHNIQUE 3: TABULATION (Bottom-Up DP)
// Time: O(n*m)   Space: O(m) — NO call stack
// ════════════════════════════════════════════════════════════
bool cs_tabulation(int target, const vector<int>& nums) {
    vector<bool> dp(target + 1, false);
    dp[0] = true;
    for (int i = 0; i <= target; i++) {
        if (!dp[i]) continue;
        for (int num : nums)
            if (i + num <= target) dp[i + num] = true;
    }
    return dp[target];
}

// ════════════════════════════════════════════════════════════
// MAIN — Compare all techniques
// ════════════════════════════════════════════════════════════
int main() {
    cout << boolalpha;
    cout << "target  numbers      Recursive  Memoized  Tabulation" << endl;
    cout << "──────────────────────────────────────────────────────" << endl;

    // For small targets only — recursion is too slow for large targets
    struct TestCase { int target; vector<int> nums; };
    vector<TestCase> tests = {
        {7,  {2, 3}},
        {7,  {5, 3, 4}},
        {7,  {2, 4}},
        {8,  {2, 3, 5}},
        {0,  {1, 2, 3}},
    };

    for (auto& [t, nums] : tests) {
        map<int, bool> memo;
        cout << t << "       [";
        for (int i = 0; i < (int)nums.size(); i++) {
            cout << nums[i];
            if (i < (int)nums.size()-1) cout << ",";
        }
        cout << "]        "
             << cs_recursive(t, nums)          << "       "
             << cs_memo(t, nums, memo)         << "       "
             << cs_tabulation(t, nums)
             << endl;
    }

    // Large targets — only memo and tabulation
    cout << "\nLarge targets (memo + tabulation only):" << endl;
    cout << "────────────────────────────────────────" << endl;
    vector<TestCase> large = {
        {300, {7, 14}},
        {150, {7, 14}},     // 7*? = 150? 150/7 ≈ 21.4, not integer → false
        {140, {7, 14}},     // 7*20 = 140 → true
    };
    for (auto& [t, nums] : large) {
        map<int, bool> memo;
        cout << "canSum(" << t << ", [" << nums[0] << "," << nums[1] << "]):  "
             << "Memo=" << cs_memo(t, nums, memo) << "  "
             << "Tab="  << cs_tabulation(t, nums)
             << endl;
    }

    return 0;
}
```

**Output:**
```
target  numbers      Recursive  Memoized  Tabulation
──────────────────────────────────────────────────────
7       [2,3]        true       true       true
7       [5,3,4]      true       true       true
7       [2,4]        false      false      false
8       [2,3,5]      true       true       true
0       [1,2,3]      true       true       true

Large targets (memo + tabulation only):
────────────────────────────────────────
canSum(300, [7,14]):  Memo=false  Tab=false
canSum(150, [7,14]):  Memo=false  Tab=false
canSum(140, [7,14]):  Memo=true   Tab=true
```

---

## Complexity Summary — Side by Side

```
┌──────────────────┬─────────────────┬────────────────────────────────────────┐
│ Technique        │      TIME       │                 SPACE                  │
├──────────────────┼─────────────────┼────────────────────────────────────────┤
│ Recursion        │   O(n^m)        │ O(m) — call stack depth                │
│                  │                 │ WARNING: Exponential — fails fast       │
├──────────────────┼─────────────────┼────────────────────────────────────────┤
│ Memoization      │   O(n * m)      │ O(m) — memo map (m+1 entries)          │
│ (Top-Down DP)    │                 │ O(m) — call stack                      │
│                  │                 │ Total: O(m) — far better than O(n^m)   │
├──────────────────┼─────────────────┼────────────────────────────────────────┤
│ Tabulation       │   O(n * m)      │ O(m) — dp boolean array                │
│ (Bottom-Up DP)   │                 │ O(1) — no call stack                   │
│                  │                 │ SAFEST — no recursion, no overflow      │
└──────────────────┴─────────────────┴────────────────────────────────────────┘

n = length of numbers array
m = targetSum
```

---

## Understanding WHY the Complexities Are What They Are

### Why Recursion is O(n^m)

```
At every call, we branch into n sub-calls (one per number in array).
The depth of the recursion tree ≤ m:
  Worst case: smallest number = 1
  canSum(m) → canSum(m-1) → canSum(m-2) → ... → canSum(0)
  Depth = m, branching factor = n
  → Total nodes ≤ n^m

Example: canSum(300, [7, 14])
  n=2, m=300 → 2^300 ≈ 10^90 calls → impossible to run.

But wait — many branches terminate early (negative target).
In the best case (first number always works), it's O(m/num) calls.
In the WORST case, it's O(n^m).
```

### Why Memoization and Tabulation are both O(n * m)

```
KEY INSIGHT: How many UNIQUE subproblems exist?

The only argument that changes between calls is targetSum.
targetSum can be: 0, 1, 2, ..., m  → (m+1) unique values.

Once we cache each unique targetSum, each is computed ONCE.
For each unique targetSum, we try n numbers → n work.

Total: (m+1) × n ≈ O(n * m)

For canSum(300, [7, 14]):
  n=2, m=300 → 2 × 300 = 600 operations → instant! ✅
```

### Why Recursion Space is O(m) but NOT O(n^m)

```
Space = how much memory is used AT ONE TIME.

The entire recursion tree is NOT in memory simultaneously.
Only the CURRENT PATH from root to current node is on the stack.

Deepest possible path (subtracting the smallest number each time):
  canSum(m) → canSum(m-1) → canSum(m-2) → ... → canSum(0)
  Length = m → at most m frames on the stack simultaneously.

Other branches are explored after earlier ones RETURN.
→ Stack space = O(m), not O(n^m).

This is why recursion can be FAST on memory but SLOW on time.
```

---

## Visualizing the "Infection" Spread — Tabulation Intuition

```
Think of dp as a NUMBER LINE 0 to targetSum.
dp[0] = true means "position 0 is lit up" (reachable).

From every lit position, we can "jump" by any number in the array.
If a jump lands within bounds, that position lights up too.

canSum(7, [2, 3]):

0   1   2   3   4   5   6   7
🟢  ⚫  ⚫  ⚫  ⚫  ⚫  ⚫  ⚫   Initial: only 0 lit

From 0, jump +2 and +3:
0   1   2   3   4   5   6   7
🟢  ⚫  🟢  🟢  ⚫  ⚫  ⚫  ⚫

From 2, jump +2 and +3:
0   1   2   3   4   5   6   7
🟢  ⚫  🟢  🟢  🟢  🟢  ⚫  ⚫

From 3, jump +2 and +3:
0   1   2   3   4   5   6   7
🟢  ⚫  🟢  🟢  🟢  🟢  🟢  ⚫

From 4, jump +3:
0   1   2   3   4   5   6   7
🟢  ⚫  🟢  🟢  🟢  🟢  🟢  🟢  ← 7 lit up! Answer = true ✅

canSum(7, [2, 4]):
0   1   2   3   4   5   6   7
🟢  ⚫  🟢  ⚫  🟢  ⚫  🟢  ⚫   Only even positions ever light up!
                                 7 stays dark → false ✗
```

---

## The Interview Answer — What to Say and When

```
INTERVIEWER: "Can you determine if a target sum is achievable from an array?"

STEP 1: Understand the problem
  "Given targetSum and array of numbers (reusable), return true if any
   combination sums to the target. Numbers are non-negative."
  
  Give 2 examples: canSum(7,[2,3])=true, canSum(7,[2,4])=false.

STEP 2: Write recursive solution
  "Base cases: return true if target==0, return false if target<0.
   For each number, subtract it and recursively check the remainder."
  → Write ~8 lines of recursive code.
  
  "Time: O(n^m) — exponential. Fails for large inputs like canSum(300,...)."

STEP 3: Identify overlapping subproblems
  "The same targetSum value is computed multiple times in different branches.
   This is the classic overlapping subproblems property of DP."
  → Draw tree, circle repeated subtrees.

STEP 4: Add memoization
  "Cache the result of each targetSum in a map.
   Before computing, check if it's already cached.
   After computing, store it."
  → Add map, check at top, store before return.
  
  "Time: O(n*m) — only m unique subproblems, each tries n numbers."

STEP 5: Convert to tabulation
  "Eliminate recursion — use a boolean array dp of size m+1.
   dp[0]=true. For each true dp[i], set dp[i+num]=true.
   Return dp[targetSum]."
  → Write the two nested loops.
  
  "Same time O(n*m), same space O(m), but no call stack — safer."

This progression: naive → identify waste → memoize → tabulate.
```

---

## Quick Cheat Sheet

```
PROBLEM: Can we make targetSum using numbers from array? (reuse allowed)
         Return: true or false

FORMULA: For each num, check canSum(target - num, numbers)
BASE:    target == 0 → true;    target < 0 → false

─────────────────────────────────────────────────────────────

RECURSION:
  if(target==0) return true;
  if(target<0)  return false;
  for(int num : nums)
    if(canSum(target-num, nums)) return true;
  return false;
  
  Time: O(n^m)   Space: O(m) stack

─────────────────────────────────────────────────────────────

MEMOIZATION:
  if(memo.count(target)) return memo[target];
  if(target==0) return true;
  if(target<0)  return false;
  for(int num : nums)
    if(cs_memo(target-num, nums, memo)) return memo[target]=true;
  return memo[target]=false;
  
  Time: O(n*m)   Space: O(m) memo + O(m) stack

─────────────────────────────────────────────────────────────

TABULATION:
  vector<bool> dp(target+1, false);
  dp[0] = true;
  for(int i=0; i<=target; i++) {
    if(!dp[i]) continue;
    for(int num : nums)
      if(i+num<=target) dp[i+num]=true;
  }
  return dp[target];
  
  Time: O(n*m)   Space: O(m) — NO STACK

─────────────────────────────────────────────────────────────

ANSWERS:
  canSum(7,[2,3])    = true    canSum(7,[2,4])    = false
  canSum(7,[5,3,4])  = true    canSum(300,[7,14]) = false
  canSum(8,[2,3,5])  = true    canSum(140,[7,14]) = true
```

---

*Compile and run: `g++ -o cansum can_sum.cpp && ./cansum`*
