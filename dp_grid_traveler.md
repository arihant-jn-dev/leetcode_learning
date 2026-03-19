# Grid Traveler Problem — Three Techniques
## Recursion vs Memoization vs Tabulation in C++

> **Problem:** You are a traveler on a 2D grid of m rows × n columns.
> You start at the **top-left** corner and must reach the **bottom-right** corner.
> You can only move **DOWN** or **RIGHT** at each step.
> How many unique paths exist to travel from start to end?

---

## Quick Comparison (Read This First)

```
TECHNIQUE       TIME          SPACE              STYLE       BEST FOR
──────────────────────────────────────────────────────────────────────────
Recursion       O(2^(m+n))    O(m+n)  [stack]    Top-down    Understanding the problem
Memoization     O(m*n)        O(m*n) + O(m+n)*   Top-down    Recursive + cache
Tabulation      O(m*n)        O(m*n)             Bottom-up   Most DP problems

* O(m*n) memo map/array + O(m+n) call stack depth
```

---

## The Problem Visualized

```
GRID 3×3: How many unique paths from S (start) to E (end)?

    col→  1     2     3
row
 ↓  1  [ S ]──►[   ]──►[   ]
           |           |           |
           ▼           ▼           ▼
    2  [   ]──►[   ]──►[   ]
           |           |           |
           ▼           ▼           ▼
    3  [   ]──►[   ]──►[ E ]

Rules: only move → (RIGHT) or ↓ (DOWN)

UNIQUE PATHS for 3×3 grid = 6:

Path 1:  → → ↓ ↓       Path 2:  → ↓ → ↓
Path 3:  → ↓ ↓ →       Path 4:  ↓ → → ↓
Path 5:  ↓ → ↓ →       Path 6:  ↓ ↓ → →

         S → · → · → ·         S → · → ·
         ↓                     ↓
         · → · → ·             · → · → ·
                 ↓                     ↓
                 ·                     ·
           (path 1)               (path 2)  ... and so on
```

**Answers for reference:**

```
gridTraveler(1, 1) = 1    (already at end — one way: stay)
gridTraveler(1, n) = 1    (only one row — must go all right, one path)
gridTraveler(m, 1) = 1    (only one col — must go all down, one path)
gridTraveler(0, n) = 0    (no grid — impossible)
gridTraveler(m, 0) = 0    (no grid — impossible)
gridTraveler(2, 3) = 3
gridTraveler(3, 2) = 3    (same as above — symmetric!)
gridTraveler(3, 3) = 6
gridTraveler(4, 4) = 20
gridTraveler(5, 5) = 70
gridTraveler(18,18) = 2333606220
```

**Why symmetric?** `gridTraveler(m, n) = gridTraveler(n, m)`
Rotating the grid 90° swaps rows and columns — paths still exist, just mirrored.

---

## The Recursive Formula

```
gridTraveler(m, n) = gridTraveler(m-1, n)   ← moved DOWN  (one fewer row)
                   + gridTraveler(m, n-1)   ← moved RIGHT (one fewer column)

BASE CASES:
  gridTraveler(1, 1) = 1    (at destination — found a valid path)
  gridTraveler(0, _) = 0    (no rows left — invalid)
  gridTraveler(_, 0) = 0    (no cols left — invalid)

INTUITION:
  Standing at top-left of a 3×3 grid.
  
  If I move DOWN  → I now have a 2×3 grid to solve = gridTraveler(2,3) = 3
  If I move RIGHT → I now have a 3×2 grid to solve = gridTraveler(3,2) = 3
  
  Total = 3 + 3 = 6 ✅
```

---

## Technique 1 — Recursion (Naive)

### The Idea

Directly call the function with reduced grid dimensions at each step.
Each call shrinks the problem: moving DOWN reduces rows by 1, moving RIGHT reduces columns by 1.

### The Recursion Tree for gridTraveler(3, 3)

```
                        gt(3,3)
                       /       \
                  gt(2,3)       gt(3,2)
                 /      \       /      \
            gt(1,3)  gt(2,2) gt(2,2)  gt(3,1)
              |       /   \   /   \      |
              1   gt(1,2)gt(2,1)gt(1,2)gt(2,1)  1
                    |      |     |      |
                   gt(1,1)gt(1,1)gt(1,1)gt(1,1)
                    1      1     1      1

gt = gridTraveler (shortened)

PROBLEM: gt(2,2) computed TWICE
         gt(1,2) computed TWICE
         gt(2,1) computed TWICE
         gt(1,1) computed FOUR times!

Same subproblem = solved multiple times = WASTED WORK

Total calls for gridTraveler(3,3)  = 27
Total calls for gridTraveler(5,5)  = 1023
Total calls for gridTraveler(10,10)= over 1 million
Total calls for gridTraveler(18,18)= astronomically large → never finishes
```

### C++ Code

```cpp
#include <iostream>
using namespace std;

// ─────────────────────────────────────────────────────────────
// TECHNIQUE 1: RECURSION (Naive)
// Time:  O(2^(m+n))
// Space: O(m+n)  [maximum call stack depth]
// ─────────────────────────────────────────────────────────────

long long gridTraveler_recursive(int m, int n) {

    // BASE CASE 1: invalid grid (no rows or no columns)
    if (m == 0 || n == 0) return 0;

    // BASE CASE 2: 1×1 grid — already at destination
    if (m == 1 && n == 1) return 1;

    // RECURSIVE CASE:
    // Move DOWN  → solve smaller grid with one fewer row
    // Move RIGHT → solve smaller grid with one fewer column
    return gridTraveler_recursive(m - 1, n)   // went DOWN
         + gridTraveler_recursive(m, n - 1);  // went RIGHT
}

int main() {
    cout << "Grid Traveler (Recursion)" << endl;
    cout << "──────────────────────────────────────" << endl;
    cout << "gridTraveler(1, 1) = " << gridTraveler_recursive(1, 1) << endl;
    cout << "gridTraveler(2, 3) = " << gridTraveler_recursive(2, 3) << endl;
    cout << "gridTraveler(3, 2) = " << gridTraveler_recursive(3, 2) << endl;
    cout << "gridTraveler(3, 3) = " << gridTraveler_recursive(3, 3) << endl;
    cout << "gridTraveler(4, 4) = " << gridTraveler_recursive(4, 4) << endl;
    cout << "gridTraveler(5, 5) = " << gridTraveler_recursive(5, 5) << endl;
    // gridTraveler_recursive(18, 18) → DO NOT RUN — takes forever

    return 0;
}
```

**Output:**
```
Grid Traveler (Recursion)
──────────────────────────────────────
gridTraveler(1, 1) = 1
gridTraveler(2, 3) = 3
gridTraveler(3, 2) = 3
gridTraveler(3, 3) = 6
gridTraveler(4, 4) = 20
gridTraveler(5, 5) = 70
```

### Complexity Analysis — Recursion

```
TIME COMPLEXITY: O(2^(m+n))
────────────────────────────
At each call, we make 2 recursive calls:
  DOWN  → (m-1, n)
  RIGHT → (m, n-1)

To reach a base case, we need (m-1) DOWN moves + (n-1) RIGHT moves.
Total moves = (m-1) + (n-1) = m+n-2

The recursion tree has depth m+n-2.
A binary tree of depth d has 2^d leaf nodes.
→ Roughly 2^(m+n) total function calls.

Real numbers:
  gridTraveler(3,3):   2^(3+3) = 2^6   = 64 (actual: 27 calls)
  gridTraveler(5,5):   2^(5+5) = 2^10  = 1024 (actual: 1023 calls)
  gridTraveler(10,10): 2^(10+10)= 2^20 = ~1 million
  gridTraveler(18,18): 2^(18+18)= 2^36 = ~68 billion ← never finishes!

SPACE COMPLEXITY: O(m+n)
─────────────────────────
The call stack depth = length of the longest path in the recursion tree.

Deepest path: always go DOWN first, then RIGHT (or vice versa):
  gt(m,n) → gt(m-1,n) → gt(m-2,n) → ... → gt(1,n)
           → gt(1,n-1) → gt(1,n-2) → ... → gt(1,1)

Total depth = (m-1) + (n-1) = m+n-2

→ At most m+n stack frames at any time → O(m+n) space.

Call stack for gt(3,3) at deepest point:
  [main]
  [gt(3,3)]
  [gt(2,3)]
  [gt(1,3)]
  [gt(1,2)]
  [gt(1,1)]  ← base case, unwinds here
```

---

## Technique 2 — Memoization (Top-Down DP)

### The Idea

**Same recursion, but CACHE each (m, n) pair so it's computed only once.**

Use a map (or 2D array) keyed by `(m, n)`.
Before computing: check if answer is already cached.
After computing: store it before returning.

### How Memoization Prunes Duplicate Branches

```
WITHOUT memo — gt(3,3) computes gt(2,2) twice:
                        gt(3,3)
                       /       \
                  gt(2,3)       gt(3,2)
                 /      \       /      \
            gt(1,3)  gt(2,2) gt(2,2)  gt(3,1)
                        ↑       ↑
                    computed  REDUNDANT!
                    first time second time

WITH memo — gt(2,2) computed once, reused:
                        gt(3,3)
                       /       \
                  gt(2,3)       gt(3,2)
                 /      \       /      \
            gt(1,3)  gt(2,2) [CACHE]  gt(3,1)
                      /   \    HIT!
                  gt(1,2)gt(2,1)
                    |       |
                  gt(1,1) gt(1,1)

memo table built during computation:
  (1,1)→1  (1,2)→1  (1,3)→1
  (2,1)→1  (2,2)→2  (2,3)→3
  (3,1)→1  (3,2)→3  (3,3)→6

Every unique (m,n) pair is computed exactly ONCE.
```

### C++ Code

```cpp
#include <iostream>
#include <map>
#include <string>
using namespace std;

// ─────────────────────────────────────────────────────────────
// TECHNIQUE 2: MEMOIZATION (Top-Down DP)
// Time:  O(m*n)        — each (m,n) pair solved once
// Space: O(m*n) + O(m+n)  — memo map + call stack
// ─────────────────────────────────────────────────────────────

// Use a map with string key "m,n" to cache results
long long gridTraveler_memo(int m, int n, map<string, long long>& memo) {

    // STEP 1: Create a unique key for this (m,n) pair
    string key = to_string(m) + "," + to_string(n);

    // STEP 2: Check cache — if already solved, return immediately
    if (memo.count(key)) {
        return memo[key];    // CACHE HIT — O(1) return
    }

    // STEP 3: Base cases
    if (m == 0 || n == 0) return 0;
    if (m == 1 && n == 1) return 1;

    // STEP 4: Compute, STORE in cache, return
    memo[key] = gridTraveler_memo(m - 1, n, memo)   // move DOWN
              + gridTraveler_memo(m, n - 1, memo);  // move RIGHT

    return memo[key];
}

int main() {
    cout << "Grid Traveler (Memoization)" << endl;
    cout << "────────────────────────────────────────────" << endl;

    map<string, long long> memo;

    // All calls share the same memo — results built up across calls
    cout << "gridTraveler(1, 1)   = " << gridTraveler_memo(1, 1, memo)   << endl;
    cout << "gridTraveler(2, 3)   = " << gridTraveler_memo(2, 3, memo)   << endl;
    cout << "gridTraveler(3, 2)   = " << gridTraveler_memo(3, 2, memo)   << endl;
    cout << "gridTraveler(3, 3)   = " << gridTraveler_memo(3, 3, memo)   << endl;
    cout << "gridTraveler(4, 4)   = " << gridTraveler_memo(4, 4, memo)   << endl;
    cout << "gridTraveler(5, 5)   = " << gridTraveler_memo(5, 5, memo)   << endl;
    cout << "gridTraveler(10, 10) = " << gridTraveler_memo(10, 10, memo) << endl;
    cout << "gridTraveler(15, 15) = " << gridTraveler_memo(15, 15, memo) << endl;
    cout << "gridTraveler(18, 18) = " << gridTraveler_memo(18, 18, memo) << endl;
    // ↑ This finishes instantly! (impossible with plain recursion)

    return 0;
}
```

**Output:**
```
Grid Traveler (Memoization)
────────────────────────────────────────────
gridTraveler(1, 1)   = 1
gridTraveler(2, 3)   = 3
gridTraveler(3, 2)   = 3
gridTraveler(3, 3)   = 6
gridTraveler(4, 4)   = 20
gridTraveler(5, 5)   = 70
gridTraveler(10, 10) = 184756
gridTraveler(15, 15) = 40116600
gridTraveler(18, 18) = 2333606220
```

### Step-by-Step Trace for gridTraveler(3, 3) with Memo

```
Call stack builds down (→ means "calls"):

gt(3,3) → key="3,3" not in memo, compute:
  gt(2,3) → key="2,3" not in memo, compute:
    gt(1,3) → key="1,3" not in memo, compute:
      gt(0,3) → returns 0   (base case)
      gt(1,2) → key="1,2" not in memo, compute:
        gt(0,2) → returns 0 (base case)
        gt(1,1) → returns 1 (base case)
        memo["1,2"] = 0+1 = 1 ✅ STORED
        return 1
      memo["1,3"] = 0+1 = 1 ✅ STORED
      return 1
    gt(2,2) → key="2,2" not in memo, compute:
      gt(1,2) → memo["1,2"] = 1 ✅ CACHE HIT! return 1
      gt(2,1) → key="2,1" not in memo, compute:
        gt(1,1) → returns 1 (base case)
        gt(2,0) → returns 0 (base case)
        memo["2,1"] = 1+0 = 1 ✅ STORED
        return 1
      memo["2,2"] = 1+1 = 2 ✅ STORED
      return 2
    memo["2,3"] = 1+2 = 3 ✅ STORED
    return 3
  gt(3,2) → key="3,2" not in memo, compute:
    gt(2,2) → memo["2,2"] = 2 ✅ CACHE HIT! return 2
    gt(3,1) → key="3,1" not in memo, compute:
      gt(2,1) → memo["2,1"] = 1 ✅ CACHE HIT! return 1
      gt(3,0) → returns 0 (base case)
      memo["3,1"] = 1+0 = 1 ✅ STORED
      return 1
    memo["3,2"] = 2+1 = 3 ✅ STORED
    return 3
  memo["3,3"] = 3+3 = 6 ✅ STORED
  return 6

CACHE HITS:
  gt(1,2) reused 1 time
  gt(2,1) reused 1 time
  gt(2,2) reused 1 time
  
Unique computations: 7   (instead of 27 without memo)
Cache hits:         3
```

### Complexity Analysis — Memoization

```
TIME COMPLEXITY: O(m * n)
──────────────────────────
How many unique (m, n) pairs can there be?
  m can be: 1, 2, 3, ..., m  → m choices
  n can be: 1, 2, 3, ..., n  → n choices
  Total unique pairs = m × n

Each pair is computed EXACTLY ONCE (then cached).
Each computation: O(1) work (one addition, one map lookup).
→ Total time = m × n × O(1) = O(m*n)

Compare:
  Recursion:   O(2^(m+n))  → billions of calls for 18×18
  Memoization: O(m*n)      → 18×18 = 324 unique pairs → instant!

SPACE COMPLEXITY: O(m*n) + O(m+n)
────────────────────────────────────
TWO sources of space:

1. MEMO MAP:
   Stores one entry per unique (m,n) pair.
   Total entries = m × n
   → O(m*n) space

2. CALL STACK:
   Recursion still happens — deepest path = m+n depth.
   At most m+n frames on stack simultaneously.
   → O(m+n) space

Total: O(m*n) + O(m+n) = O(m*n)
(O(m*n) dominates O(m+n) for typical grid sizes)
```

---

## Technique 3 — Tabulation (Bottom-Up DP)

### The Idea

**Build a 2D table filled from the top-left, where each cell = number of paths to reach it.**

- `dp[i][j]` = number of unique ways to reach cell `(i, j)` from `(1, 1)`
- Fill row by row, left to right
- Each cell pulls from the cell ABOVE it and the cell to its LEFT
- No recursion, no call stack

### Why This Formula Works

```
To reach cell (i, j), you must have come from either:
  Cell (i-1, j) — one step UP   (meaning you moved DOWN to get to (i,j))
  Cell (i, j-1) — one step LEFT (meaning you moved RIGHT to get to (i,j))

So: dp[i][j] = dp[i-1][j] + dp[i][j-1]

BASE CASES:
  dp[1][1] = 1     ← start cell, 1 way to be here
  dp[0][...] = 0   ← row 0 doesn't exist
  dp[...][0] = 0   ← col 0 doesn't exist
```

### How the Table is Filled for a 3×3 Grid

```
INITIAL TABLE (all 0, with 0-indexed border row/col as sentinels):

       col0  col1  col2  col3
row0  [  0 ] [  0 ] [  0 ] [  0 ]
row1  [  0 ] [  ? ] [  ? ] [  ? ]
row2  [  0 ] [  ? ] [  ? ] [  ? ]
row3  [  0 ] [  ? ] [  ? ] [  ? ]

STEP 1: Fill base case dp[1][1] = 1:

       col0  col1  col2  col3
row0  [  0 ] [  0 ] [  0 ] [  0 ]
row1  [  0 ] [  1 ] [  ? ] [  ? ]
row2  [  0 ] [  ? ] [  ? ] [  ? ]
row3  [  0 ] [  ? ] [  ? ] [  ? ]

STEP 2: Fill dp[1][2] = dp[0][2] + dp[1][1] = 0+1 = 1:

       col0  col1  col2  col3
row1  [  0 ] [  1 ] [  1 ] [  ? ]

STEP 3: Fill dp[1][3] = dp[0][3] + dp[1][2] = 0+1 = 1:

       col0  col1  col2  col3
row1  [  0 ] [  1 ] [  1 ] [  1 ]

  (Row 1 all 1s — makes sense: only one path along the top row, go all RIGHT)

STEP 4: Fill dp[2][1] = dp[1][1] + dp[2][0] = 1+0 = 1:

       col0  col1  col2  col3
row2  [  0 ] [  1 ] [  ? ] [  ? ]

  (Column 1 will all be 1 — only one path down the left column, go all DOWN)

STEP 5: Fill dp[2][2] = dp[1][2] + dp[2][1] = 1+1 = 2:

       col0  col1  col2  col3
row2  [  0 ] [  1 ] [  2 ] [  ? ]

STEP 6: Fill dp[2][3] = dp[1][3] + dp[2][2] = 1+2 = 3:

       col0  col1  col2  col3
row2  [  0 ] [  1 ] [  2 ] [  3 ]

STEP 7: Fill dp[3][1] = dp[2][1] + dp[3][0] = 1+0 = 1:
STEP 8: Fill dp[3][2] = dp[2][2] + dp[3][1] = 2+1 = 3:
STEP 9: Fill dp[3][3] = dp[2][3] + dp[3][2] = 3+3 = 6:

FINAL TABLE:

       col0  col1  col2  col3
row0  [  0 ] [  0 ] [  0 ] [  0 ]
row1  [  0 ] [  1 ] [  1 ] [  1 ]
row2  [  0 ] [  1 ] [  2 ] [  3 ]
row3  [  0 ] [  1 ] [  3 ] [  6 ]
                             ↑
                       Answer: dp[3][3] = 6  ✅

OBSERVE:
  First row (row1): all 1s  → only 1 path: all RIGHT
  First col (col1): all 1s  → only 1 path: all DOWN
  Interior cells: sum of cell above + cell to left
```

### C++ Code

```cpp
#include <iostream>
#include <vector>
using namespace std;

// ─────────────────────────────────────────────────────────────
// TECHNIQUE 3: TABULATION (Bottom-Up DP)
// Time:  O(m * n)
// Space: O(m * n)
// ─────────────────────────────────────────────────────────────

long long gridTraveler_tabulation(int m, int n) {

    // Create a (m+1) × (n+1) table, all zeros
    // +1 for the 0-indexed border (acts as base case boundary)
    vector<vector<long long>> dp(m + 1, vector<long long>(n + 1, 0));

    // STEP 1: Set base case — starting cell
    dp[1][1] = 1;

    // STEP 2: Fill table row by row, left to right
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {

            if (i == 1 && j == 1) continue;   // already set base case

            long long fromAbove = dp[i - 1][j];   // paths from cell above
            long long fromLeft  = dp[i][j - 1];   // paths from cell to left

            dp[i][j] = fromAbove + fromLeft;
        }
    }

    // STEP 3: Answer is at bottom-right cell
    return dp[m][n];
}

// ─────────────────────────────────────────────────────────────
// BONUS: Print the entire DP table (for learning purposes)
// ─────────────────────────────────────────────────────────────

void printTable(int m, int n) {
    vector<vector<long long>> dp(m + 1, vector<long long>(n + 1, 0));
    dp[1][1] = 1;

    for (int i = 1; i <= m; i++)
        for (int j = 1; j <= n; j++)
            if (!(i == 1 && j == 1))
                dp[i][j] = dp[i-1][j] + dp[i][j-1];

    cout << "\nDP Table for " << m << "×" << n << " grid:" << endl;
    cout << "     ";
    for (int j = 0; j <= n; j++) cout << "col" << j << "  ";
    cout << endl;

    for (int i = 0; i <= m; i++) {
        cout << "row" << i << " ";
        for (int j = 0; j <= n; j++) {
            cout << "[" << dp[i][j] << "]";
            if (dp[i][j] < 10) cout << "   ";
            else if (dp[i][j] < 100) cout << "  ";
            else cout << " ";
        }
        cout << endl;
    }
    cout << "Answer: dp[" << m << "][" << n << "] = " << dp[m][n] << endl;
}

int main() {
    cout << "Grid Traveler (Tabulation)" << endl;
    cout << "──────────────────────────────────────────" << endl;
    cout << "gridTraveler(1, 1)   = " << gridTraveler_tabulation(1, 1)   << endl;
    cout << "gridTraveler(2, 3)   = " << gridTraveler_tabulation(2, 3)   << endl;
    cout << "gridTraveler(3, 2)   = " << gridTraveler_tabulation(3, 2)   << endl;
    cout << "gridTraveler(3, 3)   = " << gridTraveler_tabulation(3, 3)   << endl;
    cout << "gridTraveler(4, 4)   = " << gridTraveler_tabulation(4, 4)   << endl;
    cout << "gridTraveler(5, 5)   = " << gridTraveler_tabulation(5, 5)   << endl;
    cout << "gridTraveler(10, 10) = " << gridTraveler_tabulation(10, 10) << endl;
    cout << "gridTraveler(15, 15) = " << gridTraveler_tabulation(15, 15) << endl;
    cout << "gridTraveler(18, 18) = " << gridTraveler_tabulation(18, 18) << endl;

    // Print the table for a 3x3 grid (educational)
    printTable(3, 3);
    printTable(4, 4);

    return 0;
}
```

**Output:**
```
Grid Traveler (Tabulation)
──────────────────────────────────────────
gridTraveler(1, 1)   = 1
gridTraveler(2, 3)   = 3
gridTraveler(3, 2)   = 3
gridTraveler(3, 3)   = 6
gridTraveler(4, 4)   = 20
gridTraveler(5, 5)   = 70
gridTraveler(10, 10) = 184756
gridTraveler(15, 15) = 40116600
gridTraveler(18, 18) = 2333606220

DP Table for 3×3 grid:
     col0  col1  col2  col3
row0 [0]   [0]   [0]   [0]
row1 [0]   [1]   [1]   [1]
row2 [0]   [1]   [2]   [3]
row3 [0]   [1]   [3]   [6]
Answer: dp[3][3] = 6

DP Table for 4×4 grid:
     col0  col1  col2  col3  col4
row0 [0]   [0]   [0]   [0]   [0]
row1 [0]   [1]   [1]   [1]   [1]
row2 [0]   [1]   [2]   [3]   [4]
row3 [0]   [1]   [3]   [6]   [10]
row4 [0]   [1]   [4]   [10]  [20]
Answer: dp[4][4] = 20
```

### Complexity Analysis — Tabulation

```
TIME COMPLEXITY: O(m * n)
──────────────────────────
Two nested loops:
  Outer loop: i = 1 to m  → m iterations
  Inner loop: j = 1 to n  → n iterations per outer loop
  Each cell: O(1) work (one addition, two lookups)

Total: m × n × O(1) = O(m*n)

SPACE COMPLEXITY: O(m * n)
────────────────────────────
dp table is (m+1) × (n+1) in size.
→ (m+1)(n+1) ≈ m*n entries → O(m*n)

NO CALL STACK (no recursion) → no stack space used.
This means no risk of stack overflow for any grid size.

COMPARISON:
  Recursion:   O(2^(m+n)) time → impossible for large grids
  Memoization: O(m*n) time but O(m+n) stack space (overflow risk)
  Tabulation:  O(m*n) time, O(m*n) space, NO stack = SAFEST
```

---

## All Three Together — One Complete File

```cpp
#include <iostream>
#include <vector>
#include <map>
#include <string>
using namespace std;

// ════════════════════════════════════════════════════════════
// TECHNIQUE 1: RECURSION
// Time: O(2^(m+n))   Space: O(m+n) call stack
// ════════════════════════════════════════════════════════════
long long gt_recursive(int m, int n) {
    if (m == 0 || n == 0) return 0;
    if (m == 1 && n == 1) return 1;
    return gt_recursive(m - 1, n) + gt_recursive(m, n - 1);
}

// ════════════════════════════════════════════════════════════
// TECHNIQUE 2: MEMOIZATION (Top-Down DP)
// Time: O(m*n)   Space: O(m*n) + O(m+n) stack
// ════════════════════════════════════════════════════════════
long long gt_memo(int m, int n, map<string, long long>& memo) {
    string key = to_string(m) + "," + to_string(n);
    if (memo.count(key)) return memo[key];    // cache hit
    if (m == 0 || n == 0) return 0;
    if (m == 1 && n == 1) return 1;
    memo[key] = gt_memo(m - 1, n, memo) + gt_memo(m, n - 1, memo);
    return memo[key];
}

// ════════════════════════════════════════════════════════════
// TECHNIQUE 3: TABULATION (Bottom-Up DP)
// Time: O(m*n)   Space: O(m*n)
// ════════════════════════════════════════════════════════════
long long gt_tabulation(int m, int n) {
    vector<vector<long long>> dp(m + 1, vector<long long>(n + 1, 0));
    dp[1][1] = 1;
    for (int i = 1; i <= m; i++)
        for (int j = 1; j <= n; j++)
            if (!(i == 1 && j == 1))
                dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
    return dp[m][n];
}

// ════════════════════════════════════════════════════════════
// MAIN — Compare all techniques
// ════════════════════════════════════════════════════════════
int main() {
    cout << "m×n      Recursive    Memoized     Tabulation" << endl;
    cout << "──────────────────────────────────────────────" << endl;

    // Test cases (keep recursive small — it's slow for large grids)
    vector<pair<int,int>> tests = {
        {1,1}, {1,4}, {4,1}, {2,3}, {3,2},
        {3,3}, {4,4}, {5,5}
    };

    for (auto [m, n] : tests) {
        map<string, long long> memo;
        cout << m << "×" << n << "      "
             << gt_recursive(m, n) << "            "
             << gt_memo(m, n, memo) << "            "
             << gt_tabulation(m, n)
             << endl;
    }

    cout << "\nLarge grids (only memoization and tabulation):" << endl;
    cout << "───────────────────────────────────────────────" << endl;

    vector<pair<int,int>> large = { {10,10}, {15,15}, {18,18} };
    for (auto [m, n] : large) {
        map<string, long long> memo;
        cout << m << "×" << n
             << "  Memo="      << gt_memo(m, n, memo)
             << "  Tab="       << gt_tabulation(m, n)
             << endl;
    }

    return 0;
}
```

**Output:**
```
m×n      Recursive    Memoized     Tabulation
──────────────────────────────────────────────
1×1      1            1            1
1×4      1            1            1
4×1      1            1            1
2×3      3            3            3
3×2      3            3            3
3×3      6            6            6
4×4      20           20           20
5×5      70           70           70

Large grids (only memoization and tabulation):
───────────────────────────────────────────────
10×10  Memo=184756       Tab=184756
15×15  Memo=40116600     Tab=40116600
18×18  Memo=2333606220   Tab=2333606220
```

---

## Complexity Summary — Side by Side

```
┌──────────────────┬─────────────────┬─────────────────────────────────────────┐
│ Technique        │      TIME       │                 SPACE                   │
├──────────────────┼─────────────────┼─────────────────────────────────────────┤
│ Recursion        │  O(2^(m+n))     │ O(m+n) — call stack depth               │
│                  │                 │ WARNING: Unusably slow for large grids  │
├──────────────────┼─────────────────┼─────────────────────────────────────────┤
│ Memoization      │  O(m * n)       │ O(m*n)  — memo map                      │
│ (Top-Down DP)    │                 │ O(m+n)  — call stack                    │
│                  │                 │ Total: O(m*n)                           │
│                  │                 │ WARNING: Stack overflow for huge grids  │
├──────────────────┼─────────────────┼─────────────────────────────────────────┤
│ Tabulation       │  O(m * n)       │ O(m*n)  — 2D dp table                  │
│ (Bottom-Up DP)   │                 │ O(1)    — no call stack                 │
│                  │                 │ Total: O(m*n)                           │
│                  │                 │ SAFE for any grid size                  │
└──────────────────┴─────────────────┴─────────────────────────────────────────┘
```

---

## Understanding WHY the Complexities Are What They Are

### Why Recursion is O(2^(m+n))

```
At every call, 2 branches are created:
  gt(m, n)
  ├── gt(m-1, n)   [move DOWN]
  └── gt(m, n-1)  [move RIGHT]

To reach a base case (1,1) or (0,x) or (x,0):
  Need at most (m-1) DOWN steps + (n-1) RIGHT steps.
  Maximum path length = m+n-2.

Binary tree with maximum depth (m+n-2):
  Maximum nodes = 2^(m+n-2) ≈ O(2^(m+n))

For gt(18,18): 2^(18+18) = 2^36 ≈ 68 billion calls → not feasible.
```

### Why Memoization and Tabulation are both O(m*n)

```
KEY INSIGHT: How many UNIQUE subproblems exist?

Every recursive call is for some (i, j) where:
  1 ≤ i ≤ m
  1 ≤ j ≤ n

All possible pairs (i, j):
  (1,1), (1,2), ..., (1,n)
  (2,1), (2,2), ..., (2,n)
  ...
  (m,1), (m,2), ..., (m,n)

Total unique pairs = m × n

With caching (memo) or a table:
  Each of the m×n subproblems solved ONCE
  Each takes O(1) to compute once subproblems below it are done
  Total: m×n × O(1) = O(m*n)

For gt(18,18): 18×18 = 324 unique subproblems → instant! ✅
```

### Why Tabulation Has No Stack Space (Unlike Memoization)

```
MEMOIZATION: Still uses recursion
  gt(3,3) calls gt(2,3) calls gt(1,3) calls gt(1,2) calls gt(1,1)
  At deepest point: 5 frames on call stack (for 3×3)
  For m×n grid: up to m+n frames on stack simultaneously.
  Stack overflow possible for very large n.

TABULATION: Pure iteration
  for i in 1..m:
    for j in 1..n:
      dp[i][j] = dp[i-1][j] + dp[i][j-1]
  
  No function calls. No call stack. Just two nested loops.
  1,000×1,000 grid? Fine. 10,000×10,000? Fine.
  Stack space used: O(1) (just loop variables i and j).
```

---

## Visualizing the Path Count Pattern

```
NOTICE THE PATTERN in dp tables:

3×3:              4×4:              5×5:
[1] [1] [1]       [1] [1] [1] [1]   [1] [1] [1]  [1]  [1]
[1] [2] [3]       [1] [2] [3] [4]   [1] [2] [3]  [4]  [5]
[1] [3] [6]       [1] [3] [6] [10]  [1] [3] [6]  [10] [15]
                  [1] [4] [10][20]  [1] [4] [10] [20] [35]
                                    [1] [5] [15] [35] [70]

Pascal's Triangle! The diagonal of Pascal's Triangle!

dp[m][n] = C(m+n-2, m-1)   (combination formula)

For gt(3,3): C(3+3-2, 3-1) = C(4,2) = 4!/(2!×2!) = 6  ✅
For gt(4,4): C(4+4-2, 4-1) = C(6,3) = 6!/(3!×3!) = 20 ✅
For gt(5,5): C(5+5-2, 5-1) = C(8,4) = 8!/(4!×4!) = 70 ✅

This makes mathematical sense:
  Total moves = (m-1) DOWN + (n-1) RIGHT = m+n-2 moves
  We need to CHOOSE which (m-1) of those moves are DOWN
  Number of ways = C(m+n-2, m-1)
```

---

## The Interview Answer — What to Say and When

```
INTERVIEWER: "Solve the Grid Traveler problem"

STEP 1: Understand and explain the problem
  "We're on an m×n grid, start top-left, end bottom-right.
   Can only move right or down. Count unique paths."
  
  Draw a 2×3 example. Show paths. Say answer is 3.

STEP 2: Write recursive solution
  "The recursive formula: gt(m,n) = gt(m-1,n) + gt(m,n-1)
   At each cell, we could have come from above or from the left."
  → Write 10 lines of recursive code

  "Time complexity: O(2^(m+n)) — exponential. For large grids, this fails."

STEP 3: Identify overlapping subproblems
  "Draw the recursion tree — gt(2,2) appears multiple times.
   Same subproblems solved repeatedly — overlapping subproblems.
   Dynamic Programming can fix this."
  → Show the tree, circle the duplicates

STEP 4: Add memoization
  "Cache results in a map/array by (m,n) key.
   Each subproblem solved once → O(m*n) time."
  → Add memo map to recursive solution

STEP 5: Convert to tabulation
  "Eliminate recursion entirely. Fill a 2D table from top-left.
   dp[i][j] = dp[i-1][j] + dp[i][j-1]
   Same time complexity O(m*n), but no stack space risk."
  → Write the nested loop version

BONUS (if asked about space):
  "Could we optimize space? Yes — we only ever read the current row
   and the row above. So we can use just two 1D arrays (or even one
   with careful ordering), reducing space to O(n)."

This progression shows: problem → brute force → insight → DP.
```

---

## Quick Cheat Sheet

```
PROBLEM: gridTraveler(m, n) = paths from (1,1) to (m,n) going only → or ↓

FORMULA: gt(m, n) = gt(m-1, n) + gt(m, n-1)
BASE:    gt(1,1)=1,  gt(0,_)=0,  gt(_,0)=0

──────────────────────────────────────────────────────────────

RECURSION:
  if(m==0||n==0) return 0;
  if(m==1&&n==1) return 1;
  return rec(m-1,n) + rec(m,n-1);
  
  Time: O(2^(m+n))   Space: O(m+n) stack

──────────────────────────────────────────────────────────────

MEMOIZATION:
  string key = to_string(m)+","+to_string(n);
  if(memo.count(key)) return memo[key];
  ... [same as recursion]
  return memo[key] = memo(m-1,n) + memo(m,n-1);
  
  Time: O(m*n)   Space: O(m*n) + O(m+n) stack

──────────────────────────────────────────────────────────────

TABULATION:
  dp[1][1] = 1;
  for i in 1..m:
    for j in 1..n:
      dp[i][j] = dp[i-1][j] + dp[i][j-1];  (skip [1][1])
  return dp[m][n];
  
  Time: O(m*n)   Space: O(m*n) — NO STACK

──────────────────────────────────────────────────────────────

ANSWERS:
  gt(1,1)=1   gt(2,3)=3   gt(3,3)=6   gt(4,4)=20
  gt(5,5)=70  gt(10,10)=184756        gt(18,18)=2333606220
```

---

*Compile and run: `g++ -o grid grid_traveler.cpp && ./grid`*
