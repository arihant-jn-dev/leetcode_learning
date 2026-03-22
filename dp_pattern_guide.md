# Dynamic Programming — Pattern Recognition & Templates
## How to Identify, Classify, and Solve Any DP Problem

> **The #1 skill in DP is not solving the problem — it's RECOGNIZING the pattern.**
> Once you know which family a problem belongs to, the template is 80% of the solution.
> This document gives you the pattern recognition radar and the templates to match.

---

## PART 1 — What Makes a Problem "DP"?

### Two Mandatory Conditions

A problem can use DP **only if it has BOTH** of these:

```
CONDITION 1: OVERLAPPING SUBPROBLEMS
  The same smaller subproblem is solved MULTIPLE TIMES.

  Example — Fibonacci(5):
    fib(5) = fib(4) + fib(3)
    fib(4) = fib(3) + fib(2)   ← fib(3) computed again!
    fib(3) = fib(2) + fib(1)   ← fib(2) computed again!

  Without DP: exponential work (recompute same things)
  With DP:    store results, never recompute → polynomial work


CONDITION 2: OPTIMAL SUBSTRUCTURE
  The OPTIMAL solution to the full problem
  can be built from OPTIMAL solutions to its subproblems.

  Example — Shortest path from A to C through B:
    shortest(A→C) = shortest(A→B) + shortest(B→C)
    The best overall path uses the best sub-paths.

  Counter-example (no optimal substructure):
    Longest path in a graph with cycles — doesn't work.
```

### Three Ways to Write DP

```
APPROACH 1: RECURSION (naive)
  Simple to write. No reuse of results.
  Time: exponential. Space: call stack depth.
  USE: Understanding the problem only.

APPROACH 2: MEMOIZATION (top-down DP)
  Recursion + a cache (map or array).
  Only compute each subproblem ONCE. Store result. Reuse on repeat calls.
  Time: O(states). Space: O(states) for cache + call stack.
  USE: Easy to derive from recursion. Natural thought process.

APPROACH 3: TABULATION (bottom-up DP)
  Fill a table iteratively, starting from smallest subproblems.
  No recursion. No stack overflow risk.
  Time: O(states). Space: O(states), often reducible.
  USE: Production code. Space optimization possible.

WHICH TO USE IN INTERVIEW?
  Start explaining with recursion (show you understand the structure).
  Upgrade to memoization first (easiest to code correctly).
  Offer tabulation as optimization if asked.
```

### The DP Thought Process (Always Follow This)

```
STEP 1: Define the STATE
  What does dp[i] mean? OR dp[i][j] mean?
  This is the hardest and most important step.
  A good state definition makes everything else fall into place.

STEP 2: Write the RECURRENCE (transition)
  How does dp[i] relate to dp[i-1], dp[i-2], etc.?
  Express the current state in terms of smaller states.

STEP 3: Identify BASE CASES
  What are the smallest subproblems with trivial answers?
  dp[0] = ?, dp[1] = ?

STEP 4: Identify the ANSWER
  Where in the table is the final answer?
  dp[n]? dp[n][W]? max(dp[i])?

STEP 5: Check for SPACE OPTIMIZATION
  Can the 2D table be compressed to 1D?
  Can two variables replace an array?
```

---

## PART 2 — The 7 DP Pattern Families

```
FAMILY 1: FIBONACCI / LINEAR DP
  Recognize: Current answer depends on 1-2 previous answers.
  State: dp[i] = answer for first i elements
  Problems: Fibonacci, Climbing Stairs, House Robber, Jump Game

FAMILY 2: 0/1 KNAPSACK
  Recognize: Items, each used AT MOST ONCE, maximize/count with a capacity W.
  State: dp[i][w] = best answer using first i items with capacity w
  Problems: 0/1 Knapsack, Subset Sum, Partition Equal Subset, Target Sum

FAMILY 3: UNBOUNDED KNAPSACK
  Recognize: Items can be used UNLIMITED times. Fill a target/capacity.
  State: dp[w] = best answer for capacity/target w
  Problems: Unbounded Knapsack, Coin Change, Coin Change II, Rod Cutting

FAMILY 4: LCS (Longest Common Subsequence) FAMILY
  Recognize: Two strings/sequences. Find common pattern.
  State: dp[i][j] = answer for first i chars of s1 and first j chars of s2
  Problems: LCS, Edit Distance, LIS, Longest Palindromic Subsequence

FAMILY 5: GRID / 2D DP
  Recognize: Moving through a grid, count paths or find min/max cost.
  State: dp[i][j] = answer to reach cell (i,j)
  Problems: Grid Traveler, Min Path Sum, Unique Paths, Triangle

FAMILY 6: INTERVAL DP
  Recognize: Answer for a range [i..j] depends on splitting at some k.
  State: dp[i][j] = answer for range i to j
  Problems: Matrix Chain Multiplication, Burst Balloons, Palindrome Partition

FAMILY 7: STATE MACHINE DP
  Recognize: You're in one of several states, transitions have costs.
  State: dp[i][state] = best answer at index i in given state
  Problems: Stock Buy/Sell (all variants), Regular Expression
```

---

## PART 3 — FAMILY 1: Fibonacci / Linear DP

### Recognition Pattern

```
KEYWORDS: "Number of ways", "minimum steps", "maximum score"
          where each step depends only on a FEW previous steps.

SIGNALS:
  ✓ You're making choices at each position (step 1, step 2, ...)
  ✓ Each choice depends on 1-3 previous positions
  ✓ No "items" to pick from a set — just a sequence

TEMPLATE FEEL:
  dp[i] = f(dp[i-1], dp[i-2], ...)
```

### Template

```cpp
// LINEAR DP TEMPLATE
// dp[i] = answer for position i

int linearDP(int n) {
    vector<int> dp(n + 1, 0);

    // BASE CASES
    dp[0] = base0;
    dp[1] = base1;

    // FILL TABLE
    for (int i = 2; i <= n; i++) {
        dp[i] = /* recurrence using dp[i-1], dp[i-2] */;
    }

    return dp[n];
}

// SPACE OPTIMIZED (when only last 2 states needed)
int linearDPOpt(int n) {
    int prev2 = base0, prev1 = base1;
    for (int i = 2; i <= n; i++) {
        int curr = /* f(prev1, prev2) */;
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
```

### Problem 1A: Climbing Stairs

**Problem:** You can climb 1 or 2 steps. How many ways to reach step N?

```
STATE:  dp[i] = number of ways to reach step i
RECURRENCE: dp[i] = dp[i-1] + dp[i-2]
  (came from step i-1 in 1 step, OR from step i-2 in 2 steps)
BASE: dp[0]=1 (1 way to stand at ground), dp[1]=1
```

```cpp
int climbStairs(int n) {
    if (n <= 1) return 1;
    int prev2 = 1, prev1 = 1;
    for (int i = 2; i <= n; i++) {
        int curr = prev1 + prev2;
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
// Time: O(n)  Space: O(1)
```

### Problem 1B: House Robber

**Problem:** Rob houses in a row. Adjacent houses have alarms. Maximize money robbed.

```
STATE:  dp[i] = max money robbing from houses 0..i
RECURRENCE: dp[i] = max(dp[i-1],          ← skip house i
                        dp[i-2] + nums[i]) ← rob house i
BASE: dp[0]=nums[0], dp[1]=max(nums[0],nums[1])
```

```cpp
int rob(vector<int>& nums) {
    int n = nums.size();
    if (n == 1) return nums[0];
    int prev2 = nums[0], prev1 = max(nums[0], nums[1]);
    for (int i = 2; i < n; i++) {
        int curr = max(prev1, prev2 + nums[i]);
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
// Time: O(n)  Space: O(1)
```

### Problem 1C: Jump Game II (Min Jumps)

**Problem:** Array of jump lengths. Find minimum jumps to reach last index.

```
STATE:  dp[i] = min jumps to reach index i
RECURRENCE: dp[i] = min over all j < i where j + nums[j] >= i:
                      dp[j] + 1
BASE: dp[0] = 0
```

```cpp
int jump(vector<int>& nums) {
    int n = nums.size();
    vector<int> dp(n, INT_MAX);
    dp[0] = 0;
    for (int i = 1; i < n; i++) {
        for (int j = 0; j < i; j++) {
            if (dp[j] != INT_MAX && j + nums[j] >= i) {
                dp[i] = min(dp[i], dp[j] + 1);
            }
        }
    }
    return dp[n - 1];
}
// Time: O(n²)  Space: O(n)
// Note: Greedy gives O(n) but DP approach shown for pattern
```

---

## PART 4 — FAMILY 2: 0/1 Knapsack

### Recognition Pattern

```
KEYWORDS: "Given N items, each with weight and value"
          "Cannot use same item twice"
          "Maximize value / count subsets / check if sum possible"
          "Capacity W"

SIGNALS:
  ✓ Each item either INCLUDED or EXCLUDED (binary choice per item)
  ✓ There's a "budget" or "capacity" constraint
  ✓ Items are distinct — you pick each at most ONCE
  ✓ Question asks max/min/count/possible

THE DISGUISES (all are 0/1 Knapsack in disguise):
  "Can you make sum S from this array?"           → subset sum
  "Partition array into two equal halves?"        → equal subset sum
  "Count subsets with sum S?"                     → count variant
  "Assign + or - to nums, count ways to get T?"  → target sum
```

### Template

```cpp
// 0/1 KNAPSACK TEMPLATE
// n items, capacity W
// items have weight[] and value[]
// dp[i][w] = max value using first i items with capacity w

int knapsack01(int W, vector<int>& weight, vector<int>& value, int n) {
    // TABLE: (n+1) x (W+1)
    vector<vector<int>> dp(n + 1, vector<int>(W + 1, 0));

    for (int i = 1; i <= n; i++) {
        for (int w = 0; w <= W; w++) {
            // OPTION 1: DON'T take item i
            dp[i][w] = dp[i-1][w];

            // OPTION 2: TAKE item i (only if it fits)
            if (weight[i-1] <= w) {
                dp[i][w] = max(dp[i][w],
                               dp[i-1][w - weight[i-1]] + value[i-1]);
            }
        }
    }
    return dp[n][W];
}

// ─── SPACE OPTIMIZED to 1D ────────────────────────────────────
// Key: iterate w from RIGHT to LEFT to avoid using item i twice
int knapsack01Opt(int W, vector<int>& weight, vector<int>& value, int n) {
    vector<int> dp(W + 1, 0);

    for (int i = 0; i < n; i++) {
        for (int w = W; w >= weight[i]; w--) {  // ← RIGHT TO LEFT (crucial!)
            dp[w] = max(dp[w], dp[w - weight[i]] + value[i]);
        }
    }
    return dp[W];
}
```

**Why right to left in 1D?**
```
When we compute dp[w] for item i, we need dp[w - weight[i]] from BEFORE item i.
If we go left to right, dp[w - weight[i]] is ALREADY UPDATED for item i.
→ We'd be using item i twice! (making it unbounded knapsack by mistake)
Going RIGHT to LEFT ensures dp[w - weight[i]] is still the i-1 version.
```

### Problem 2A: Subset Sum

**Problem:** Does any subset of `nums` sum to `target`?

```
STATE:  dp[i][s] = true if subset of first i elements can sum to s
RECURRENCE:
  dp[i][s] = dp[i-1][s]                  (don't take element i)
           OR dp[i-1][s - nums[i-1]]     (take element i, if s >= nums[i-1])
BASE: dp[i][0] = true (empty subset sums to 0)
ANSWER: dp[n][target]
```

```cpp
bool canPartition(vector<int>& nums, int target) {
    int n = nums.size();
    vector<bool> dp(target + 1, false);
    dp[0] = true;                         // empty subset sums to 0

    for (int i = 0; i < n; i++) {
        for (int s = target; s >= nums[i]; s--) {  // RIGHT TO LEFT
            dp[s] = dp[s] || dp[s - nums[i]];
        }
    }
    return dp[target];
}
// Time: O(n * target)  Space: O(target)
```

### Problem 2B: Partition Equal Subset Sum

**Problem:** Can you split array into two subsets with equal sum?

```
KEY INSIGHT:
  Total sum = S. Equal partition means each half = S/2.
  If S is odd → impossible.
  Otherwise → Subset Sum problem with target = S/2.

REDUCES TO: Problem 2A with target = totalSum / 2
```

```cpp
bool canPartitionEqualSubset(vector<int>& nums) {
    int total = 0;
    for (int x : nums) total += x;
    if (total % 2 != 0) return false;
    int target = total / 2;

    vector<bool> dp(target + 1, false);
    dp[0] = true;
    for (int num : nums) {
        for (int s = target; s >= num; s--)
            dp[s] = dp[s] || dp[s - num];
    }
    return dp[target];
}
// Time: O(n * sum)  Space: O(sum)
```

### Problem 2C: Count of Subsets with Given Sum

**Problem:** Count subsets of `nums` that sum to `target`.

```
STATE:  dp[s] = number of subsets summing to s
RECURRENCE: dp[s] += dp[s - nums[i]]  (for each item, right to left)
BASE: dp[0] = 1 (one way to make sum 0: empty subset)
ANSWER: dp[target]

DIFFERENCE FROM SUBSET SUM:
  Subset Sum: dp[s] = dp[s] || dp[s - num]   (boolean OR)
  Count:      dp[s] = dp[s] + dp[s - num]    (addition/accumulation)
```

```cpp
int countSubsets(vector<int>& nums, int target) {
    vector<int> dp(target + 1, 0);
    dp[0] = 1;
    for (int num : nums) {
        for (int s = target; s >= num; s--)
            dp[s] += dp[s - num];
    }
    return dp[target];
}
// Time: O(n * target)  Space: O(target)
```

### Problem 2D: Target Sum (Assign + and -)

**Problem:** Assign + or − to each element. Count ways to make sum = target.

```
KEY INSIGHT (the transformation):
  Let P = sum of elements assigned +
  Let N = sum of elements assigned -
  P + N = totalSum
  P - N = target
  → 2P = totalSum + target
  → P = (totalSum + target) / 2

  Count subsets with sum = (totalSum + target) / 2
  REDUCES TO: Problem 2C (count subsets with given sum)

EDGE CASES:
  If (totalSum + target) is odd → 0 ways
  If target > totalSum → 0 ways
```

```cpp
int findTargetSumWays(vector<int>& nums, int target) {
    int total = 0;
    for (int x : nums) total += x;
    if ((total + target) % 2 != 0) return 0;
    if (abs(target) > total) return 0;
    int newTarget = (total + target) / 2;

    vector<int> dp(newTarget + 1, 0);
    dp[0] = 1;
    for (int num : nums) {
        for (int s = newTarget; s >= num; s--)
            dp[s] += dp[s - num];
    }
    return dp[newTarget];
}
// Time: O(n * sum)  Space: O(sum)
```

### 0/1 Knapsack Family — All Variants at a Glance

```
PROBLEM                      TARGET TYPE   DP TYPE    RECURRENCE KEY
──────────────────────────────────────────────────────────────────────────
0/1 Knapsack (max value)     maximize      max        dp[w] = max(dp[w], dp[w-wt]+val)
Subset Sum (possible?)       boolean       bool       dp[s] = dp[s] || dp[s-num]
Count subsets with sum S     count         int/add    dp[s] += dp[s-num]
Partition equal subset       boolean       bool       subset sum with target=sum/2
Target Sum (+/-)             count         int/add    count subsets = (sum+target)/2
Min subset sum difference    minimize      int        minimize |subset1 - subset2|
──────────────────────────────────────────────────────────────────────────
All use: right-to-left inner loop, dp[0]=base, items processed one by one
```

---

## PART 5 — FAMILY 3: Unbounded Knapsack

### Recognition Pattern

```
KEYWORDS: "Unlimited supply", "use as many times as you want"
          "Coin change", "minimum coins", "rod cutting"
          "Fill exactly / at minimum cost"

SIGNALS:
  ✓ Items CAN be reused (infinite copies of each)
  ✓ There's a target to fill (amount, length, weight)
  ✓ Minimize count/cost OR count ways to fill

DIFFERENCE FROM 0/1 KNAPSACK:
  0/1 Knapsack: each item used AT MOST ONCE → right to left
  Unbounded:    each item used UNLIMITED    → LEFT TO RIGHT (can reuse)
```

### Template

```cpp
// UNBOUNDED KNAPSACK TEMPLATE
// Items can be used unlimited times
// dp[w] = best answer for capacity/target w

int unboundedKnapsack(int W, vector<int>& weight, vector<int>& value) {
    int n = weight.size();
    vector<int> dp(W + 1, 0);

    for (int i = 0; i < n; i++) {
        for (int w = weight[i]; w <= W; w++) {  // ← LEFT TO RIGHT (reuse allowed!)
            dp[w] = max(dp[w], dp[w - weight[i]] + value[i]);
        }
    }
    return dp[W];
}
```

**Left to right = reuse allowed:**
```
When we compute dp[w] for item i using dp[w - weight[i]]:
  If we go LEFT to RIGHT, dp[w - weight[i]] is ALREADY UPDATED for item i.
  → We're allowed to use item i again! That's exactly unbounded behavior.
0/1:      right → left  (no reuse)
Unbounded: left → right  (reuse ok)
```

### Problem 3A: Coin Change (Minimum Coins)

**Problem:** Given coins of various denominations, find minimum coins to make amount.

```
STATE:  dp[a] = min coins to make amount a
RECURRENCE: dp[a] = min over all coins c:
              dp[a - c] + 1    (if a >= c and dp[a-c] is reachable)
BASE: dp[0] = 0 (0 coins to make amount 0)
ANSWER: dp[amount]  (if still INT_MAX → impossible)
```

```cpp
int coinChange(vector<int>& coins, int amount) {
    vector<int> dp(amount + 1, INT_MAX);
    dp[0] = 0;

    for (int a = 1; a <= amount; a++) {
        for (int coin : coins) {
            if (coin <= a && dp[a - coin] != INT_MAX) {
                dp[a] = min(dp[a], dp[a - coin] + 1);
            }
        }
    }
    return (dp[amount] == INT_MAX) ? -1 : dp[amount];
}
// Time: O(amount * coins)  Space: O(amount)
```

**Dry Run:**
```
coins = [1, 2, 5],  amount = 5

dp[0]=0
dp[1]: coin=1: dp[0]+1=1 → dp[1]=1
dp[2]: coin=1: dp[1]+1=2, coin=2: dp[0]+1=1 → dp[2]=1
dp[3]: coin=1: dp[2]+1=2, coin=2: dp[1]+1=2 → dp[3]=2
dp[4]: coin=1: dp[3]+1=3, coin=2: dp[2]+1=2 → dp[4]=2
dp[5]: coin=1: dp[4]+1=3, coin=2: dp[3]+1=3, coin=5: dp[0]+1=1 → dp[5]=1

Answer: 1 (one coin of value 5)  ✓
```

### Problem 3B: Coin Change II (Count Ways)

**Problem:** Count the number of ways to make the amount using coins (unlimited).

```
STATE:  dp[a] = number of ways to make amount a
RECURRENCE: dp[a] += dp[a - coin]   for each coin (left to right)
BASE: dp[0] = 1 (one way to make 0: use no coins)

CRITICAL: Process COINS in outer loop, AMOUNT in inner loop.
          This avoids counting [1,2] and [2,1] as different (combinations not permutations).
```

```cpp
int change(int amount, vector<int>& coins) {
    vector<int> dp(amount + 1, 0);
    dp[0] = 1;

    for (int coin : coins) {                    // ← coin in outer loop!
        for (int a = coin; a <= amount; a++) {  // ← left to right
            dp[a] += dp[a - coin];
        }
    }
    return dp[amount];
}
// Time: O(amount * coins)  Space: O(amount)
```

**Why coin in outer loop?**
```
OUTER LOOP = COINS, INNER = AMOUNT (combinations — order doesn't matter):
  Processing coin=1 first, then coin=2:
  [1,1,2] and [1,2,1] and [2,1,1] all count as ONE way (same coins, different order)
  dp builds ways to make each amount using coins seen so far.

OUTER LOOP = AMOUNT, INNER = COINS (permutations — order matters):
  Would count [1,2] and [2,1] as DIFFERENT ways.
  Use this for "number of ordered ways" / permutations.
```

### Problem 3C: Rod Cutting (Max Revenue)

**Problem:** Rod of length n. Can cut into pieces. Each length has a price. Maximize total price.

```
STATE:  dp[l] = max revenue for rod of length l
RECURRENCE: dp[l] = max over all cuts c from 1 to l:
              price[c] + dp[l - c]
BASE: dp[0] = 0
ANSWER: dp[n]
```

```cpp
int rodCutting(vector<int>& price, int n) {
    vector<int> dp(n + 1, 0);

    for (int l = 1; l <= n; l++) {
        for (int c = 1; c <= l; c++) {
            dp[l] = max(dp[l], price[c - 1] + dp[l - c]);
        }
    }
    return dp[n];
}
// Time: O(n²)  Space: O(n)
```

### Unbounded Knapsack Family — All Variants

```
PROBLEM                     GOAL        INNER LOOP      DP INIT   RECURRENCE
────────────────────────────────────────────────────────────────────────────────
Unbounded Knapsack          maximize    left→right      dp[0]=0   dp[w]=max(dp[w], dp[w-wt]+val)
Coin Change (min coins)     minimize    amount outer    dp[0]=0   dp[a]=min(dp[a], dp[a-c]+1)
Coin Change II (count ways) count       coin outer→amt  dp[0]=1   dp[a]+=dp[a-c]
Rod Cutting (max revenue)   maximize    length outer    dp[0]=0   dp[l]=max(dp[l], price+dp[l-c])
────────────────────────────────────────────────────────────────────────────────
KEY RULE: All use left-to-right inner loop (allowing item reuse)
```

---

## PART 6 — FAMILY 4: LCS (Longest Common Subsequence) Family

### Recognition Pattern

```
KEYWORDS: "Two strings", "common subsequence/substring",
          "edit distance", "delete/insert/replace to convert",
          "longest palindromic subsequence"

SIGNALS:
  ✓ Two sequences (strings, arrays) given
  ✓ Compare elements of both at positions i and j
  ✓ Choice depends on whether s1[i] == s2[j]

THE DISGUISES (all are LCS variants):
  "Longest common subsequence"             → classic LCS
  "Longest common substring"               → LCS variant (reset on mismatch)
  "Min operations to convert s1 to s2"    → Edit Distance
  "Min insertions+deletions to make same" → n+m - 2*LCS
  "Shortest common supersequence"          → n+m - LCS
  "Longest palindromic subsequence"        → LCS(s, reverse(s))
  "Min insertions to make palindrome"      → n - LCS(s, reverse(s))
```

### Template

```cpp
// LCS TEMPLATE
// dp[i][j] = answer for s1[0..i-1] and s2[0..j-1]

int lcsTemplate(string s1, string s2) {
    int m = s1.size(), n = s2.size();
    vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (s1[i-1] == s2[j-1]) {
                dp[i][j] = dp[i-1][j-1] + 1;     // characters MATCH
            } else {
                dp[i][j] = max(dp[i-1][j],         // skip char from s1
                               dp[i][j-1]);         // skip char from s2
            }
        }
    }
    return dp[m][n];
}
// Time: O(m*n)  Space: O(m*n), reducible to O(n)
```

### Problem 4A: Longest Common Subsequence (LCS)

**Problem:** Find the length of the longest subsequence common to both strings.

```
STATE:  dp[i][j] = LCS of s1[0..i-1] and s2[0..j-1]
RECURRENCE:
  if s1[i-1] == s2[j-1]: dp[i][j] = dp[i-1][j-1] + 1
  else:                   dp[i][j] = max(dp[i-1][j], dp[i][j-1])
BASE: dp[0][j] = dp[i][0] = 0 (empty string has LCS 0)
```

```cpp
int lcs(string s1, string s2) {
    int m = s1.size(), n = s2.size();
    vector<vector<int>> dp(m+1, vector<int>(n+1, 0));
    for (int i = 1; i <= m; i++)
        for (int j = 1; j <= n; j++)
            if (s1[i-1] == s2[j-1]) dp[i][j] = dp[i-1][j-1] + 1;
            else                     dp[i][j] = max(dp[i-1][j], dp[i][j-1]);
    return dp[m][n];
}
// Time: O(m*n)  Space: O(m*n)
```

### Problem 4B: Edit Distance (Levenshtein)

**Problem:** Minimum insert, delete, replace operations to convert s1 to s2.

```
STATE:  dp[i][j] = min operations to convert s1[0..i-1] to s2[0..j-1]
RECURRENCE:
  if s1[i-1] == s2[j-1]:
    dp[i][j] = dp[i-1][j-1]               ← no operation needed
  else:
    dp[i][j] = 1 + min(
      dp[i-1][j-1],                        ← REPLACE s1[i-1] with s2[j-1]
      dp[i-1][j],                          ← DELETE s1[i-1]
      dp[i][j-1]                           ← INSERT s2[j-1] into s1
    )
BASE: dp[i][0] = i (delete all i chars of s1)
      dp[0][j] = j (insert all j chars of s2)
```

```cpp
int minDistance(string s1, string s2) {
    int m = s1.size(), n = s2.size();
    vector<vector<int>> dp(m+1, vector<int>(n+1, 0));

    for (int i = 0; i <= m; i++) dp[i][0] = i;
    for (int j = 0; j <= n; j++) dp[0][j] = j;

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (s1[i-1] == s2[j-1])
                dp[i][j] = dp[i-1][j-1];
            else
                dp[i][j] = 1 + min({dp[i-1][j-1], dp[i-1][j], dp[i][j-1]});
        }
    }
    return dp[m][n];
}
// Time: O(m*n)  Space: O(m*n)
```

### Problem 4C: Longest Palindromic Subsequence

**Problem:** Find the longest subsequence of a string that is a palindrome.

```
KEY INSIGHT: LPS(s) = LCS(s, reverse(s))
  A palindrome reads the same forward and backward.
  The longest common subsequence of s and its reverse = longest palindrome.

Example: s = "bbbab"
  reverse = "babbb"
  LCS("bbbab", "babbb") = "bbbb" → length 4  ✓
```

```cpp
int longestPalinSubseq(string s) {
    string rev = s;
    reverse(rev.begin(), rev.end());
    return lcs(s, rev);   // reuse LCS function
}
// Time: O(n²)  Space: O(n²)
```

### Problem 4D: Shortest Common Supersequence

**Problem:** Find length of shortest string that has both s1 and s2 as subsequences.

```
KEY INSIGHT:
  SCS length = m + n - LCS(s1, s2)
  We include all chars of s1 and s2, but share the LCS part (not doubled).

  Once you know LCS, this is O(1) extra work.
```

```cpp
int shortestCommonSupersequence(string s1, string s2) {
    return s1.size() + s2.size() - lcs(s1, s2);
}
```

### LCS Family — All Variants

```
PROBLEM                          FORMULA / APPROACH
──────────────────────────────────────────────────────────────────────────
Longest Common Subsequence       Standard LCS DP
Longest Common Substring         LCS variant: reset to 0 on mismatch, track max
Edit Distance                    LCS variant: 3-way recurrence (replace/del/ins)
Shortest Common Supersequence    m + n - LCS
Min insertions+deletions (same)  (m - LCS) + (n - LCS)
Longest Palindromic Subsequence  LCS(s, reverse(s))
Min insertions for palindrome    n - LCS(s, reverse(s))
Sequence Pattern Matching (s1⊆s2?) LCS(s1,s2) == len(s1)
──────────────────────────────────────────────────────────────────────────
```

---

## PART 7 — FAMILY 4B: Longest Increasing Subsequence (LIS)

### Recognition Pattern

```
KEYWORDS: "Longest increasing subsequence", "chain of pairs",
          "Russian doll envelopes", "number of buildings visible"

SIGNALS:
  ✓ Single array (not two strings like LCS)
  ✓ Must maintain ORDER (not rearrange)
  ✓ Looking for increasing/decreasing/non-decreasing pattern

NOTE: LIS is often grouped with LCS but has its own distinct approach.
```

### Template

```cpp
// LIS TEMPLATE — O(n²)
// dp[i] = length of LIS ending at index i

int lis(vector<int>& nums) {
    int n = nums.size();
    vector<int> dp(n, 1);          // every element is LIS of length 1 alone

    int maxLen = 1;
    for (int i = 1; i < n; i++) {
        for (int j = 0; j < i; j++) {
            if (nums[j] < nums[i]) {       // nums[i] can extend LIS ending at j
                dp[i] = max(dp[i], dp[j] + 1);
            }
        }
        maxLen = max(maxLen, dp[i]);
    }
    return maxLen;
}
// Time: O(n²)  Space: O(n)

// LIS — O(n log n) using patience sort / binary search
int lisOptimal(vector<int>& nums) {
    vector<int> tails;                      // tails[i] = smallest tail of IS of length i+1

    for (int num : nums) {
        auto it = lower_bound(tails.begin(), tails.end(), num);
        if (it == tails.end())
            tails.push_back(num);           // extend longest
        else
            *it = num;                      // replace to keep tail small
    }
    return tails.size();
}
// Time: O(n log n)  Space: O(n)
```

---

## PART 8 — FAMILY 5: Grid / 2D DP

### Recognition Pattern

```
KEYWORDS: "Grid", "matrix", "paths from top-left to bottom-right"
          "Minimum cost path", "number of unique paths"

SIGNALS:
  ✓ 2D grid given
  ✓ Move in limited directions (right/down, or all 4)
  ✓ Count paths OR find min/max cost to reach destination
  ✓ dp[i][j] naturally maps to cell (i,j)
```

### Template

```cpp
// GRID DP TEMPLATE
// dp[i][j] = answer for cell (i,j)
// Movement: right and down only

int gridDP(vector<vector<int>>& grid) {
    int m = grid.size(), n = grid[0].size();
    vector<vector<int>> dp(m, vector<int>(n, 0));

    // BASE CASE: first row and first column
    dp[0][0] = grid[0][0];
    for (int j = 1; j < n; j++) dp[0][j] = dp[0][j-1] + grid[0][j];  // first row
    for (int i = 1; i < m; i++) dp[i][0] = dp[i-1][0] + grid[i][0];  // first col

    // FILL: came from top OR from left
    for (int i = 1; i < m; i++) {
        for (int j = 1; j < n; j++) {
            dp[i][j] = grid[i][j] + min(dp[i-1][j],   // came from above
                                         dp[i][j-1]);   // came from left
        }
    }
    return dp[m-1][n-1];
}
```

### Problem 5A: Unique Paths

**Problem:** Count paths from top-left to bottom-right. Move only right or down.

```
STATE:  dp[i][j] = number of paths to reach cell (i,j)
RECURRENCE: dp[i][j] = dp[i-1][j] + dp[i][j-1]
BASE: dp[0][j] = 1 (only one way across first row — keep going right)
     dp[i][0] = 1 (only one way down first col — keep going down)
```

```cpp
int uniquePaths(int m, int n) {
    vector<vector<int>> dp(m, vector<int>(n, 1));
    for (int i = 1; i < m; i++)
        for (int j = 1; j < n; j++)
            dp[i][j] = dp[i-1][j] + dp[i][j-1];
    return dp[m-1][n-1];
}
// Time: O(m*n)  Space: O(m*n), reducible to O(n)
```

### Problem 5B: Minimum Path Sum

**Problem:** Grid with non-negative integers. Find minimum sum path from top-left to bottom-right.

```cpp
int minPathSum(vector<vector<int>>& grid) {
    int m = grid.size(), n = grid[0].size();
    vector<vector<int>> dp(m, vector<int>(n));
    dp[0][0] = grid[0][0];
    for (int j = 1; j < n; j++) dp[0][j] = dp[0][j-1] + grid[0][j];
    for (int i = 1; i < m; i++) dp[i][0] = dp[i-1][0] + grid[i][0];
    for (int i = 1; i < m; i++)
        for (int j = 1; j < n; j++)
            dp[i][j] = grid[i][j] + min(dp[i-1][j], dp[i][j-1]);
    return dp[m-1][n-1];
}
// Time: O(m*n)  Space: O(m*n)
```

### Problem 5C: Triangle (Min Path from Top to Bottom)

**Problem:** Given triangle of numbers, find minimum path sum from top to bottom. At each row i, you can move to adjacent element in row i+1.

```
STATE:  dp[i][j] = min path sum to reach triangle[i][j]
RECURRENCE: dp[i][j] = triangle[i][j] + min(dp[i-1][j-1], dp[i-1][j])

SPACE TRICK: Process bottom-up in 1D array
```

```cpp
int minimumTotal(vector<vector<int>>& triangle) {
    int n = triangle.size();
    vector<int> dp = triangle[n-1];          // start from bottom row

    for (int i = n-2; i >= 0; i--)
        for (int j = 0; j <= i; j++)
            dp[j] = triangle[i][j] + min(dp[j], dp[j+1]);

    return dp[0];
}
// Time: O(n²)  Space: O(n)
```

---

## PART 9 — FAMILY 6: Interval DP

### Recognition Pattern

```
KEYWORDS: "Optimal way to split/merge an interval"
          "Matrix chain", "burst balloons", "palindrome partitioning"

SIGNALS:
  ✓ Problem is about a RANGE [i..j]
  ✓ Answer for [i..j] depends on SPLITTING at some point k
  ✓ You try all possible split points and pick the best
  ✓ Subproblems are smaller intervals [i..k] and [k+1..j]

LOOP ORDER (critical!):
  Outer loop: LENGTH of interval (from small to large)
  Inner loops: left boundary i, then split point k
  j = i + len - 1
```

### Template

```cpp
// INTERVAL DP TEMPLATE
// dp[i][j] = optimal answer for range [i..j]

int intervalDP(vector<int>& arr) {
    int n = arr.size();
    vector<vector<int>> dp(n, vector<int>(n, 0));

    // Base: single elements (length 1) — already 0 or trivial

    // Fill by increasing LENGTH of interval
    for (int len = 2; len <= n; len++) {          // interval length
        for (int i = 0; i <= n - len; i++) {      // left boundary
            int j = i + len - 1;                  // right boundary
            dp[i][j] = INT_MAX;                   // or INT_MIN / 0

            // Try all split points k
            for (int k = i; k < j; k++) {
                int cost = dp[i][k] + dp[k+1][j] + /* merge cost */;
                dp[i][j] = min(dp[i][j], cost);
            }
        }
    }
    return dp[0][n-1];
}
```

### Problem 6A: Matrix Chain Multiplication

**Problem:** Given n matrices, find the minimum number of multiplications to compute their product.

```
STATE:  dp[i][j] = min multiplications to compute matrices i to j
RECURRENCE: dp[i][j] = min over k from i to j-1:
              dp[i][k] + dp[k+1][j] + dims[i] * dims[k+1] * dims[j+1]
BASE: dp[i][i] = 0 (single matrix, no multiplication)
ANSWER: dp[0][n-1]
```

```cpp
int matrixChain(vector<int>& dims) {
    int n = dims.size() - 1;              // number of matrices
    vector<vector<int>> dp(n, vector<int>(n, 0));

    for (int len = 2; len <= n; len++) {
        for (int i = 0; i <= n - len; i++) {
            int j = i + len - 1;
            dp[i][j] = INT_MAX;
            for (int k = i; k < j; k++) {
                int cost = dp[i][k] + dp[k+1][j]
                         + dims[i] * dims[k+1] * dims[j+1];
                dp[i][j] = min(dp[i][j], cost);
            }
        }
    }
    return dp[0][n-1];
}
// Time: O(n³)  Space: O(n²)
```

### Problem 6B: Burst Balloons

**Problem:** Array of balloons with numbers. Bursting balloon i gives `nums[left]*nums[i]*nums[right]` coins. Maximize total coins.

```
KEY INSIGHT:
  Instead of thinking "which to burst first",
  think "which is the LAST balloon to burst in range [i..j]".
  Last to burst means both its neighbors are still the boundaries.

STATE:  dp[i][j] = max coins from bursting all balloons between i and j
                   (not including i and j — they're boundaries)
RECURRENCE: dp[i][j] = max over k (last to burst):
              dp[i][k] + dp[k][j] + nums[i]*nums[k]*nums[j]

Add sentinel 1 at both ends for boundary handling.
```

```cpp
int maxCoins(vector<int>& nums) {
    nums.insert(nums.begin(), 1);
    nums.push_back(1);
    int n = nums.size();
    vector<vector<int>> dp(n, vector<int>(n, 0));

    for (int len = 2; len < n; len++) {
        for (int i = 0; i < n - len; i++) {
            int j = i + len;
            for (int k = i + 1; k < j; k++) {
                dp[i][j] = max(dp[i][j],
                    dp[i][k] + dp[k][j] + nums[i]*nums[k]*nums[j]);
            }
        }
    }
    return dp[0][n-1];
}
// Time: O(n³)  Space: O(n²)
```

---

## PART 10 — FAMILY 7: State Machine DP

### Recognition Pattern

```
KEYWORDS: "Stock trading", "buy and sell", "cooldown", "at most K transactions"
          "Finite set of states you transition between"

SIGNALS:
  ✓ At each step, you're in one of a small number of states
  ✓ Each transition (action) changes your state and has a cost/profit
  ✓ dp[i][state] captures position + state

STATES FOR STOCK PROBLEMS:
  State 0: Not holding stock (can buy)
  State 1: Holding stock (can sell)
  (State 2: Cooldown — for cooldown variant)
```

### Problem 7A: Best Time to Buy/Sell Stock (Unlimited Transactions)

```
STATES:
  hold[i]  = max profit on day i if holding stock
  free[i]  = max profit on day i if NOT holding stock

TRANSITIONS:
  hold[i] = max(hold[i-1],        ← keep holding (do nothing)
                free[i-1] - price[i])  ← buy today

  free[i] = max(free[i-1],        ← stay free (do nothing)
                hold[i-1] + price[i])  ← sell today

BASE:
  hold[0] = -price[0]   (bought on day 0)
  free[0] = 0           (didn't buy on day 0)
```

```cpp
int maxProfit(vector<int>& prices) {
    int hold = -prices[0], free = 0;

    for (int i = 1; i < prices.size(); i++) {
        int newHold = max(hold, free - prices[i]);
        int newFree = max(free, hold + prices[i]);
        hold = newHold;
        free = newFree;
    }
    return free;
}
// Time: O(n)  Space: O(1)
```

### Problem 7B: Best Time to Buy/Sell with Cooldown

```
STATES:
  hold[i]  = max profit holding stock on day i
  sold[i]  = max profit just sold on day i (must cooldown tomorrow)
  rest[i]  = max profit in cooldown / rest state on day i

TRANSITIONS:
  hold[i] = max(hold[i-1], rest[i-1] - price[i])   ← buy only from rest state
  sold[i] = hold[i-1] + price[i]                    ← sell today
  rest[i] = max(rest[i-1], sold[i-1])               ← either continue rest or come from sold
```

```cpp
int maxProfitCooldown(vector<int>& prices) {
    int hold = -prices[0], sold = 0, rest = 0;

    for (int i = 1; i < prices.size(); i++) {
        int newHold = max(hold, rest - prices[i]);
        int newSold = hold + prices[i];
        int newRest = max(rest, sold);
        hold = newHold; sold = newSold; rest = newRest;
    }
    return max(sold, rest);
}
// Time: O(n)  Space: O(1)
```

### Problem 7C: At Most K Transactions

```
STATE:  dp[k][0] = max profit with at most k transactions, not holding
        dp[k][1] = max profit with at most k transactions, holding

TRANSITIONS:
  dp[k][0] = max(dp[k][0], dp[k][1] + price)     ← sell
  dp[k][1] = max(dp[k][1], dp[k-1][0] - price)   ← buy (uses one transaction)
```

```cpp
int maxProfitK(int K, vector<int>& prices) {
    int n = prices.size();
    if (K >= n / 2) {                          // unlimited transactions case
        int profit = 0;
        for (int i = 1; i < n; i++)
            profit += max(0, prices[i] - prices[i-1]);
        return profit;
    }

    vector<vector<int>> dp(K+1, vector<int>(2, 0));
    for (int k = 1; k <= K; k++) dp[k][1] = INT_MIN;

    for (int price : prices) {
        for (int k = K; k >= 1; k--) {
            dp[k][0] = max(dp[k][0], dp[k][1] + price);
            dp[k][1] = max(dp[k][1], dp[k-1][0] - price);
        }
    }
    return dp[K][0];
}
// Time: O(n*K)  Space: O(K)
```

---

## PART 11 — THE MASTER RECOGNITION TABLE

```
PATTERN SIGNAL IN PROBLEM                   → FAMILY
────────────────────────────────────────────────────────────────────────────────
"Number of ways to climb / reach"           → Fibonacci / Linear DP
"Rob houses / can't take adjacent"          → Fibonacci / Linear DP
"Given items, capacity W, maximize value"   → 0/1 Knapsack
"Subset with sum S possible?"               → 0/1 Knapsack (Subset Sum)
"Equal partition / two balanced halves"     → 0/1 Knapsack (target=sum/2)
"Assign + or -, count ways to reach T"     → 0/1 Knapsack (Target Sum trick)
"Count subsets with sum S"                  → 0/1 Knapsack (count variant)
"Items can be reused, fill target"          → Unbounded Knapsack
"Minimum coins to make amount"              → Unbounded Knapsack (Coin Change)
"Number of ways to make amount with coins"  → Unbounded Knapsack (Coin Change II)
"Two strings, find common pattern"          → LCS Family
"Convert string A to B, min operations"    → Edit Distance (LCS family)
"Longest palindromic subsequence"           → LCS(s, reverse(s))
"Longest increasing subsequence"            → LIS (own template)
"Grid from top-left to bottom-right"        → Grid DP
"Min cost to traverse grid"                 → Grid DP
"Optimal way to split range [i..j]"         → Interval DP
"Matrix chain / merge intervals"            → Interval DP
"Burst balloons / stone merging"            → Interval DP
"Stock buy/sell with states"               → State Machine DP
"Buy/sell with cooldown or fee"             → State Machine DP
────────────────────────────────────────────────────────────────────────────────
```

---

## PART 12 — STATE DEFINITION PATTERNS

The hardest part of any DP. Here's how to think about it:

```
TYPE 1: dp[i] — "Answer for first i elements"
  Used when: Linear problems, one sequence
  Examples:  dp[i] = max money robbing houses 0..i
             dp[i] = min jumps to reach index i
             dp[i] = LIS ending at index i

TYPE 2: dp[i][j] — "Answer for range [i..j]" or "first i of s1, first j of s2"
  Used when: Two sequences, OR interval problems
  Examples:  dp[i][j] = LCS of s1[0..i-1] and s2[0..j-1]
             dp[i][j] = min cost to split array from i to j
             dp[i][j] = min edits to convert s1[0..i] to s2[0..j]

TYPE 3: dp[i][w] — "Answer for first i items with budget w"
  Used when: Knapsack-style (items + capacity)
  Examples:  dp[i][w] = max value from first i items with capacity w
             dp[i][s] = true if first i elements can make sum s

TYPE 4: dp[i][state] — "Answer at position i in some state"
  Used when: State machine / multiple modes
  Examples:  dp[i][0] = max profit at day i, not holding stock
             dp[i][1] = max profit at day i, holding stock
             dp[i][k] = max profit with at most k transactions at day i

TYPE 5: dp[mask] — "Answer for a subset encoded as bitmask"
  Used when: Small set of elements (≤20), all subsets needed
  Examples:  dp[mask] = min cost to visit cities in mask (TSP)
```

---

## PART 13 — COMPLEXITY IMPACT OF DP

```
PROBLEM                      BRUTE FORCE       DP              IMPROVEMENT
─────────────────────────────────────────────────────────────────────────────
Fibonacci(n)                 O(2^n)            O(n)            exponential→linear
0/1 Knapsack (n items, W)    O(2^n)            O(n*W)          exponential→polynomial
Coin Change (amount A, k)    O(k^A)            O(k*A)          exponential→polynomial
LCS (m,n chars)              O(2^min(m,n))     O(m*n)          exponential→polynomial
Edit Distance                O(3^min(m,n))     O(m*n)          exponential→polynomial
Matrix Chain (n matrices)    O(n! / 2)         O(n³)           factorial→polynomial
─────────────────────────────────────────────────────────────────────────────

SPACE OPTIMIZATION SUMMARY:
  FULL TABLE:    O(n²) or O(n*W)
  1D ARRAY:      O(n) or O(W)     ← when only prev row needed
  2 VARIABLES:   O(1)             ← when only last 2 values needed

HOW TO OPTIMIZE SPACE:
  If dp[i][j] only depends on dp[i-1][j] and dp[i][j-1]:
    → Use two 1D arrays (curr and prev row), or even one with careful update
  If dp[i] only depends on dp[i-1] and dp[i-2]:
    → Use two variables prev1 and prev2
  If dp[i][w] for knapsack only needs previous row:
    → Use 1D array with right-to-left (0/1) or left-to-right (unbounded)
```

---

## PART 14 — HOW TO APPROACH A NEW DP PROBLEM

```
STEP 1: IDENTIFY DP-ABILITY
  □ Does it have overlapping subproblems?
  □ Does it have optimal substructure?
  □ Am I asked for min/max/count/true-false over a large search space?

STEP 2: RECOGNIZE THE FAMILY
  □ One sequence, depends on prev 1-2? → Fibonacci/Linear
  □ Items + capacity, use each once? → 0/1 Knapsack
  □ Items + capacity, unlimited use? → Unbounded Knapsack
  □ Two strings, matching characters? → LCS family
  □ Single array, increasing order? → LIS
  □ Grid, top-left to bottom-right? → Grid DP
  □ Optimal split of range [i..j]? → Interval DP
  □ States + transitions (stock)? → State Machine DP

STEP 3: DEFINE THE STATE
  □ What does dp[i] represent? (be precise in words)
  □ If 2D: what does dp[i][j] represent?
  □ What are the dimensions? (n? n*W? m*n?)

STEP 4: WRITE THE RECURRENCE
  □ If current char matches: dp[i][j] = dp[i-1][j-1] + 1  (LCS style)
  □ If taking item i: dp[i][w] = dp[i-1][w-wt] + val  (Knapsack)
  □ Take max of choices: dp[i] = max(dp[i-1], dp[i-2] + nums[i])
  □ Split at k: dp[i][j] = min(dp[i][k] + dp[k+1][j] + cost)

STEP 5: SET BASE CASES
  □ dp[0][...] = ? (empty first sequence)
  □ dp[...][0] = ? (empty second sequence / zero capacity)
  □ Typically 0, 1, or a trivially known value

STEP 6: CODE IT UP
  □ Start with memoization (recursion + map) — easiest to verify
  □ Convert to tabulation if needed
  □ Apply space optimization if asked

STEP 7: VERIFY WITH SMALL EXAMPLE
  □ Draw the DP table for small input
  □ Trace through the recurrence manually
  □ Check base cases and final answer location
```

---

## QUICK REFERENCE: ALL TEMPLATES SIDE BY SIDE

```cpp
// ─── LINEAR / FIBONACCI ──────────────────────────────────────────────
// dp[i] = f(dp[i-1], dp[i-2])
int prev2 = base0, prev1 = base1;
for (int i = 2; i <= n; i++) { int c = f(prev1,prev2); prev2=prev1; prev1=c; }


// ─── 0/1 KNAPSACK ────────────────────────────────────────────────────
// dp[w]: RIGHT to LEFT inner loop (no reuse)
vector<int> dp(W+1, 0); dp[0] = initial;
for (int item : items)
    for (int w = W; w >= item.weight; w--)
        dp[w] = best(dp[w], dp[w - item.weight] + item.value);


// ─── UNBOUNDED KNAPSACK ───────────────────────────────────────────────
// dp[w]: LEFT to RIGHT inner loop (reuse ok)
vector<int> dp(W+1, 0); dp[0] = initial;
for (int item : items)
    for (int w = item.weight; w <= W; w++)
        dp[w] = best(dp[w], dp[w - item.weight] + item.value);


// ─── LCS ─────────────────────────────────────────────────────────────
// dp[i][j]: two strings
vector<vector<int>> dp(m+1, vector<int>(n+1, 0));
for (int i=1;i<=m;i++) for(int j=1;j<=n;j++)
    dp[i][j] = (s1[i-1]==s2[j-1]) ? dp[i-1][j-1]+1 : max(dp[i-1][j], dp[i][j-1]);


// ─── LIS ─────────────────────────────────────────────────────────────
// dp[i] = LIS ending at i
vector<int> dp(n, 1);
for (int i=1;i<n;i++) for (int j=0;j<i;j++)
    if (nums[j]<nums[i]) dp[i]=max(dp[i],dp[j]+1);


// ─── GRID ─────────────────────────────────────────────────────────────
// dp[i][j] = answer to reach (i,j)
dp[0][0]=grid[0][0];
for(int j=1;j<n;j++) dp[0][j]=dp[0][j-1]+grid[0][j];
for(int i=1;i<m;i++) dp[i][0]=dp[i-1][0]+grid[i][0];
for(int i=1;i<m;i++) for(int j=1;j<n;j++)
    dp[i][j]=grid[i][j]+min(dp[i-1][j],dp[i][j-1]);


// ─── INTERVAL ─────────────────────────────────────────────────────────
// dp[i][j]: fill by increasing length
for(int len=2;len<=n;len++) for(int i=0;i<=n-len;i++) {
    int j=i+len-1; dp[i][j]=WORST;
    for(int k=i;k<j;k++) dp[i][j]=best(dp[i][j], dp[i][k]+dp[k+1][j]+cost);
}


// ─── STATE MACHINE ────────────────────────────────────────────────────
// hold/free/sold: update all states simultaneously each day
int hold=-prices[0], free=0;
for(int i=1;i<n;i++) {
    int h=max(hold, free-prices[i]);
    int f=max(free, hold+prices[i]);
    hold=h; free=f;
}
```
