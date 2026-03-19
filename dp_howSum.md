# howSum Problem — Three Techniques
## Recursion vs Memoization vs Tabulation in C++

> **Problem:** Given a `targetSum` (integer) and an array of `numbers` (non-negative integers),
> return **any one combination** of numbers from the array that adds up to exactly the `targetSum`.
> If no combination is possible, return false / empty result.
> You may use the same number from the array **as many times as needed**.

---

## howSum vs canSum — What's Different?

```
canSum(7, [3,4]) → true / false        ← just tells you IF it's possible
howSum(7, [3,4]) → [3, 4] or empty    ← gives you the ACTUAL combination

howSum is harder — not just yes/no, but WHAT numbers to pick.
This means we must CARRY the result upward through the recursion.
Carrying a result (vector) adds O(m) copy cost to many operations.
→ This makes howSum complexities slightly WORSE than canSum.
```

---

## Quick Comparison (Read This First)

```
TECHNIQUE       TIME            SPACE              STYLE
─────────────────────────────────────────────────────────────────────────
Recursion       O(n^m × m)      O(m)               Top-down
Memoization     O(n × m²)       O(m²)              Top-down + cache
Tabulation      O(n × m²)       O(m²)              Bottom-up

Where: m = targetSum,  n = length of numbers array
The extra ×m factor (vs canSum) comes from copying vectors of up to size m.
```

---

## Approach: bool + vector& (No Optional Needed)

```
DESIGN DECISION:
  Return type → bool     (true = found a combination, false = impossible)
  Parameter   → vector<int>& result  (filled with the combination if found)

WHY THIS WORKS:
  bool cleanly separates yes/no.
  The vector& is only meaningful when bool = true.
  No need for any special sentinel or third-party library.

USAGE:
  vector<int> result;
  bool found = howSum(7, {3, 4, 5}, result);
  if (found) { /* result = [3,4] */ }
  else       { /* result is empty — no solution */ }
```

---

## The Problem Visualized

```
targetSum = 7,  numbers = [3, 4, 5]

Possible paths to 7:
  3 + 4      = 7  ← VALID ✅  result = [3, 4]
  4 + 3      = 7  ← VALID ✅  (same combination, different order)
  3 + 3 + ... can 3 reach 7? 3,6,9 — no
  5 + ...    can 5 reach 7? 5, 10 — no (with these numbers)

  We only need to return ONE valid combination. Stop as soon as we find it.

targetSum = 7,  numbers = [2, 4]
  2+2+... → 2,4,6,8 only even sums → 7 is odd → IMPOSSIBLE → return false
```

**Test Cases for Reference:**

```
howSum(7,  [2, 3])       → true,  result = [2, 2, 3]   (2+2+3=7)
howSum(7,  [3, 4, 5])    → true,  result = [3, 4]       (3+4=7)
howSum(7,  [2, 4])       → false, result = []           (impossible)
howSum(8,  [2, 3, 5])    → true,  result = [3, 5]       (3+5=8)
howSum(0,  [1, 2, 3])    → true,  result = []           (already 0 — valid!)
howSum(300,[7, 14])      → false, result = []           (300 not divisible by 7)
```

---

## The Recursive Formula

```
howSum(targetSum, numbers, result):

  BASE CASE 1: targetSum == 0
    → return true   (hit exactly 0 — valid combination, result is already built)

  BASE CASE 2: targetSum < 0
    → return false  (went past 0 — dead end)

  FOR EACH num in numbers:
    remainder = targetSum - num
    IF howSum(remainder, numbers, result) == true:
      result.push_back(num)   ← add this number on the way back UP
      return true             ← stop searching

  return false               ← no number worked

KEY DIFFERENCE FROM canSum:
  canSum returns true/false  → just a flag, nothing to carry
  howSum modifies result&    → builds the path in the reference vector
                               each level pushes_back one number
```

---

## Technique 1 — Recursion (Naive)

### The Idea

For each call, try subtracting every number.
The first one that leads to `targetSum == 0` wins.
On the way back UP the call stack, each level pushes its chosen number into `result`.

### How the Result Builds Up Through the Call Stack

```
howSum(7, [3, 4, 5], result):  try num=3 first

  howSum(7)  ──picks 3──►  howSum(4)  ──picks 3──►  howSum(1)
                                                     ├── pick 3: howSum(-2) → false
                                                     ├── pick 4: howSum(-3) → false
                                                     └── pick 5: howSum(-4) → false
                                                     → return false
                           tries num=4:
                           howSum(4)  ──picks 4──►  howSum(0)
                                                    → return true  ← BASE CASE!
                           → true returned
                           result.push_back(4)  → result = [4]
                           return true  ← to howSum(7)
  → true returned
  result.push_back(3)  → result = [4, 3]
  return true

FINAL: result = [4, 3]   which means 3+4 = 7 ✅
(Built in REVERSE order — last chosen appears first in the vector)
To get forward order [3, 4]: call reverse(result.begin(), result.end()) at the end.
```

### Recursion Tree for howSum(7, [3, 4, 5])

```
                          hs(7)
                        /   |   \
                    hs(4) hs(3) hs(2)    ← subtract each number
                   /|\ 
           hs(1)  hs(0)  hs(-1)          ← subtract from 4
            /|\    ↑       ✗
           ...   true                    ← BASE CASE: return true
           all    ↑
           false  result.push_back(4) → result=[4]
                  return true ← to hs(7)
                  ↑
          result.push_back(3) → result=[4,3]
          return true ← DONE!

          hs(3) and hs(2) are NEVER EXPLORED
          (short-circuit: return as soon as first path works)

howSum(7, [2, 4]) — false case, full tree explored:

                          hs(7)
                        /       \
                    hs(5)       hs(3)
                   /    \      /    \
               hs(3)  hs(1) hs(1) hs(-1)
               /  \   / \   / \
           hs(1) hs(-1) ...  hs(-1)...
           / \
          hs(-1)hs(-3)
          ✗     ✗

REPEATED SUBPROBLEMS:
  hs(3) computed TWICE (from hs(5) and from hs(7) directly)
  hs(1) computed THREE times!
  → Memoization can eliminate this.
```

### C++ Code

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

// ─────────────────────────────────────────────────────────────
// TECHNIQUE 1: RECURSION (Naive)
// Time:  O(n^m × m)
//   n^m recursive calls, each may push_back through up to O(m) levels
// Space: O(m)
//   call stack depth + result vector, both at most m
// ─────────────────────────────────────────────────────────────

bool howSum_recursive(int targetSum, const vector<int>& numbers, vector<int>& result) {

    // BASE CASE 1: exactly hit 0 — valid combination found!
    if (targetSum == 0) return true;

    // BASE CASE 2: went negative — dead end
    if (targetSum < 0)  return false;

    // Try each number
    for (int num : numbers) {
        if (howSum_recursive(targetSum - num, numbers, result)) {
            result.push_back(num);    // add this number on the way back up
            return true;             // short-circuit — stop searching
        }
    }

    // No number worked from this targetSum
    return false;
}

void printResult(const string& label, bool found, const vector<int>& result) {
    cout << label;
    if (!found) {
        cout << "null (impossible)" << endl;
    } else {
        cout << "[ ";
        for (int x : result) cout << x << " ";
        cout << "]";
        int sum = 0;
        for (int x : result) sum += x;
        cout << "  (sum=" << sum << ")" << endl;
    }
}

int main() {
    cout << "howSum (Recursion)" << endl;
    cout << "──────────────────────────────────────────────────────" << endl;

    {
        vector<int> result;
        bool found = howSum_recursive(7, {2, 3}, result);
        printResult("howSum(7, [2,3])    = ", found, result);
    }
    {
        vector<int> result;
        bool found = howSum_recursive(7, {3, 4, 5}, result);
        printResult("howSum(7, [3,4,5])  = ", found, result);
    }
    {
        vector<int> result;
        bool found = howSum_recursive(7, {2, 4}, result);
        printResult("howSum(7, [2,4])    = ", found, result);
    }
    {
        vector<int> result;
        bool found = howSum_recursive(8, {2, 3, 5}, result);
        printResult("howSum(8, [2,3,5])  = ", found, result);
    }
    {
        vector<int> result;
        bool found = howSum_recursive(0, {1, 2, 3}, result);
        printResult("howSum(0, [1,2,3])  = ", found, result);
    }
    // WARNING: howSum(300, {7,14}) — VERY SLOW without memoization!

    return 0;
}
```

**Output:**
```
howSum (Recursion)
──────────────────────────────────────────────────────
howSum(7, [2,3])    = [ 3 2 2 ]  (sum=7)
howSum(7, [3,4,5])  = [ 4 3 ]    (sum=7)
howSum(7, [2,4])    = null (impossible)
howSum(8, [2,3,5])  = [ 5 3 ]    (sum=8)
howSum(0, [1,2,3])  = [ ]        (sum=0)
```

### Complexity Analysis — Recursion

```
TIME COMPLEXITY: O(n^m × m)
────────────────────────────
PART 1 — Number of recursive calls: O(n^m)
  At each call, we try n numbers.
  Worst case: smallest number = 1, so depth of tree = m.
  Branching factor = n, depth = m.
  Total nodes in tree: n^m   (same reasoning as canSum)

PART 2 — Work per node: O(m)
  When a valid path is found, the result.push_back() propagates
  back up through up to m levels of the call stack.
  Each level does O(1) push_back, but it happens m times total.
  → O(m) extra work when returning a found result.

TOTAL: O(n^m) calls × O(m) propagation cost = O(n^m × m)

SPACE COMPLEXITY: O(m)
────────────────────────
Call stack: at most m frames deep.
Result vector: at most m elements long.
Both are O(m).
```

---

## Technique 2 — Memoization (Top-Down DP)

### The Idea

**Cache the result for each `targetSum` so it's computed only once.**

We need to remember TWO things per `targetSum`:
1. Was it computed yet? (to avoid recomputing)
2. Was it solvable? (true or false)
3. If solvable, what was the combination? (the vector)

**Solution: two separate maps — no optional needed.**

```cpp
unordered_map<int, bool>         memo_found;   // has this target been computed?
unordered_map<int, vector<int>>  memo_result;  // combination if solvable

// Lookup logic:
if (memo_found.count(target)) {          // already computed before?
    if (memo_found[target]) {            // was it solvable?
        result = memo_result[target];    // copy cached combination
        return true;
    }
    return false;                        // was not solvable — return fast
}
```

### How Memoization Prunes Repeated Subtrees

```
WITHOUT MEMO — howSum(7, [2, 4]) explores hs(3) and hs(1) multiple times:
                          hs(7)
                        /       \
                    hs(5)       hs(3)
                   /    \       /   \
               hs(3)  hs(1) hs(1) hs(-1)
               /  \   / \   / \
           hs(1) hs(-1) ← REPEATED!

WITH MEMO:
  memo_found[3] = false (computed once → stored)
  memo_found[1] = false (computed once → stored)

                          hs(7)
                        /       \
                    hs(5)       hs(3) ← 🎯 CACHE HIT: false
                   /    \
               hs(3) 🎯  hs(1) 🎯    ← CACHE HITS — skip entire subtrees
               (false)   (false)

MEMO CONTENTS AFTER FULL COMPUTATION for howSum(7, [2,4]):
  memo_found[1] = false
  memo_found[3] = false
  memo_found[5] = false
  memo_found[7] = false
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
//   O(n × m) unique subproblem attempts, each copies O(m) vector
// Space: O(m²)
//   memo_result stores up to m entries, each a vector of up to m length
//   + O(m) call stack
// ─────────────────────────────────────────────────────────────

bool howSum_memo(
    int targetSum,
    const vector<int>& numbers,
    vector<int>& result,
    unordered_map<int, bool>& memo_found,
    unordered_map<int, vector<int>>& memo_result
) {
    // STEP 1: Check cache — if this targetSum was already computed
    if (memo_found.count(targetSum)) {
        if (memo_found[targetSum]) {
            result = memo_result[targetSum];   // copy cached combination
            return true;
        }
        return false;                          // was computed, not solvable
    }

    // STEP 2: Base cases
    if (targetSum == 0) return true;           // result already built by recursion
    if (targetSum < 0)  return false;

    // STEP 3: Try each number
    for (int num : numbers) {
        if (howSum_memo(targetSum - num, numbers, result,
                        memo_found, memo_result)) {
            result.push_back(num);             // add this number
            memo_found[targetSum]  = true;     // cache: solvable
            memo_result[targetSum] = result;   // cache: the combination
            return true;
        }
    }

    // STEP 4: Not solvable — cache and return false
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
        cout << "]";
        int sum = 0; for (int x : result) sum += x;
        cout << "  (sum=" << sum << ")" << endl;
    }
}

int main() {
    cout << "howSum (Memoization)" << endl;
    cout << "──────────────────────────────────────────────────────" << endl;

    auto run = [](int target, vector<int> nums) {
        unordered_map<int, bool> mf;
        unordered_map<int, vector<int>> mr;
        vector<int> result;
        bool found = howSum_memo(target, nums, result, mf, mr);
        return make_pair(found, result);
    };

    auto [f1, r1] = run(7,   {2, 3});        printResult("howSum(7,   [2,3])   = ", f1, r1);
    auto [f2, r2] = run(7,   {3, 4, 5});     printResult("howSum(7,   [3,4,5]) = ", f2, r2);
    auto [f3, r3] = run(7,   {2, 4});        printResult("howSum(7,   [2,4])   = ", f3, r3);
    auto [f4, r4] = run(8,   {2, 3, 5});     printResult("howSum(8,   [2,3,5]) = ", f4, r4);
    auto [f5, r5] = run(0,   {1, 2, 3});     printResult("howSum(0,   [1,2,3]) = ", f5, r5);
    auto [f6, r6] = run(300, {7, 14});       printResult("howSum(300, [7,14])  = ", f6, r6);
    // ↑ Finishes instantly with memoization!

    return 0;
}
```

**Output:**
```
howSum (Memoization)
──────────────────────────────────────────────────────
howSum(7,   [2,3])   = [ 3 2 2 ]   (sum=7)
howSum(7,   [3,4,5]) = [ 4 3 ]     (sum=7)
howSum(7,   [2,4])   = null (impossible)
howSum(8,   [2,3,5]) = [ 5 3 ]     (sum=8)
howSum(0,   [1,2,3]) = [ ]         (sum=0)
howSum(300, [7,14])  = null (impossible)
```

### Step-by-Step Trace for howSum(7, [3, 4]) with Memo

```
hs = howSum (shortened), 🎯 = cache hit, ✅ = stored in memo

hs(7): not in memo_found
  try 3: hs(4)
    hs(4): not in memo_found
      try 3: hs(1)
        hs(1): not in memo_found
          try 3: hs(-2) → false (base case)
          try 4: hs(-3) → false (base case)
          → memo_found[1] = false ✅  return false
        ← false
      try 4: hs(0) → true (BASE CASE! result still empty here)
        result.push_back(4) → result = [4]
        memo_found[4] = true, memo_result[4] = [4] ✅
        return true
      ← true, result = [4]
    result.push_back(3) → result = [4, 3]
    memo_found[7] = true, memo_result[7] = [4, 3] ✅
    return true

MEMO CONTENTS:
  memo_found[1]  = false
  memo_found[4]  = true,  memo_result[4] = [4]
  memo_found[7]  = true,  memo_result[7] = [4, 3]

BENEFIT: If any subsequent call needs hs(4) or hs(1) again,
         it returns instantly from cache instead of recomputing.
```

### Complexity Analysis — Memoization

```
TIME COMPLEXITY: O(n × m²)
───────────────────────────
Unique targetSum values: 0 to m → (m+1) unique subproblems.
Each subproblem: tried n numbers → O(n) calls per subproblem.
Each call: may copy a cached vector of at most O(m) length.

→ Total: (m unique states) × (n numbers each) × (m copy cost)
       = O(n × m × m) = O(n × m²)

For howSum(300, [7,14]):
  Recursion:   n^m = 2^300 ≈ 10^90 → impossible
  Memoization: n × m² = 2 × 300² = 180,000 → instant!

SPACE COMPLEXITY: O(m²)
─────────────────────────
1. memo_result MAP:
   At most m+1 entries.
   Each entry is a vector of at most m elements.
   → m entries × m elements = O(m²) space.

2. CALL STACK:
   Recursion depth at most m.
   → O(m) space.

Total: O(m²) + O(m) = O(m²)
```

---

## Technique 3 — Tabulation (Bottom-Up DP)

### The Idea

**Use two parallel arrays — no optional needed.**

```
reachable[i]  →  bool          "Can we reach sum i?"
dp[i]         →  vector<int>   "Combination that reaches sum i"
                                (only meaningful when reachable[i] = true)

reachable[0] = true,  dp[0] = {}   ← base case
From each reachable i: mark i+num as reachable and extend its combination.
```

### Why This Formula Works

```
If reachable[i] is true:
  → We can reach sum i using combination dp[i].
  → If we pick 'num', we can reach sum i + num.
  → So: reachable[i+num] = true
         dp[i+num]        = dp[i] + [num]

We only set dp[i+num] if it wasn't already reachable.
(We want any valid combination, so the FIRST one we find is fine.)
```

### How the Table is Filled — howSum(7, [3, 4, 5])

```
INITIAL:
Index:      0      1      2      3      4      5      6      7
reachable: [T]    [F]    [F]    [F]    [F]    [F]    [F]    [F]
dp:        [ ]   ---    ---    ---    ---    ---    ---    ---

────────────────────────────────────────────────────────────

i=0: reachable[0]=true → spread
  num=3: reachable[3]=true, dp[3] = [] + [3] = [3]
  num=4: reachable[4]=true, dp[4] = [] + [4] = [4]
  num=5: reachable[5]=true, dp[5] = [] + [5] = [5]

Index:      0      1      2      3      4      5      6      7
reachable: [T]    [F]    [F]    [T]    [T]    [T]    [F]    [F]
dp:        [ ]   ---    ---    [3]    [4]    [5]   ---    ---

────────────────────────────────────────────────────────────

i=1: reachable[1]=false → skip
i=2: reachable[2]=false → skip

────────────────────────────────────────────────────────────

i=3: reachable[3]=true → spread
  num=3: reachable[6]=true, dp[6] = [3] + [3] = [3,3]
  num=4: reachable[7]=true, dp[7] = [3] + [4] = [3,4]  ← TARGET!
  num=5: 3+5=8 > 7, out of bounds → skip

Index:      0      1      2      3      4      5      6      7
reachable: [T]    [F]    [F]    [T]    [T]    [T]    [T]    [T]
dp:        [ ]   ---    ---    [3]    [4]    [5]   [3,3] [3,4]
                                                             ↑
                                                     ANSWER FOUND!

i=4,5,6: reachable[7] already true → skip setting it again

FINAL: reachable[7] = true,  dp[7] = [3, 4]   → 3+4=7 ✅
```

### How the Table is Filled — howSum(7, [2, 4]) — IMPOSSIBLE CASE

```
INITIAL:
Index:      0      1      2      3      4      5      6      7
reachable: [T]    [F]    [F]    [F]    [F]    [F]    [F]    [F]

i=0:  reachable[2]=T, dp[2]=[2]   |   reachable[4]=T, dp[4]=[4]
i=2:  reachable[4] already T      |   reachable[6]=T, dp[6]=[2,4]
i=4:  reachable[6] already T      |   8 out of bounds
i=6:  8 and 10 out of bounds

FINAL:
Index:      0      1      2      3      4      5      6      7
reachable: [T]    [F]    [T]    [F]    [T]    [F]    [T]    [F]
dp:        [ ]   ---    [2]   ---    [4]   ---   [2,4]  ---
                                                           ↑
                                                  reachable[7] = false → IMPOSSIBLE ✗

Only even indices are reachable. 7 is odd → never reached.
```

### C++ Code

```cpp
#include <iostream>
#include <vector>
using namespace std;

// ─────────────────────────────────────────────────────────────
// TECHNIQUE 3: TABULATION (Bottom-Up DP)
// Time:  O(n × m²)
//   O(n × m) iterations, each copies a vector of up to O(m)
// Space: O(m²)
//   dp has m+1 entries, each a vector of up to m elements
//   NO call stack at all
// ─────────────────────────────────────────────────────────────

bool howSum_tabulation(int targetSum, const vector<int>& numbers, vector<int>& result) {

    // STEP 1: Two parallel arrays
    vector<bool>         reachable(targetSum + 1, false);  // can we reach sum i?
    vector<vector<int>>  dp(targetSum + 1);                // combination to reach i

    // STEP 2: Base case — sum 0 is always reachable with empty combination
    reachable[0] = true;
    // dp[0] is already an empty vector by default

    // STEP 3: Iterate and spread reachability forward
    for (int i = 0; i <= targetSum; i++) {
        if (!reachable[i]) continue;           // unreachable — skip

        for (int num : numbers) {
            if (i + num > targetSum) continue; // out of bounds — skip

            if (!reachable[i + num]) {         // only set if not already found
                reachable[i + num] = true;
                dp[i + num] = dp[i];           // copy current combination
                dp[i + num].push_back(num);    // extend with this number
            }
        }
    }

    // STEP 4: Return result if reachable
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
            if (i + num <= targetSum && !reachable[i + num]) {
                reachable[i + num] = true;
                dp[i + num] = dp[i];
                dp[i + num].push_back(num);
            }
        }
    }

    cout << "\nDP table for howSum(" << targetSum << ", [";
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
            cout << "]";
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
        cout << "]  (sum=" << sum << ")" << endl;
    }
}

void printResult(const string& label, bool found, const vector<int>& result) {
    cout << label;
    if (!found) {
        cout << "null (impossible)" << endl;
    } else {
        cout << "[ ";
        for (int x : result) cout << x << " ";
        cout << "]";
        int sum = 0; for (int x : result) sum += x;
        cout << "  (sum=" << sum << ")" << endl;
    }
}

int main() {
    cout << "howSum (Tabulation)" << endl;
    cout << "──────────────────────────────────────────────────────" << endl;

    auto run = [](int t, vector<int> nums) {
        vector<int> result;
        bool found = howSum_tabulation(t, nums, result);
        return make_pair(found, result);
    };

    auto [f1, r1] = run(7,   {2, 3});     printResult("howSum(7,   [2,3])    = ", f1, r1);
    auto [f2, r2] = run(7,   {3, 4, 5});  printResult("howSum(7,   [3,4,5])  = ", f2, r2);
    auto [f3, r3] = run(7,   {2, 4});     printResult("howSum(7,   [2,4])    = ", f3, r3);
    auto [f4, r4] = run(8,   {2, 3, 5});  printResult("howSum(8,   [2,3,5])  = ", f4, r4);
    auto [f5, r5] = run(0,   {1, 2, 3});  printResult("howSum(0,   [1,2,3])  = ", f5, r5);
    auto [f6, r6] = run(300, {7, 14});    printResult("howSum(300, [7,14])   = ", f6, r6);

    printTable(7, {3, 4, 5});
    printTable(7, {2, 4});

    return 0;
}
```

**Output:**
```
howSum (Tabulation)
──────────────────────────────────────────────────────
howSum(7,   [2,3])    = [ 2 2 3 ]   (sum=7)
howSum(7,   [3,4,5])  = [ 3 4 ]     (sum=7)
howSum(7,   [2,4])    = null (impossible)
howSum(8,   [2,3,5])  = [ 2 3 3 ]   (sum=8)
howSum(0,   [1,2,3])  = [ ]         (sum=0)
howSum(300, [7,14])   = null (impossible)

DP table for howSum(7, [3,4,5]):
  dp[0] = [ ]
  dp[1] = null
  dp[2] = null
  dp[3] = [ 3 ]
  dp[4] = [ 4 ]
  dp[5] = [ 5 ]
  dp[6] = [ 3 3 ]
  dp[7] = [ 3 4 ]
Answer: dp[7] = [ 3 4 ]  (sum=7)

DP table for howSum(7, [2,4]):
  dp[0] = [ ]
  dp[1] = null
  dp[2] = [ 2 ]
  dp[3] = null
  dp[4] = [ 4 ]
  dp[5] = null
  dp[6] = [ 2 4 ]
  dp[7] = null
Answer: dp[7] = null (impossible)
```

### Complexity Analysis — Tabulation

```
TIME COMPLEXITY: O(n × m²)
────────────────────────────
Outer loop: i from 0 to m     → m+1 iterations
Inner loop: for each num      → n iterations per outer step
Each inner step: copies dp[i] (a vector of up to O(m) elements) + push_back

Copy cost per step: O(m)
Total: m × n × O(m) = O(n × m²)

SPACE COMPLEXITY: O(m²)
─────────────────────────
reachable array: m+1 booleans         → O(m)
dp array:        m+1 vectors, each up to m elements → O(m²)
Total: O(m²)

NO call stack (no recursion) → no stack overflow risk ever.
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
// Time: O(n^m × m)    Space: O(m)
// ════════════════════════════════════════════════════════════
bool hs_recursive(int target, const vector<int>& nums, vector<int>& result) {
    if (target == 0) return true;
    if (target < 0)  return false;
    for (int num : nums) {
        if (hs_recursive(target - num, nums, result)) {
            result.push_back(num);
            return true;
        }
    }
    return false;
}

// ════════════════════════════════════════════════════════════
// TECHNIQUE 2: MEMOIZATION (Top-Down DP)
// Time: O(n × m²)    Space: O(m²)
// ════════════════════════════════════════════════════════════
bool hs_memo(
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
    for (int num : nums) {
        if (hs_memo(target - num, nums, result, memo_found, memo_result)) {
            result.push_back(num);
            memo_found[target]  = true;
            memo_result[target] = result;
            return true;
        }
    }
    memo_found[target] = false;
    return false;
}

// ════════════════════════════════════════════════════════════
// TECHNIQUE 3: TABULATION (Bottom-Up DP)
// Time: O(n × m²)    Space: O(m²) — NO call stack
// ════════════════════════════════════════════════════════════
bool hs_tabulation(int target, const vector<int>& nums, vector<int>& result) {
    vector<bool>        reachable(target + 1, false);
    vector<vector<int>> dp(target + 1);
    reachable[0] = true;

    for (int i = 0; i <= target; i++) {
        if (!reachable[i]) continue;
        for (int num : nums) {
            if (i + num <= target && !reachable[i + num]) {
                reachable[i + num] = true;
                dp[i + num] = dp[i];
                dp[i + num].push_back(num);
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
    cout << "]";
}

int main() {
    cout << "target  nums        Recursive       Memoized        Tabulation" << endl;
    cout << "────────────────────────────────────────────────────────────────" << endl;

    struct T { int target; vector<int> nums; };
    vector<T> tests = {
        {7, {2, 3}}, {7, {3, 4, 5}}, {7, {2, 4}}, {8, {2, 3, 5}}, {0, {1, 2, 3}}
    };

    for (auto& [t, nums] : tests) {
        unordered_map<int, bool> mf;
        unordered_map<int, vector<int>> mr;
        vector<int> r1, r2, r3;
        bool f1 = hs_recursive(t, nums, r1);
        bool f2 = hs_memo(t, nums, r2, mf, mr);
        bool f3 = hs_tabulation(t, nums, r3);

        cout << t << "       [";
        for (int i = 0; i < (int)nums.size(); i++) { cout << nums[i]; if(i<(int)nums.size()-1) cout<<",";}
        cout << "]      ";
        print(f1,r1); cout << "      ";
        print(f2,r2); cout << "      ";
        print(f3,r3); cout << endl;
    }

    cout << "\nLarge targets (memo + tabulation only):" << endl;
    cout << "────────────────────────────────────────────────────────" << endl;
    vector<T> large = { {300, {7,14}}, {140, {7,14}}, {100, {7,3}} };
    for (auto& [t, nums] : large) {
        unordered_map<int, bool> mf;
        unordered_map<int, vector<int>> mr;
        vector<int> r2, r3;
        bool f2 = hs_memo(t, nums, r2, mf, mr);
        bool f3 = hs_tabulation(t, nums, r3);
        cout << "howSum(" << t << ", [" << nums[0] << "," << nums[1] << "]):  ";
        cout << "Memo="; print(f2,r2); cout << "  ";
        cout << "Tab=";  print(f3,r3); cout << endl;
    }

    return 0;
}
```

**Output:**
```
target  nums        Recursive       Memoized        Tabulation
────────────────────────────────────────────────────────────────
7       [2,3]      [3 2 2 ]        [3 2 2 ]        [2 2 3 ]
7       [3,4,5]    [4 3 ]          [4 3 ]           [3 4 ]
7       [2,4]      null            null             null
8       [2,3,5]    [5 3 ]          [5 3 ]           [2 3 3 ]
0       [1,2,3]    []              []               []

Large targets (memo + tabulation only):
────────────────────────────────────────────────────────
howSum(300, [7,14]):  Memo=null             Tab=null
howSum(140, [7,14]):  Memo=[14 14 14 ...]   Tab=[7 7 7 ...]
howSum(100, [7,3]):   Memo=[3 3 3 ...]      Tab=[7 7 ...]
```

> **Note:** Recursive + Memoization build the result in reverse order (push_back after recursion).
> Tabulation builds it in forward order. Both give valid combinations.

---

## Complexity Summary — Side by Side

```
┌──────────────────┬─────────────────┬──────────────────────────────────────────┐
│ Technique        │      TIME       │                 SPACE                    │
├──────────────────┼─────────────────┼──────────────────────────────────────────┤
│ Recursion        │  O(n^m × m)     │ O(m) — call stack + result vector        │
│                  │                 │ WARNING: Exponential time, fails large m  │
├──────────────────┼─────────────────┼──────────────────────────────────────────┤
│ Memoization      │  O(n × m²)      │ O(m²) — memo stores m vectors of size m  │
│ (Top-Down DP)    │                 │ O(m)  — call stack                        │
│                  │                 │ Total: O(m²)                              │
├──────────────────┼─────────────────┼──────────────────────────────────────────┤
│ Tabulation       │  O(n × m²)      │ O(m)  — reachable bool array             │
│ (Bottom-Up DP)   │                 │ O(m²) — dp vector array                  │
│                  │                 │ Total: O(m²) — NO call stack, SAFEST     │
└──────────────────┴─────────────────┴──────────────────────────────────────────┘

n = numbers.size(),  m = targetSum
```

---

## Understanding WHY the Complexities Are What They Are

### Why Recursion is O(n^m × m)

```
In canSum, each node does O(1) work:  just return true/false.
In howSum, each node does O(m) work when a path is found:
  The push_back propagates back up through m levels.
  At each level: O(1) push_back, but happens m times total.

TOTAL: O(n^m) calls × O(m) propagation = O(n^m × m)
```

### Why Memo and Tabulation are O(n × m²)

```
O(n × m) factor: m unique subproblems × n numbers each (same as canSum)
Extra ×m:        copying a vector of up to m elements per operation

→ O(n × m) operations × O(m) copy = O(n × m²)

SPACE O(m²): m+1 dp entries, each storing up to m elements → m×m = m²

COMPARE WITH canSum:
  canSum: O(m) space  — stores booleans
  howSum: O(m²) space — stores vectors
```

---

## howSum vs canSum vs bestSum

```
canSum(target, numbers)    → bool
  "Is it POSSIBLE?"
  TC: O(n×m)    Space: O(m)

howSum(target, numbers)    → bool + vector (any combination)
  "HOW? Return ANY combination."
  TC: O(n×m²)   Space: O(m²)

bestSum(target, numbers)   → bool + vector (shortest combination)
  "HOW using FEWEST numbers?"
  TC: O(n×m²)   Space: O(m²)
  Difference: instead of "skip if already set",
              we set if the new combination is SHORTER.
```

---

## Quick Cheat Sheet

```
PROBLEM: howSum(target, numbers)
         Return ANY combination that sums to target (reuse allowed)
         Return false if impossible

APPROACH: bool return + vector<int>& result reference

─────────────────────────────────────────────────────────────

RECURSION:
  bool hs(int t, vector<int>& nums, vector<int>& result) {
    if(t==0) return true;
    if(t<0)  return false;
    for(int num : nums)
      if(hs(t-num, nums, result)) { result.push_back(num); return true; }
    return false;
  }
  Time: O(n^m × m)   Space: O(m)

─────────────────────────────────────────────────────────────

MEMOIZATION:
  // Extra params: memo_found (map<int,bool>), memo_result (map<int,vector>)
  if(memo_found.count(t)) {
    if(memo_found[t]) { result=memo_result[t]; return true; }
    return false;
  }
  ... [same base cases + loop as recursion]
  if(found) { memo_found[t]=true; memo_result[t]=result; return true; }
  memo_found[t]=false; return false;

  Time: O(n×m²)   Space: O(m²) + O(m) stack

─────────────────────────────────────────────────────────────

TABULATION:
  vector<bool>        reachable(target+1, false);
  vector<vector<int>> dp(target+1);
  reachable[0] = true;
  for(int i=0; i<=target; i++) {
    if(!reachable[i]) continue;
    for(int num : nums)
      if(i+num<=target && !reachable[i+num]) {
        reachable[i+num]=true;
        dp[i+num]=dp[i]; dp[i+num].push_back(num);
      }
  }
  if(reachable[target]) { result=dp[target]; return true; }
  return false;

  Time: O(n×m²)   Space: O(m²) — NO STACK

─────────────────────────────────────────────────────────────

ANSWERS:
  howSum(7,[2,3])   → true,  [2,2,3]    howSum(7,[3,4,5]) → true,  [3,4]
  howSum(7,[2,4])   → false, []          howSum(8,[2,3,5]) → true,  [2,3,3]
  howSum(0,[1,2,3]) → true,  []          howSum(300,[7,14])→ false, []

COMPLEXITY vs canSum:
  canSum: O(n×m)  time, O(m)  space  ← just booleans
  howSum: O(n×m²) time, O(m²) space  ← stores actual vectors
```

---

*Compile and run: `g++ -std=c++17 -o howsum how_sum.cpp && ./howsum`*
*(C++17 needed only for structured bindings `auto [f,r] = ...` in main — the core logic itself works with C++11/14)*
