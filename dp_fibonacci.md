# Fibonacci Series — Three Techniques
## Recursion vs Memoization vs Tabulation in C++

> Fibonacci: F(n) = F(n-1) + F(n-2)
> Base cases: F(0) = 0, F(1) = 1
> Series:     0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55 ...

---

## Quick Comparison (Read This First)

```
TECHNIQUE       TIME        SPACE       STYLE       BEST FOR
────────────────────────────────────────────────────────────────
Recursion       O(2^n)      O(n)        Top-down    Understanding the problem
Memoization     O(n)        O(n)+O(n)*  Top-down    Recursive problems + cache
Tabulation      O(n)        O(n)        Bottom-up   Most DP problems
Space-Opt Tab   O(n)        O(1)        Bottom-up   When memory matters

* O(n) memo array + O(n) call stack
```

---

## The Problem Visualized

```
Fibonacci(5) = ?

F(0)=0  F(1)=1  F(2)=1  F(3)=2  F(4)=3  F(5)=5

         Index:  0   1   2   3   4   5
         Value:  0   1   1   2   3   5
                         ↑           ↑
                       F(2)=        F(5)=
                       F(1)+F(0)    F(4)+F(3)
                       = 1+0 = 1    = 3+2 = 5
```

---

## Technique 1 — Recursion (Naive)

### The Idea

Directly translate the mathematical definition into code.
**F(n) = F(n-1) + F(n-2)** becomes a function that calls itself.

### The Recursion Tree for F(5)

```
                         F(5)
                        /    \
                    F(4)      F(3)
                   /    \    /    \
               F(3)   F(2) F(2)  F(1)
              /    \   / \   / \
           F(2)  F(1) F(1)F(0)F(1)F(0)
           / \
         F(1) F(0)

PROBLEM: See F(3) computed TWICE, F(2) computed THREE times
         Same subproblem solved over and over = WASTED WORK

Total function calls for F(5)  = 15
Total function calls for F(10) = 177
Total function calls for F(20) = 21891
Total function calls for F(40) = 331,160,281  ← takes seconds
Total function calls for F(50) = ~34 BILLION  ← takes minutes!
```

### C++ Code

```cpp
#include <iostream>
using namespace std;

// ─────────────────────────────────────────────
// TECHNIQUE 1: RECURSION (Naive)
// ─────────────────────────────────────────────

int fibonacci_recursive(int n) {
    // BASE CASES: stop recursion here
    if (n == 0) return 0;
    if (n == 1) return 1;

    // RECURSIVE CASE: break into smaller subproblems
    return fibonacci_recursive(n - 1) + fibonacci_recursive(n - 2);
}

int main() {
    int n = 10;

    cout << "Fibonacci (Recursion)" << endl;
    cout << "─────────────────────" << endl;

    for (int i = 0; i <= n; i++) {
        cout << "F(" << i << ") = " << fibonacci_recursive(i) << endl;
    }

    return 0;
}
```

**Output:**
```
Fibonacci (Recursion)
─────────────────────
F(0) = 0
F(1) = 1
F(2) = 1
F(3) = 2
F(4) = 3
F(5) = 5
F(6) = 8
F(7) = 13
F(8) = 21
F(9) = 34
F(10) = 55
```

### Complexity Analysis

```
TIME COMPLEXITY: O(2^n)
───────────────────────
Each call makes 2 more calls (except base cases).
This forms a binary tree of calls.

A binary tree of depth n has at most 2^n nodes.
→ approximately 2^n function calls total.

n=10  → ~1,024 calls
n=20  → ~1,048,576 calls
n=40  → ~1,099,511,627,776 calls  ← 1 trillion! very slow
n=50  → ~1,125,899,906,842,624 calls ← will never finish

Growth rate: EXPONENTIAL → doubles with every increase in n

SPACE COMPLEXITY: O(n)
──────────────────────
Space = call stack depth = depth of recursion tree.

The deepest path in the tree:
  F(n) → F(n-1) → F(n-2) → ... → F(1) → F(0)
  Depth = n

At any moment, at most n frames are on the call stack.

Stack at deepest point for F(5):
  [main]
  [fib(5)]
  [fib(4)]
  [fib(3)]
  [fib(2)]
  [fib(1)]   ← returns 1, stack starts unwinding
  
→ Space = O(n) for the call stack.

NOTE: O(n) space seems fine, but for n=10,000:
  → 10,000 stack frames
  → Risk of STACK OVERFLOW on most systems
```

### When to Use Recursion

```
✅ Learning / understanding the recursive structure
✅ Very small n (n < 20)
✅ Interview explanation of the problem
❌ Never use in production for large n
❌ n > 40 becomes unusably slow
```

---

## Technique 2 — Memoization (Top-Down DP)

### The Idea

**Same recursion, but CACHE results so each subproblem is solved only ONCE.**

Before computing F(n):
- Check if we already computed it
- If YES → return cached result (no recomputation)
- If NO  → compute it, store it, return it

This is called **Top-Down** because we start from F(n) and break it down.
It is also called **Dynamic Programming with memoization**.

### How Memoization Prunes the Tree

```
WITHOUT memoization — F(5) computes F(3) twice:
                         F(5)
                        /    \
                    F(4)      F(3) ← computed again!
                   /    \    /    \
               F(3)   F(2) F(2)  F(1)
               ...

WITH memoization — F(3) computed once, result stored:
                         F(5)
                        /    \
                    F(4)      F(3) ← CACHE HIT! returns memo[3] instantly
                   /    \
               F(3)   F(2)  ← F(3) computed HERE first, stored in memo
              /    \
           F(2)  F(1)
           / \
         F(1) F(0)

memo array after computing F(5):
  index: [0]  [1]  [2]  [3]  [4]  [5]
  value: [ 0]  [ 1]  [ 1]  [ 2]  [ 3]  [ 5]
         filled as each subproblem is solved for the first time
```

### C++ Code

```cpp
#include <iostream>
#include <vector>
using namespace std;

// ─────────────────────────────────────────────
// TECHNIQUE 2: MEMOIZATION (Top-Down DP)
// ─────────────────────────────────────────────

// -1 means "not yet computed"
// We use a global/passed memo array to cache results
int fibonacci_memo(int n, vector<int>& memo) {

    // STEP 1: Check cache first
    if (memo[n] != -1) {
        return memo[n];   // already computed, return immediately
    }

    // STEP 2: Base cases
    if (n == 0) {
        memo[0] = 0;
        return 0;
    }
    if (n == 1) {
        memo[1] = 1;
        return 1;
    }

    // STEP 3: Compute, STORE in cache, then return
    memo[n] = fibonacci_memo(n - 1, memo) + fibonacci_memo(n - 2, memo);
    return memo[n];
}

int main() {
    int n = 10;

    // Initialize memo with -1 (means "not computed yet")
    vector<int> memo(n + 1, -1);

    cout << "Fibonacci (Memoization)" << endl;
    cout << "────────────────────────" << endl;

    for (int i = 0; i <= n; i++) {
        // Reset memo for each fresh call in this demo
        // (in real use, compute F(n) once with one memo)
        vector<int> m(i + 1, -1);
        cout << "F(" << i << ") = " << fibonacci_memo(i, m) << endl;
    }

    // More practical usage — compute all at once:
    cout << "\nAll at once:" << endl;
    vector<int> memo2(n + 1, -1);
    cout << "F(" << n << ") = " << fibonacci_memo(n, memo2) << endl;
    cout << "Memo cache: ";
    for (int i = 0; i <= n; i++) cout << memo2[i] << " ";
    cout << endl;

    return 0;
}
```

**Output:**
```
Fibonacci (Memoization)
────────────────────────
F(0) = 0
F(1) = 1
F(2) = 1
F(3) = 2
F(4) = 3
F(5) = 5
F(6) = 8
F(7) = 13
F(8) = 21
F(9) = 34
F(10) = 55

All at once:
F(10) = 55
Memo cache: 0 1 1 2 3 5 8 13 21 34 55
```

### Step-by-Step Trace for F(5) with Memoization

```
Call stack builds up (going DOWN):
  fib(5) → memo[5] = -1, compute it
    fib(4) → memo[4] = -1, compute it
      fib(3) → memo[3] = -1, compute it
        fib(2) → memo[2] = -1, compute it
          fib(1) → BASE CASE → memo[1] = 1, return 1
          fib(0) → BASE CASE → memo[0] = 0, return 0
        fib(2) = 1 + 0 = 1 → memo[2] = 1 ✅ STORED
        return 1
      fib(1) → memo[1] = 1 ✅ CACHE HIT! return 1
      fib(3) = 1 + 1 = 2 → memo[3] = 2 ✅ STORED
      return 2
    fib(2) → memo[2] = 1 ✅ CACHE HIT! return 1
    fib(4) = 2 + 1 = 3 → memo[4] = 3 ✅ STORED
    return 3
  fib(3) → memo[3] = 2 ✅ CACHE HIT! return 2
  fib(5) = 3 + 2 = 5 → memo[5] = 5 ✅ STORED
  return 5

Total unique calls: 6  (instead of 15 without memoization)
Cache hits: 2  (fib(1) and fib(2) and fib(3) reused)
```

### Complexity Analysis

```
TIME COMPLEXITY: O(n)
──────────────────────
Each unique subproblem F(0), F(1), ..., F(n) is computed EXACTLY ONCE.
After first computation → stored in memo → any future call = O(1) lookup.

Total unique subproblems = n + 1
Each solved in O(1) (just addition after subproblems are solved)
→ Total time = O(n)

Compare:
  Recursion:    O(2^n)   → millions of calls for n=40
  Memoization:  O(n)     → exactly n+1 calls for any n

SPACE COMPLEXITY: O(n) + O(n) = O(n)
──────────────────────────────────────
TWO sources of space:

1. MEMO ARRAY:
   vector<int> memo(n + 1)  → stores n+1 values → O(n)

2. CALL STACK:
   Recursion still happens — max depth is n
   At deepest point: fib(n) → fib(n-1) → ... → fib(0) = n frames
   → O(n) stack space

Total: O(n) + O(n) = O(n)

NOTE: For n = 10,000 you get:
  → 10,000 stack frames (still risk of stack overflow)
  → Tabulation (Technique 3) solves this — no recursion, no stack issue
```

### When to Use Memoization

```
✅ When recursion is natural for the problem structure
✅ When you don't need ALL subproblem answers (sparse subproblems)
✅ When subproblems have complex indices (not simple 0..n)
✅ Tree DP, graph DP — where tabulation order is hard to define
❌ When n is very large (stack overflow risk)
❌ When you need absolute minimum space (use tabulation)
```

---

## Technique 3 — Tabulation (Bottom-Up DP)

### The Idea

**Build the answer from the ground up, using a table (array).**

Instead of starting at F(n) and recursing down:
- Start at base cases F(0) and F(1)
- Fill the table from left to right
- Each entry uses previously filled entries

This is called **Bottom-Up** because we start from the smallest subproblems
and build up to F(n). No recursion, no call stack.

### How the Table is Filled

```
n = 7

Start with base cases:
  index: [0]  [1]  [2]  [3]  [4]  [5]  [6]  [7]
  dp:    [ 0]  [ 1]  [?]  [?]  [?]  [?]  [?]  [?]

Fill dp[2] = dp[1] + dp[0] = 1 + 0 = 1:
  index: [0]  [1]  [2]  [3]  [4]  [5]  [6]  [7]
  dp:    [ 0]  [ 1]  [ 1]  [?]  [?]  [?]  [?]  [?]

Fill dp[3] = dp[2] + dp[1] = 1 + 1 = 2:
  index: [0]  [1]  [2]  [3]  [4]  [5]  [6]  [7]
  dp:    [ 0]  [ 1]  [ 1]  [ 2]  [?]  [?]  [?]  [?]

Fill dp[4] = dp[3] + dp[2] = 2 + 1 = 3:
  index: [0]  [1]  [2]  [3]  [4]  [5]  [6]  [7]
  dp:    [ 0]  [ 1]  [ 1]  [ 2]  [ 3]  [?]  [?]  [?]

Fill dp[5] = dp[4] + dp[3] = 3 + 2 = 5:
  index: [0]  [1]  [2]  [3]  [4]  [5]  [6]  [7]
  dp:    [ 0]  [ 1]  [ 1]  [ 2]  [ 3]  [ 5]  [?]  [?]

Fill dp[6] = dp[5] + dp[4] = 5 + 3 = 8:
  index: [0]  [1]  [2]  [3]  [4]  [5]  [6]  [7]
  dp:    [ 0]  [ 1]  [ 1]  [ 2]  [ 3]  [ 5]  [ 8]  [?]

Fill dp[7] = dp[6] + dp[5] = 8 + 5 = 13:
  index: [0]  [1]  [2]  [3]  [4]  [5]  [6]  [7]
  dp:    [ 0]  [ 1]  [ 1]  [ 2]  [ 3]  [ 5]  [ 8]  [13]

Answer: dp[7] = 13  ✅

No recursion. No call stack. Pure iteration.
```

### C++ Code

```cpp
#include <iostream>
#include <vector>
using namespace std;

// ─────────────────────────────────────────────
// TECHNIQUE 3A: TABULATION (Bottom-Up DP)
// Space: O(n)
// ─────────────────────────────────────────────

int fibonacci_tabulation(int n) {
    if (n == 0) return 0;
    if (n == 1) return 1;

    // Create table of size n+1
    vector<int> dp(n + 1);

    // STEP 1: Fill base cases
    dp[0] = 0;
    dp[1] = 1;

    // STEP 2: Fill table from bottom up (i=2 to n)
    for (int i = 2; i <= n; i++) {
        dp[i] = dp[i - 1] + dp[i - 2];   // each entry uses previous two
    }

    // STEP 3: Answer is at dp[n]
    return dp[n];
}

// ─────────────────────────────────────────────
// TECHNIQUE 3B: TABULATION WITH SPACE OPTIMIZATION
// Space: O(1) — we only need the last two values!
// ─────────────────────────────────────────────

int fibonacci_optimized(int n) {
    if (n == 0) return 0;
    if (n == 1) return 1;

    int prev2 = 0;   // F(i-2)
    int prev1 = 1;   // F(i-1)
    int curr  = 0;   // F(i)

    for (int i = 2; i <= n; i++) {
        curr  = prev1 + prev2;   // F(i) = F(i-1) + F(i-2)
        prev2 = prev1;           // shift: F(i-2) becomes old F(i-1)
        prev1 = curr;            // shift: F(i-1) becomes current
    }

    return curr;
}

int main() {
    int n = 10;

    // TABULATION (O(n) space)
    cout << "Fibonacci (Tabulation — O(n) space)" << endl;
    cout << "─────────────────────────────────────" << endl;
    for (int i = 0; i <= n; i++) {
        cout << "F(" << i << ") = " << fibonacci_tabulation(i) << endl;
    }

    cout << endl;

    // SPACE-OPTIMIZED TABULATION (O(1) space)
    cout << "Fibonacci (Space Optimized — O(1) space)" << endl;
    cout << "──────────────────────────────────────────" << endl;
    for (int i = 0; i <= n; i++) {
        cout << "F(" << i << ") = " << fibonacci_optimized(i) << endl;
    }

    return 0;
}
```

**Output:**
```
Fibonacci (Tabulation — O(n) space)
─────────────────────────────────────
F(0) = 0
F(1) = 1
F(2) = 1
F(3) = 2
F(4) = 3
F(5) = 5
F(6) = 8
F(7) = 13
F(8) = 21
F(9) = 34
F(10) = 55

Fibonacci (Space Optimized — O(1) space)
──────────────────────────────────────────
F(0) = 0
F(1) = 1
F(2) = 1
F(3) = 2
F(4) = 3
F(5) = 5
F(6) = 8
F(7) = 13
F(8) = 21
F(9) = 34
F(10) = 55
```

### Step-by-Step Trace of Space-Optimized for n=7

```
i=2: curr = 1+0 = 1   | prev2=0,  prev1=1          → shift: prev2=1,  prev1=1
i=3: curr = 1+1 = 2   | prev2=1,  prev1=1          → shift: prev2=1,  prev1=2
i=4: curr = 2+1 = 3   | prev2=1,  prev1=2          → shift: prev2=2,  prev1=3
i=5: curr = 3+2 = 5   | prev2=2,  prev1=3          → shift: prev2=3,  prev1=5
i=6: curr = 5+3 = 8   | prev2=3,  prev1=5          → shift: prev2=5,  prev1=8
i=7: curr = 8+5 = 13  | prev2=5,  prev1=8          → shift: prev2=8,  prev1=13

return curr = 13  ✅

Variables used: only 3 integers (prev2, prev1, curr) regardless of n.
→ CONSTANT SPACE O(1)
```

### Complexity Analysis

```
TABULATION (O(n) space):

TIME COMPLEXITY: O(n)
──────────────────────
Single for loop from i=2 to i=n.
Each iteration: O(1) work (one addition, one assignment).
Total: n-1 iterations → O(n)

SPACE COMPLEXITY: O(n)
──────────────────────
dp array of size n+1.
→ O(n)

No call stack at all (no recursion).
Safe for any n (n=1,000,000 works fine).


SPACE-OPTIMIZED TABULATION (O(1) space):

TIME COMPLEXITY: O(n)
──────────────────────
Same single loop → O(n)

SPACE COMPLEXITY: O(1)
──────────────────────
Only 3 integer variables (prev2, prev1, curr).
Doesn't matter if n = 10 or n = 10,000,000.
→ CONSTANT SPACE

This is the BEST possible space complexity for Fibonacci.
```

### When to Use Tabulation

```
✅ Standard choice for most DP problems
✅ When you need all subproblem answers
✅ When n is large (no stack overflow risk)
✅ When you want O(1) space optimization
✅ When iteration order is clear (left to right, bottom to top)
❌ When subproblems have complex dependencies (memoization is easier)
```

---

## All Three Together — One Complete File

```cpp
#include <iostream>
#include <vector>
using namespace std;

// ════════════════════════════════════════════════════
// TECHNIQUE 1: RECURSION
// Time:  O(2^n)   Space: O(n) [call stack]
// ════════════════════════════════════════════════════
int fib_recursive(int n) {
    if (n == 0) return 0;
    if (n == 1) return 1;
    return fib_recursive(n - 1) + fib_recursive(n - 2);
}

// ════════════════════════════════════════════════════
// TECHNIQUE 2: MEMOIZATION (Top-Down DP)
// Time:  O(n)     Space: O(n) memo + O(n) stack = O(n)
// ════════════════════════════════════════════════════
int fib_memo(int n, vector<int>& memo) {
    if (memo[n] != -1) return memo[n];     // cache hit
    if (n == 0) return memo[0] = 0;
    if (n == 1) return memo[1] = 1;
    return memo[n] = fib_memo(n - 1, memo) + fib_memo(n - 2, memo);
}

// ════════════════════════════════════════════════════
// TECHNIQUE 3A: TABULATION (Bottom-Up DP)
// Time:  O(n)     Space: O(n)
// ════════════════════════════════════════════════════
int fib_tabulation(int n) {
    if (n == 0) return 0;
    if (n == 1) return 1;
    vector<int> dp(n + 1);
    dp[0] = 0;
    dp[1] = 1;
    for (int i = 2; i <= n; i++)
        dp[i] = dp[i - 1] + dp[i - 2];
    return dp[n];
}

// ════════════════════════════════════════════════════
// TECHNIQUE 3B: SPACE-OPTIMIZED TABULATION
// Time:  O(n)     Space: O(1)  ← BEST!
// ════════════════════════════════════════════════════
int fib_optimized(int n) {
    if (n == 0) return 0;
    if (n == 1) return 1;
    int prev2 = 0, prev1 = 1, curr = 0;
    for (int i = 2; i <= n; i++) {
        curr  = prev1 + prev2;
        prev2 = prev1;
        prev1 = curr;
    }
    return curr;
}

// ════════════════════════════════════════════════════
// MAIN — Compare all techniques
// ════════════════════════════════════════════════════
int main() {
    int n = 10;

    cout << "n    Recursive  Memoized  Tabulation  Optimized" << endl;
    cout << "────────────────────────────────────────────────" << endl;

    for (int i = 0; i <= n; i++) {
        vector<int> memo(i + 1, -1);

        cout << "F(" << i << ")  "
             << "  " << fib_recursive(i)
             << "         " << fib_memo(i, memo)
             << "         " << fib_tabulation(i)
             << "           " << fib_optimized(i)
             << endl;
    }

    // Test with larger n (recursion would be too slow for large n)
    cout << "\nLarge n test (only fast techniques):" << endl;
    int big_n = 40;
    vector<int> memo(big_n + 1, -1);
    cout << "F(" << big_n << ") via Memoization  = " << fib_memo(big_n, memo)       << endl;
    cout << "F(" << big_n << ") via Tabulation   = " << fib_tabulation(big_n)       << endl;
    cout << "F(" << big_n << ") via Optimized    = " << fib_optimized(big_n)        << endl;
    // fib_recursive(40) would take ~seconds — not called here

    return 0;
}
```

**Output:**
```
n    Recursive  Memoized  Tabulation  Optimized
────────────────────────────────────────────────
F(0)    0         0         0           0
F(1)    1         1         1           1
F(2)    1         1         1           1
F(3)    2         2         2           2
F(4)    3         3         3           3
F(5)    5         5         5           5
F(6)    8         8         8           8
F(7)    13        13        13          13
F(8)    21        21        21          21
F(9)    34        34        34          34
F(10)   55        55        55          55

Large n test (only fast techniques):
F(40) via Memoization  = 102334155
F(40) via Tabulation   = 102334155
F(40) via Optimized    = 102334155
```

---

## Complexity Summary — Side by Side

```
┌────────────────────┬────────────┬────────────────────────────────────────────┐
│ Technique          │    TIME    │                  SPACE                     │
├────────────────────┼────────────┼────────────────────────────────────────────┤
│ Recursion          │   O(2^n)   │ O(n) — call stack depth                    │
│                    │            │ WARNING: Stack overflow for large n         │
├────────────────────┼────────────┼────────────────────────────────────────────┤
│ Memoization        │   O(n)     │ O(n) — memo array                          │
│ (Top-Down DP)      │            │ O(n) — call stack                          │
│                    │            │ Total: O(n) + O(n) = O(n)                  │
│                    │            │ WARNING: Stack overflow still possible      │
├────────────────────┼────────────┼────────────────────────────────────────────┤
│ Tabulation         │   O(n)     │ O(n) — dp array                            │
│ (Bottom-Up DP)     │            │ O(1) — no call stack (no recursion)        │
│                    │            │ Total: O(n)                                │
│                    │            │ SAFE for any n                             │
├────────────────────┼────────────┼────────────────────────────────────────────┤
│ Tabulation         │   O(n)     │ O(1) — only 3 variables                    │
│ Space-Optimized    │            │ BEST space possible!                       │
│                    │            │ SAFE for any n                             │
└────────────────────┴────────────┴────────────────────────────────────────────┘
```

---

## Understanding WHY the Complexities Are What They Are

### Why Recursion is O(2^n)

```
Think of it this way — every call creates 2 more calls:

F(n)
├── F(n-1)
│   ├── F(n-2)
│   │   ├── F(n-3) ...
│   │   └── F(n-4) ...
│   └── F(n-3) ...  ← DUPLICATE WORK
│       ├── F(n-4) ...
│       └── F(n-5) ...
└── F(n-2) ...       ← DUPLICATE WORK (same as above)
    ├── F(n-3) ...
    └── F(n-4) ...

Each level doubles the number of calls.

Level 0:  1 call     (F(n))
Level 1:  2 calls    (F(n-1), F(n-2))
Level 2:  4 calls    (F(n-2), F(n-3), F(n-3), F(n-4))
Level 3:  8 calls
...
Level n:  2^n calls (roughly)

Total ≈ 2^n → EXPONENTIAL
```

### Why Memoization brings it to O(n)

```
WITH MEMO:
  F(0) computed: 1 time → stored
  F(1) computed: 1 time → stored
  F(2) computed: 1 time → stored
  F(3) computed: 1 time → stored
  ...
  F(n) computed: 1 time → stored

Each subproblem: solved ONCE, looked up in O(1) afterwards.
Total unique subproblems = n+1

Total work = (n+1) × O(1) = O(n)

The memo "cuts off" all duplicate branches of the recursion tree.
```

### Why Tabulation is O(n) with O(1) space possible

```
TABULATION fills a table:
  One pass: i = 2, 3, 4, ..., n  → n-1 iterations → O(n)
  Each iteration: O(1) work

For SPACE:
  We need dp[i-1] and dp[i-2] to compute dp[i].
  After computing dp[i], we never need dp[i-2] again.
  
  Full table (O(n)):  [0, 1, 1, 2, 3, 5, 8, 13, ...]
                               ↑           ↑
                            dp[i-2]      dp[i]
  
  Space-optimized (O(1)):
    Keep only prev2 (= dp[i-2]) and prev1 (= dp[i-1])
    Compute curr = prev1 + prev2
    Shift: prev2 = prev1, prev1 = curr
    → Only 3 variables at any time → O(1) space
```

---

## The Interview Answer — When to Say What

```
INTERVIEWER: "Solve Fibonacci"

STEP 1: Write recursion first
  "The naive recursive solution directly translates the definition.
   It runs in O(2^n) time due to repeated subproblems — let me show that."
  → Write the recursive solution

STEP 2: Identify the problem
  "We're recomputing F(3) multiple times. This is an overlapping subproblems
   problem — a sign that Dynamic Programming will help."
  → Draw the recursion tree, circle the duplicates

STEP 3: Add memoization
  "The simplest fix: cache results in a memo array.
   Same recursive structure, but each subproblem solved once → O(n) time, O(n) space."
  → Add memo array to recursive solution

STEP 4: Convert to tabulation
  "Even better: eliminate recursion entirely with bottom-up DP.
   Fill a table from base cases upward → O(n) time, O(n) space, no stack risk."
  → Write the iterative tabulation

STEP 5: Space optimization
  "We can further optimize: we only need the last 2 values at each step.
   Use 3 variables instead of an array → O(n) time, O(1) space."
  → Write the 3-variable version

This shows: problem analysis → brute force → optimization journey.
Interviewers love seeing this thought process.
```

---

## Quick Cheat Sheet

```
n = 5

RECURSION calls:
  fib(5)
  fib(4), fib(3)                           [level 1: 2 calls]
  fib(3), fib(2), fib(2), fib(1)           [level 2: 4 calls]
  fib(2), fib(1), fib(1), fib(0), ...      [level 3: 8 calls]
  Total: 15 calls for n=5
  Formula: roughly 2^n calls

MEMOIZATION memo array for n=5:
  Initially: [-1, -1, -1, -1, -1, -1]
  After all:  [ 0,  1,  1,  2,  3,  5]
  Total unique calls: 6 (= n+1)

TABULATION dp array for n=5:
  Start:  [ 0,  1,  ?,  ?,  ?,  ?]
  i=2:    [ 0,  1,  1,  ?,  ?,  ?]
  i=3:    [ 0,  1,  1,  2,  ?,  ?]
  i=4:    [ 0,  1,  1,  2,  3,  ?]
  i=5:    [ 0,  1,  1,  2,  3,  5]  ← answer: dp[5] = 5

SPACE-OPTIMIZED for n=5:
  start:  prev2=0, prev1=1
  i=2:    curr=1,  prev2=1, prev1=1
  i=3:    curr=2,  prev2=1, prev1=2
  i=4:    curr=3,  prev2=2, prev1=3
  i=5:    curr=5   ← answer: 5
```

---

*Compile and run: `g++ -o fib fibonacci.cpp && ./fib`*
