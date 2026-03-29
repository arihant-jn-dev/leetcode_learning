# Dynamic Programming — Interview Questions Guide
## All 8 Patterns × Real Interview Questions × Solving Strategies

> **This document maps every DP pattern you have studied to real interview questions.**
> For each problem: which pattern it is, the key insight, the complexity, and
> what the interviewer is actually testing.
> Use this as your interview prep reference — not just a problem list.

---

## PART 0 — The 8 DP Patterns You Have Mastered

```
PATTERN            PROBLEM TYPE              RETURN TYPE         REFERENCE DOC
───────────────────────────────────────────────────────────────────────────────
1. Fibonacci       1D linear recurrence      int / long long     fibonacci_techniques.md
2. Grid Traveler   2D grid paths             int / long long     grid_traveler_techniques.md
3. canSum          "Can we reach target?"    bool                can_sum_techniques.md
4. howSum          "Give me any path"        vector<int>         how_sum_techniques.md
5. bestSum         "Give me shortest path"   vector<int>         best_sum_techniques.md
6. canConstruct    "Can we build string?"    bool                can_construct_techniques.md
7. countConstruct  "How many ways?"          int                 count_construct_techniques.md
8. allConstruct    "Show me all ways"        vector<vector<T>>   all_construct_techniques.md
```

---

## PART 1 — Pattern Recognition: Which Pattern Is This?

```
STEP 1: IS THE PROBLEM ABOUT NUMBERS OR STRINGS?
  Numbers (add up to target) → Patterns 1, 2, 3, 4, 5
  Strings (concatenate to target) → Patterns 6, 7, 8

STEP 2 (for numbers): WHAT DOES IT RETURN?
  true/false → canSum (#3)
  any one combination → howSum (#4)
  shortest/minimum combination → bestSum (#5)
  count of ways → countConstruct number version (#7)
  all combinations → allConstruct number version (#8)

STEP 3 (for strings): WHAT DOES IT RETURN?
  true/false → canConstruct (#6)
  count of ways → countConstruct (#7)
  all combinations → allConstruct (#8)

STEP 4: IS IT 2D / GRID-BASED?
  Moving through a grid → Grid Traveler (#2)
  Plain sequence / single dimension → Fibonacci (#1) or canSum (#3)

STEP 5: MAGIC WORDS TO PATTERN MAP
  "minimum number of..."    → bestSum / Coin Change variant
  "how many ways..."        → countConstruct / Fibonacci variant
  "is it possible..."       → canSum / canConstruct
  "all possible..."         → allConstruct
  "any combination..."      → howSum
  "longest..."              → LIS / LCS family (separate pattern)
  "word break"              → canConstruct / countConstruct
  "unique paths"            → Grid Traveler
  "climbing stairs"         → Fibonacci
  "decode ways"             → countConstruct number variant
```

---

## PART 2 — Pattern 1: Fibonacci / Linear 1D DP

```
SIGNATURE: F(n) = F(n-1) + F(n-2)  [or similar linear recurrence]
COMPLEXITY: Recursion O(2^n) → Memoization/Tabulation O(n) → Space-opt O(1)
KEY INSIGHT: Current answer depends on a fixed number of previous answers.
             Build bottom-up. Often space-optimizable to O(1).
```

### Q1 — Climbing Stairs (LeetCode #70)  ★★ [Easy]

```
PROBLEM:
  You are climbing n stairs. You can climb 1 or 2 steps at a time.
  How many distinct ways can you reach the top?

WHY IT'S FIBONACCI:
  ways(n) = ways(n-1) + ways(n-2)
  From step n-1: take 1 step.
  From step n-2: take 2 steps.
  → Exactly Fibonacci!

BASE CASES:
  ways(0) = 1  (at top, done)
  ways(1) = 1  (1 step only)
  ways(2) = 2  (1+1 or 2)

EXAMPLES:
  n=1 → 1         n=2 → 2        n=3 → 3
  n=4 → 5         n=5 → 8

TABULATION SOLUTION:
  vector<int> dp(n+1);
  dp[0] = 1; dp[1] = 1;
  for(int i=2; i<=n; i++) dp[i] = dp[i-1] + dp[i-2];
  return dp[n];

SPACE-OPTIMIZED (O(1)):
  int a=1, b=1;
  for(int i=2; i<=n; i++) { int c = a+b; a=b; b=c; }
  return b;

COMPLEXITY: O(n) time, O(1) space (optimized)

INTERVIEWER TESTS: Can you reduce to O(1) space? Can you handle large n (use long long)?
FOLLOW-UP: What if you can climb 1, 2, or 3 steps? → Tribonacci pattern.
```

### Q2 — Min Cost Climbing Stairs (LeetCode #746)  ★★ [Easy]

```
PROBLEM:
  Each step i has cost[i]. You can start from step 0 or 1.
  After paying, you can climb 1 or 2 steps.
  Find minimum cost to reach the top (past last step).

PATTERN: Fibonacci variant — minimize instead of count.

dp[i] = min cost to reach step i
dp[i] = min(dp[i-1], dp[i-2]) + cost[i]   ← pay current cost, come from best previous

BASE CASES:
  dp[0] = cost[0]
  dp[1] = cost[1]
ANSWER: min(dp[n-1], dp[n-2])

COMPLEXITY: O(n) time, O(1) space (use two variables)
INTERVIEWER TESTS: Correctly handling the "top" (past last step).
```

### Q3 — House Robber (LeetCode #198)  ★★★ [Medium]

```
PROBLEM:
  Array of houses with money. Cannot rob two adjacent houses.
  Maximize total money robbed.

dp[i] = max money from houses 0..i
dp[i] = max(dp[i-1], dp[i-2] + nums[i])
       ↑ skip house i   ↑ rob house i (skip i-1)

BASE CASES: dp[0]=nums[0],  dp[1]=max(nums[0],nums[1])

COMPLEXITY: O(n) time, O(1) space (two variables)
FOLLOW-UPS:
  House Robber II (LC 213): houses in a circle → run twice (skip first or skip last)
  House Robber III (LC 337): houses in a tree → tree DP

INTERVIEWER TESTS: Can you handle circular/tree variants?
```

### Q4 — Decode Ways (LeetCode #91)  ★★★ [Medium]

```
PROBLEM:
  A message encoded: 'A'→1, 'B'→2, ..., 'Z'→26.
  Given a string of digits, count the number of ways to decode it.
  "12" → "AB"(1,2) or "L"(12) → 2 ways

PATTERN: countConstruct NUMBER VERSION — Fibonacci with conditions.

dp[i] = ways to decode s[0..i-1]
dp[0] = 1  (empty string: 1 way)
dp[1] = s[0]!='0' ? 1 : 0

For each i from 2 to n:
  oneDigit = s[i-1] - '0'          // last 1 char
  twoDigit = stoi(s.substr(i-2,2)) // last 2 chars
  if(oneDigit >= 1) dp[i] += dp[i-1];         // valid single digit
  if(twoDigit >= 10 && twoDigit <= 26) dp[i] += dp[i-2];  // valid double digit

TRICKY CASES: '0' alone is invalid; "06" is NOT 6 (leading zero invalid).

COMPLEXITY: O(n) time, O(n) space → O(1) optimized
FOLLOW-UP: Decode Ways II (LC 639): '*' can be 1-9 → multiply counts
```

### Q5 — Fibonacci Number (LeetCode #509)  ★ [Easy]

```
PROBLEM: Return F(n).
ANSWER: Classic tabulation or space-optimized two-variable approach.
COMPLEXITY: O(n) time, O(1) space.
NOTE: Can also be solved in O(log n) via matrix exponentiation (rare interview ask).
```

### Q6 — N-th Tribonacci Number (LeetCode #1137)  ★ [Easy]

```
T(n) = T(n-1) + T(n-2) + T(n-3)
Same pattern as Fibonacci — extend to 3 previous values.
Space: O(1) with 3 rolling variables.
```

---

## PART 3 — Pattern 2: Grid Traveler / 2D DP

```
SIGNATURE: dp[i][j] = f(dp[i-1][j], dp[i][j-1])  [or similar 2D recurrence]
COMPLEXITY: O(m×n) time, O(m×n) space → often O(n) space optimized
KEY INSIGHT: Each cell depends on cells above and to the left.
             Fill row by row, left to right.
```

### Q7 — Unique Paths (LeetCode #62)  ★★ [Medium]

```
PROBLEM:
  m × n grid. Start top-left, end bottom-right.
  Can only move RIGHT or DOWN. Count unique paths.

PATTERN: Grid Traveler exactly.

dp[i][j] = paths to reach cell (i,j)
dp[i][j] = dp[i-1][j] + dp[i][j-1]   ← from above + from left
BASE: dp[0][j] = 1, dp[i][0] = 1  (only one way along edges)

SPACE OPTIMIZATION: Use single 1D array dp[n], update in-place.
  dp[j] += dp[j-1]  ← dp[j] was dp[i-1][j], dp[j-1] is dp[i][j-1]

COMPLEXITY: O(m×n) time, O(n) space (optimized)
MATH SHORTCUT: C(m+n-2, m-1) — choose m-1 down moves from total m+n-2 moves.
```

### Q8 — Unique Paths II (LeetCode #63)  ★★★ [Medium]

```
PROBLEM: Same as Unique Paths but grid has obstacles (marked as 1).

CHANGE: if(grid[i][j] == 1) dp[i][j] = 0;  ← obstacle blocks all paths
        else dp[i][j] = dp[i-1][j] + dp[i][j-1];

TRICKY: Start or end cell blocked → answer is 0.
COMPLEXITY: O(m×n) time, O(n) space
```

### Q9 — Minimum Path Sum (LeetCode #64)  ★★★ [Medium]

```
PROBLEM:
  m × n grid with non-negative integers.
  Find path from top-left to bottom-right that minimizes the sum.

dp[i][j] = min(dp[i-1][j], dp[i][j-1]) + grid[i][j]
BASE: dp[0][0]=grid[0][0], fill first row and first column separately.

COMPLEXITY: O(m×n) time, O(1) space (modify grid in-place)
```

### Q10 — Triangle (LeetCode #120)  ★★★ [Medium]

```
PROBLEM:
  Triangle of integers. Find the minimum path sum from top to bottom.
  Each step you may move to adjacent numbers in the row below.

TRICK: Solve BOTTOM-UP — start from second-to-last row, update upward.
  dp[i][j] = triangle[i][j] + min(dp[i+1][j], dp[i+1][j+1])
ANSWER: dp[0][0]

BONUS: Can be done with O(n) extra space using only the bottom row.
COMPLEXITY: O(n²) time, O(n) space
```

### Q11 — Maximal Square (LeetCode #221)  ★★★ [Medium]

```
PROBLEM:
  Binary matrix. Find the largest square containing only 1s.
  Return the area.

KEY INSIGHT:
  dp[i][j] = side length of largest square with bottom-right at (i,j)
  dp[i][j] = min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1]) + 1
             (when matrix[i][j] == '1')

COMPLEXITY: O(m×n) time, O(m×n) space → O(n) optimized
```

---

## PART 4 — Pattern 3: canSum → Reachability with Numbers

```
SIGNATURE: Can we reach targetSum using numbers from array (reusable)?
RETURNS: true / false
COMPLEXITY: O(n×m) time, O(m) space  [m=targetSum, n=array size]
KEY INSIGHT: dp[i] = true means sum i is reachable.
             Spread: if dp[i] is true, dp[i+num] = true for each num.
```

### Q12 — Coin Change Feasibility  ★★ [Easy-Medium]

```
PROBLEM:
  Given coin denominations and a target amount.
  Can you make exact change?

EXACT canSum PATTERN:
  dp[i] = can we make amount i?
  dp[0] = true
  For each i, for each coin: if dp[i] and i+coin<=target → dp[i+coin]=true
  Return dp[target]

NOTE: LeetCode's Coin Change (LC 322) asks for MINIMUM coins → bestSum pattern.
      This feasibility version is a common interview warmup.
COMPLEXITY: O(n×m) time, O(m) space
```

### Q13 — Partition Equal Subset Sum (LeetCode #416)  ★★★ [Medium]

```
PROBLEM:
  Given an array of positive integers.
  Can you partition it into two subsets with equal sum?

REDUCTION TO canSum:
  totalSum = sum of all elements.
  If totalSum is odd → false (can't split evenly).
  target = totalSum / 2.
  Can we pick elements (each used at most once) to reach target?

THIS IS canSum — but each number used AT MOST ONCE (0/1 knapsack)!
  Iterate i from target DOWN to num (to avoid reusing same element):
  dp[i] = dp[i] || dp[i - num]

BASE: dp[0] = true
COMPLEXITY: O(n×target) time, O(target) space
INTERVIEWER TESTS: 0/1 knapsack vs unbounded knapsack distinction.
```

### Q14 — Jump Game (LeetCode #55)  ★★★ [Medium]

```
PROBLEM:
  Array where nums[i] = max jump from position i.
  Can you reach the last index from index 0?

GREEDY (optimal): Track max reachable index. O(n) time.
  int maxReach = 0;
  for(int i=0; i<=maxReach && i<n; i++) maxReach = max(maxReach, i+nums[i]);
  return maxReach >= n-1;

DP APPROACH (less optimal but shows the pattern):
  dp[i] = can we reach index i?
  dp[0] = true
  For each reachable i: for k=1 to nums[i]: dp[i+k] = true
  Return dp[n-1]

COMPLEXITY: O(n²) DP, O(n) greedy
NOTE: Greedy always preferred in interview for O(n) — shows deeper insight.
```

### Q15 — Target Sum (LeetCode #494)  ★★★ [Medium]

```
PROBLEM:
  Array of integers. Assign + or - to each. Count ways to reach target sum.

REDUCTION:
  Let P = sum of positives, N = sum of negatives (|N|).
  P - N = target, P + N = totalSum
  → P = (target + totalSum) / 2
  Count subsets that sum to P (if valid integer).

PATTERN: countConstruct NUMBER VERSION with 0/1 knapsack.
  dp[j] += dp[j - num]  (iterate j from target down to num)

COMPLEXITY: O(n×target) time, O(target) space
```

---

## PART 5 — Pattern 4: howSum → Find Any One Combination

```
SIGNATURE: Give any one combination of numbers that sums to target.
RETURNS: vector<int> (the combination) or empty (impossible)
COMPLEXITY: O(n×m²) time, O(m²) space  [m=targetSum]
KEY INSIGHT: Like canSum, but carry the combination backward through the call stack.
```

### Q16 — Combination Sum (LeetCode #39)  ★★★ [Medium]

```
PROBLEM:
  Array of distinct integers. Find ONE combination summing to target.
  Numbers can be reused. Return any valid combination.

EXACT howSum PATTERN (first result only):
  Recursive: try each number, subtract, recurse on remainder.
  Return the first combination found.

COMPLEXITY: O(n^target × target) naive, O(n × target²) with memo
NOTE: LeetCode #39 asks for ALL combinations → allConstruct number version.
      This "find any one" is commonly asked in phone screens.
```

### Q17 — Combination Sum IV (LeetCode #377)  ★★★ [Medium]

```
PROBLEM:
  Count ways to reach target by ORDERED selections from array.
  [1,1,2] and [1,2,1] and [2,1,1] are different (order matters).

THIS IS countConstruct NUMBER VERSION:
  dp[i] = number of ordered ways to reach sum i
  dp[0] = 1
  For each i: for each num: dp[i] += dp[i - num]   (if i >= num)

COMPLEXITY: O(n×target) time, O(target) space
CONTRAST: Coin Change 2 (LC 518) counts UNORDERED combinations.
          Order outer/inner loops differently → key interview question!
```

---

## PART 6 — Pattern 5: bestSum → Minimum / Shortest Combination

```
SIGNATURE: Find the FEWEST numbers from array that sum to target.
RETURNS: vector<int> (shortest combo) or impossible
COMPLEXITY: O(n×m²) time, O(m²) space
KEY INSIGHT: Like howSum but try ALL branches and keep the shortest.
             In tabulation: dp[j] updates if new combination is shorter.
```

### Q18 — Coin Change (LeetCode #322)  ★★★★ [Medium]

```
PROBLEM:
  Array of coin denominations. Minimum number of coins to make amount.
  Return -1 if impossible.

EXACT bestSum PATTERN (but track count, not actual coins):
  dp[i] = min coins to make amount i
  dp[0] = 0
  dp[i] = INT_MAX  (impossible)
  For each i: for each coin:
    if(i >= coin && dp[i-coin] != INT_MAX)
      dp[i] = min(dp[i], dp[i-coin] + 1)
  Return dp[amount] == INT_MAX ? -1 : dp[amount]

OPTIMIZATION: Store count (int), not the actual coin vector.
              This is O(n×m) time, O(m) space — much better than O(n×m²)!

COMPLEXITY: O(n×amount) time, O(amount) space
TRICK: INT_MAX + 1 causes overflow — use a safe sentinel or check.
FOLLOW-UP: Return actual coins used → track back via dp array.
           Exactly bestSum but store int not vector<int>.
```

### Q19 — Perfect Squares (LeetCode #279)  ★★★ [Medium]

```
PROBLEM:
  Given n. Find minimum number of perfect squares (1,4,9,16...) that sum to n.

EXACT bestSum / Coin Change pattern — coins are perfect squares.
  squares = [1, 4, 9, 16, ...] (all ≤ n)
  dp[i] = min squares to sum to i
  dp[0] = 0;  dp[i] = INT_MAX initially.
  For each i, for each sq in squares:
    if(i >= sq) dp[i] = min(dp[i], dp[i-sq]+1)

COMPLEXITY: O(n × √n) time, O(n) space
MATH FACT: By Lagrange's four-square theorem, answer is always 1, 2, 3, or 4.
```

### Q20 — Jump Game II (LeetCode #45)  ★★★★ [Medium]

```
PROBLEM:
  Array where nums[i] = max jump from i. Find minimum jumps to reach end.

GREEDY (O(n)) — preferred:
  Track current range and next range.
  When you exhaust current range, increment jump count.

DP approach (O(n²)):
  dp[i] = min jumps to reach index i
  dp[0] = 0; dp[others] = INT_MAX
  For each i reachable: for k=1 to nums[i]:
    dp[i+k] = min(dp[i+k], dp[i]+1)

COMPLEXITY: O(n) greedy, O(n²) DP
INTERVIEWER TESTS: Can you solve in O(n)? Greedy shows higher insight.
```

### Q21 — Word Break with Minimum Cuts  ★★★★ [Medium]

```
PROBLEM:
  Given target string and wordBank. Find the minimum number of words
  needed to build target (or minimum cuts/splits).

PATTERN: bestSum applied to strings (canConstruct + length tracking).
  dp[i] = min words to construct target[0..i-1]
  dp[0] = 0
  For each i, for each word matching at position j (j < i):
    dp[i] = min(dp[i], dp[j] + 1)

COMPLEXITY: O(n × m²) time, O(m) space
```

---

## PART 7 — Pattern 6: canConstruct → String Reachability

```
SIGNATURE: Can target string be built by concatenating words from wordBank?
RETURNS: true / false
COMPLEXITY: O(n × m²) time  [n=wordBank size, m=target length]
            O(m) space (tabulation) or O(m²) space (memoization)
KEY INSIGHT: dp[i] = can target[0..i-1] be constructed?
             dp[0] = true. Spread: if dp[i] and word matches → dp[i+wlen]=true.
```

### Q22 — Word Break (LeetCode #139)  ★★★★ [Medium]  ← Most Asked!

```
PROBLEM:
  Given string s and list of words wordDict.
  Can s be segmented into words from wordDict?

EXACT canConstruct PATTERN:
  dp[i] = can s[0..i-1] be built from wordDict?
  dp[0] = true
  For each i: for each word in wordDict:
    if(i >= word.size() && dp[i - word.size()] &&
       s.substr(i - word.size(), word.size()) == word)
      dp[i] = true;
  Return dp[s.size()]

ALTERNATIVE: Check from position i forward (same as our doc's approach):
  for(int i=0; i<=m; i++) {
    if(!dp[i]) continue;
    for(auto& w : wordDict)
      if(s.substr(i, w.size()) == w) dp[i + w.size()] = true;
  }

COMPLEXITY: O(n × m²) time, O(m) space
FOLLOW-UPS:
  → Word Break II (LC 140): return all ways → allConstruct
  → Word Break count: count ways → countConstruct
  → These three together form a natural progression interviewers love!
```

### Q23 — Concatenated Words (LeetCode #472)  ★★★★★ [Hard]

```
PROBLEM:
  Given array of words. Find all words that can be formed by
  concatenating two or more shorter words from the same list.

PATTERN: canConstruct applied to each word, using rest of list as wordBank.
  For each word: run canConstruct(word, wordBank - {word})
  If canConstruct returns true → add to result.

COMPLEXITY: O(k × n × m²) where k = number of words to check
INTERVIEWER TESTS: Can you handle the "exclude itself" edge case?
```

---

## PART 8 — Pattern 7: countConstruct → Count All Ways (Strings)

```
SIGNATURE: How many ways can target be built from wordBank?
RETURNS: int
COMPLEXITY: O(n × m²) time, O(m) space (tabulation)
KEY INSIGHT: dp[i] = number of ways to build target[0..i-1].
             dp[0] = 1. Accumulate: dp[i + wlen] += dp[i].
```

### Q24 — Word Break II — Count Only (Variant)  ★★★ [Medium]

```
PROBLEM:
  Count the number of ways to segment string s using wordDict.

EXACT countConstruct PATTERN:
  dp[i] = number of ways to build s[0..i-1]
  dp[0] = 1
  For each i, for each word: dp[i + wlen] += dp[i]
  Return dp[s.size()]

COMPLEXITY: O(n × m²) time, O(m) space
```

### Q25 — Coin Change 2 (LeetCode #518)  ★★★ [Medium]

```
PROBLEM:
  Count the number of COMBINATIONS (unordered) of coins summing to amount.
  [1,2] and [2,1] are the same combination (counted once).

countConstruct NUMBER VERSION — UNORDERED:
  Loop coins in OUTER, amounts in INNER:
  for(int coin : coins)
    for(int j = coin; j <= amount; j++)
      dp[j] += dp[j - coin];

  This ensures each coin is considered after all smaller coins,
  preventing [1,2] and [2,1] from being double-counted.

CONTRAST WITH Combination Sum IV (LC 377) — ORDERED:
  Loop amounts in OUTER, coins in INNER:
  for(int j = 1; j <= amount; j++)
    for(int coin : coins)
      if(j >= coin) dp[j] += dp[j - coin];

  Order of loops = ORDERED vs UNORDERED. This is the classic interview trap!

dp[0] = 1  (1 way to make 0: use no coins)
COMPLEXITY: O(n × amount) time, O(amount) space
INTERVIEWER TESTS: Know the loop order difference!
```

### Q26 — Climbing Stairs Variants  ★★ [Easy-Medium]

```
VARIANT 1: "You can climb 1, 2, or 3 steps at a time."
  This is countConstruct number version with "wordBank" = [1, 2, 3].
  dp[i] += dp[i-1] + dp[i-2] + dp[i-3]
  → Tribonacci!

VARIANT 2: "Some steps are broken (can't land on them)."
  if(broken step) dp[i] = 0.
  Otherwise dp[i] = dp[i-1] + dp[i-2].

VARIANT 3: "Cost to use each step. Count paths within budget."
  2D DP: dp[i][cost] = ways to reach step i spending exactly 'cost'.
```

### Q27 — Unique Paths — Count Variant  ★★ [Medium]

```
The grid traveler problem IS countConstruct in 2D:
  dp[i][j] = ways to reach cell (i,j)
  dp[i][j] = dp[i-1][j] + dp[i][j-1]
  dp[0][j] = dp[i][0] = 1

Exact countConstruct structure, just in 2 dimensions.
```

---

## PART 9 — Pattern 8: allConstruct → List All Combinations

```
SIGNATURE: Return ALL ways to build target from wordBank.
RETURNS: vector<vector<string>>
COMPLEXITY: O(n^m × m) time and space — output-dominated. Memoization doesn't help!
KEY INSIGHT: dp[i] = all combinations that build target[0..i-1].
             Extend each combination by the matching word.
```

### Q28 — Word Break II (LeetCode #140)  ★★★★★ [Hard]  ← Most Asked allConstruct!

```
PROBLEM:
  Given string s and wordDict. Return all ways to segment s into words.

EXACT allConstruct PATTERN:
  dp[i] = all segmentations of s[0..i-1]
  dp[0] = [[]]  (one way: empty list)
  For each i, for each word matching at position j:
    for(each way in dp[j]):
      newWay = way + [word]
      dp[j + word.size()].push_back(newWay)
  Return dp[s.size()]

OUTPUT FORMAT: Usually want space-separated strings, not vectors.
  Transform: ["cat","sat","on","mat"] → "cat sat on mat"

COMPLEXITY: O(n^m × m) time and space — output-dominated.
INTERVIEWER TESTS: Why doesn't memoization help here?
  Answer: "The output itself can be exponentially large. Copying cached
           results costs proportional to their size — same as recomputing."
```

### Q29 — Combination Sum (LeetCode #39) — All Combinations  ★★★ [Medium]

```
PROBLEM:
  Array of distinct positive integers. Find ALL combinations summing to target.
  Numbers reusable. Return all unique combinations.

EXACT allConstruct NUMBER VERSION:
  result = []
  dfs(target, current_path, start_index):
    if target == 0: result.push_back(current_path)
    for i from start_index to n:
      if candidates[i] <= target:
        current_path.push_back(candidates[i])
        dfs(target - candidates[i], current_path, i)  // i, not i+1 (reuse!)
        current_path.pop_back()

  (Backtracking DFS — cleaner than tabulation for this variant)

COMPLEXITY: O(n^target × target) — output-dominated
FOLLOW-UP: Combination Sum II (LC 40): each number used once → use i+1, handle duplicates
```

### Q30 — Subsets (LeetCode #78)  ★★★ [Medium]

```
PROBLEM: Return all possible subsets (power set) of an array.

PATTERN: allConstruct variant — enumerate all combinations (choosing or skipping).
  Classic backtracking:
  dfs(index, current):
    result.push_back(current)  // add current subset
    for i from index to n:
      current.push_back(nums[i])
      dfs(i+1, current)
      current.pop_back()

COMPLEXITY: O(2^n × n) — 2^n subsets, each up to n elements
```

---

## PART 10 — The Top 20 Most Asked DP Interview Questions

```
RANK  PROBLEM                           PATTERN              LC #   DIFFICULTY
────────────────────────────────────────────────────────────────────────────────
 1    Climbing Stairs                   Fibonacci            70     Easy
 2    Coin Change (min coins)           bestSum              322    Medium ★
 3    Word Break                        canConstruct         139    Medium ★★
 4    Unique Paths                      Grid Traveler        62     Medium
 5    Longest Common Subsequence        LCS (bonus)          1143   Medium ★★
 6    House Robber                      Fibonacci            198    Medium
 7    Decode Ways                       countConstruct       91     Medium ★
 8    Partition Equal Subset Sum        canSum (0/1)         416    Medium ★
 9    Coin Change 2 (count combos)      countConstruct       518    Medium ★
10    Jump Game                         canSum/Greedy        55     Medium
11    Word Break II (all ways)          allConstruct         140    Hard ★★
12    Target Sum                        countConstruct       494    Medium
13    Combination Sum (all combos)      allConstruct         39     Medium
14    Perfect Squares                   bestSum              279    Medium
15    Minimum Path Sum                  Grid DP              64     Medium
16    Longest Increasing Subsequence    LIS (bonus)          300    Medium ★★
17    Jump Game II (min jumps)          bestSum/Greedy       45     Medium ★
18    Combination Sum IV (count)        countConstruct       377    Medium ★
19    Maximal Square                    Grid DP              221    Medium
20    Decode Ways II                    countConstruct       639    Hard

★ = Pattern directly from our studied documents
★★ = Slight variation requiring extra insight
```

---

## PART 11 — The 5-Step Approach to ANY DP Problem in Interview

```
STEP 1: UNDERSTAND THE PROBLEM (1-2 min)
  □ Write 2-3 concrete input-output examples
  □ Identify: what am I optimizing/counting/finding?
  □ Are inputs reusable or single-use?
  □ Identify the "target" (what am I working toward?)

STEP 2: IDENTIFY THE PATTERN (30 sec)
  → See Part 1 of this document for the decision tree.
  → Most DP problems map to one of the 8 patterns.
  State it aloud: "This is the canSum pattern because..."

STEP 3: DEFINE THE SUBPROBLEM (1 min)
  → What does dp[i] or dp[i][j] MEAN in words?
  → Write this as a comment before writing any code.
  Bad: "dp[i] = some DP value"
  Good: "dp[i] = minimum coins needed to make amount i"
        "dp[i][j] = number of paths to reach cell (i,j)"

STEP 4: WRITE RECURSIVE SOLUTION + BASE CASES (2 min)
  → Start with recursion — it's closest to the mathematical definition.
  → Identify base cases (when do we stop recursing?).
  → Add memoization (map/vector + cache check before recursing).

STEP 5: CONVERT TO TABULATION (2 min)
  → Identify loop direction (bottom-up).
  → Initialize dp array with base case values.
  → Write the two nested loops.
  → State complexity: time and space.

BONUS STEP: OPTIMIZE SPACE
  → If dp[i] only depends on dp[i-1] and dp[i-2]: use two variables O(1).
  → If dp[i][j] only depends on previous row: use 1D array O(n).
```

---

## PART 12 — Ordered vs Unordered: The Classic Interview Trap

```
This is asked in almost every senior interview involving "count ways."

QUESTION: "Does the order of selection matter?"

ORDERED (Permutations) — [1,2] and [2,1] are DIFFERENT:
  Loop amounts in OUTER, items in INNER:
  for(int j = 1; j <= target; j++)
    for(int item : items)
      dp[j] += dp[j - item]

  Example: Combination Sum IV (LC 377)
  Ways to make 3 with [1,2]: 1+1+1, 1+2, 2+1 = 3 ways

UNORDERED (Combinations) — [1,2] and [2,1] are SAME:
  Loop items in OUTER, amounts in INNER:
  for(int item : items)
    for(int j = item; j <= target; j++)
      dp[j] += dp[j - item]

  Example: Coin Change 2 (LC 518)
  Ways to make 3 with [1,2]: {1,1,1}, {1,2} = 2 combinations

WHY DOES LOOP ORDER MATTER?
  ORDERED (target outer): When computing dp[j], all items are considered.
    → Each item can follow any previous selection → order matters.
  UNORDERED (item outer): When processing item k, only items 1..k are available.
    → Previous selections are always of items ≤ k → no repetition of order.

INTERVIEW TIP: State this distinction clearly even if not asked.
               It shows mastery beyond just knowing the algorithm.
```

---

## PART 13 — 0/1 Knapsack vs Unbounded Knapsack

```
UNBOUNDED KNAPSACK (items reusable) — canSum / howSum / bestSum:
  for(int i = 0; i <= target; i++)
    for(int item : items)
      if(i >= item && dp[i-item]) dp[i] = true/count/min

  Iterate target from LOW to HIGH → same item can be reused!

0/1 KNAPSACK (each item used at most once) — Partition Sum variant:
  for(int item : items)
    for(int j = target; j >= item; j--)   ← iterate HIGH to LOW!
      dp[j] = dp[j] || dp[j - item];       (or += or min)

  Iterating HIGH to LOW prevents using the SAME item twice:
  If we process item=3 and dp[6] uses dp[3] which already used this item,
  iterating backward ensures dp[3] is still from BEFORE this item was added.

MEMORY AID:
  Reusable items (coins, stairs) → forward loop (low to high)
  Single-use items (subset sum)  → backward loop (high to low)
```

---

## PART 14 — Complexity Quick Reference

```
┌─────────────────────┬────────────────┬─────────────────┬───────────────────┐
│ Pattern             │ Time (memo/tab)│ Space (tab)     │ Key Operation     │
├─────────────────────┼────────────────┼─────────────────┼───────────────────┤
│ Fibonacci           │ O(n)           │ O(1) optimized  │ dp[i]=dp[i-1]+... │
│ Grid Traveler       │ O(m×n)         │ O(n) optimized  │ dp[i][j]=from top/left│
│ canSum              │ O(n×m)         │ O(m)            │ dp[j] = true      │
│ howSum              │ O(n×m²)        │ O(m²)           │ carry vector back │
│ bestSum             │ O(n×m²)        │ O(m²)           │ dp[j] if shorter  │
│ canConstruct        │ O(n×m²)        │ O(m)  ← tab wins│ dp[j] = true      │
│ countConstruct      │ O(n×m²)        │ O(m)  ← tab wins│ dp[j] += dp[i]   │
│ allConstruct        │ O(n^m × m)     │ O(n^m × m)      │ extend each combo │
├─────────────────────┼────────────────┼─────────────────┼───────────────────┤
│ Coin Change (min)   │ O(n×amount)    │ O(amount)       │ dp[j]=min(dp[j],dp[j-c]+1)│
│ Coin Change 2 (count)│ O(n×amount)  │ O(amount)       │ dp[j]+=dp[j-c]   │
│ Partition Sum       │ O(n×sum/2)     │ O(sum/2)        │ backward loop     │
│ Word Break          │ O(n×m²)        │ O(m)            │ dp[j]=true        │
│ Word Break II       │ O(n^m × m)     │ O(n^m × m)      │ extend all combos │
└─────────────────────┴────────────────┴─────────────────┴───────────────────┘

n = size of choices array / wordBank
m = target value / target string length
```

---

## PART 15 — Common Mistakes and How to Avoid Them

```
MISTAKE 1: Wrong base case
  canSum/canConstruct: dp[0] = true  (not false!)
  countConstruct:      dp[0] = 1     (not 0!)
  allConstruct:        dp[0] = [[]]  (not []!)
  FIX: Always state base case meaning in words before setting it.

MISTAKE 2: Off-by-one in dp array size
  Target sum = 7 → need dp[0..7] = vector of size 8 (target+1).
  String length = m → need dp[0..m] = vector of size m+1.
  FIX: dp size = target + 1, always.

MISTAKE 3: Wrong loop direction for 0/1 knapsack
  For single-use items: iterate HIGH to LOW (not low to high).
  FIX: When you see "each item at most once" → immediately reverse inner loop.

MISTAKE 4: Reusing the same item accidentally
  "Reusable" → forward inner loop.
  "Single-use" → backward inner loop.
  Mixing these up is the #1 interview mistake in DP.

MISTAKE 5: Integer overflow in countConstruct / count problems
  dp[i] accumulates — can exceed INT_MAX for large inputs.
  FIX: Use long long if n > 10^9 or result could be large.

MISTAKE 6: Forgetting to check bounds in tabulation
  if(i + word.length() > target): skip (don't access dp out of bounds).
  if(j - coin < 0): skip.

MISTAKE 7: allConstruct base case [[]] vs []
  [[]] = one way: empty combination.
  []   = zero ways: impossible.
  WRONG: return {} in base case → loop never adds anything → always returns []
  RIGHT: return {{}} in base case → prepend works correctly.

MISTAKE 8: Returning early in countConstruct / allConstruct
  Adding "return" after finding one path → becomes howSum/canConstruct instead.
  FIX: Remove return; let the loop complete and accumulate all results.
```

---

## PART 16 — What Interviewers Are Really Testing

```
LEVEL 1 — Can you code it? (Junior)
  Can you write working recursive code with base cases?
  Can you identify the subproblem structure?

LEVEL 2 — Can you optimize? (Mid-level)
  Can you add memoization to eliminate repeated computation?
  Can you convert to tabulation?
  Do you know why tabulation avoids stack overflow?

LEVEL 3 — Do you understand complexity? (Senior)
  Can you state time AND space complexity for all three approaches?
  Do you know when memoization space > tabulation space (canConstruct)?
  Do you know when memoization doesn't help at all (allConstruct)?

LEVEL 4 — Can you see connections? (Staff+)
  Can you explain how allConstruct relates to countConstruct?
  Can you explain the ordered vs unordered distinction in loop design?
  Can you reduce Partition Sum to canSum?
  Can you explain why 0/1 knapsack needs backward iteration?

LEVEL 5 — Can you handle variants on the fly?
  "Now what if numbers can only be used once?"
  "Now what if we also want the actual combination?"
  "Now what if the grid has obstacles?"
  "What if the string has wildcards?"
```

---

## PART 17 — Pattern Template Code (Copy-Paste Starting Points)

### Template A — Fibonacci Pattern

```cpp
// dp[i] depends on dp[i-1] and dp[i-2]
int fib_style(int n) {
    if(n <= 1) return n;
    vector<long long> dp(n+1);
    dp[0] = BASE_0;
    dp[1] = BASE_1;
    for(int i=2; i<=n; i++)
        dp[i] = dp[i-1] + dp[i-2];  // or min/max
    return dp[n];
}
// Space-opt: use two variables (a, b) instead of array
```

### Template B — Grid Traveler Pattern

```cpp
// dp[i][j] depends on dp[i-1][j] and dp[i][j-1]
int grid_style(int m, int n) {
    vector<vector<long long>> dp(m, vector<long long>(n, 0));
    // Initialize first row and col
    for(int i=0; i<m; i++) dp[i][0] = 1;
    for(int j=0; j<n; j++) dp[0][j] = 1;
    for(int i=1; i<m; i++)
        for(int j=1; j<n; j++)
            dp[i][j] = dp[i-1][j] + dp[i][j-1];
    return dp[m-1][n-1];
}
```

### Template C — canSum / canConstruct Pattern

```cpp
// "Is it possible?" → boolean dp
bool can_style(int target, vector<int>& nums) {
    vector<bool> dp(target+1, false);
    dp[0] = true;
    for(int i=0; i<=target; i++) {
        if(!dp[i]) continue;
        for(int num : nums)
            if(i+num <= target) dp[i+num] = true;
    }
    return dp[target];
}
```

### Template D — bestSum / Coin Change Pattern

```cpp
// "Minimum count?" → int dp, initialized to INT_MAX
int best_style(int target, vector<int>& nums) {
    vector<int> dp(target+1, INT_MAX);
    dp[0] = 0;
    for(int i=0; i<=target; i++) {
        if(dp[i] == INT_MAX) continue;
        for(int num : nums)
            if(i+num <= target)
                dp[i+num] = min(dp[i+num], dp[i]+1);
    }
    return dp[target] == INT_MAX ? -1 : dp[target];
}
```

### Template E — countConstruct / Combination Count Pattern

```cpp
// "How many ways?" → int dp
// ORDERED (order matters — like Combination Sum IV):
int count_ordered(int target, vector<int>& nums) {
    vector<long long> dp(target+1, 0);
    dp[0] = 1;
    for(int i=1; i<=target; i++)           // amount outer
        for(int num : nums)                // item inner
            if(i >= num) dp[i] += dp[i-num];
    return dp[target];
}

// UNORDERED (like Coin Change 2):
int count_unordered(int target, vector<int>& nums) {
    vector<long long> dp(target+1, 0);
    dp[0] = 1;
    for(int num : nums)                    // item outer
        for(int i=num; i<=target; i++)     // amount inner
            dp[i] += dp[i-num];
    return dp[target];
}
```

### Template F — 0/1 Knapsack Pattern

```cpp
// Single-use items — backward loop prevents reuse
bool knapsack_01(int target, vector<int>& nums) {
    vector<bool> dp(target+1, false);
    dp[0] = true;
    for(int num : nums)
        for(int j=target; j>=num; j--)    // HIGH to LOW!
            dp[j] = dp[j] || dp[j-num];
    return dp[target];
}
```

### Template G — canConstruct / Word Break Pattern

```cpp
// String version — check word match at each position
bool word_break(string& s, vector<string>& wordDict) {
    int m = s.size();
    vector<bool> dp(m+1, false);
    dp[0] = true;
    for(int i=0; i<=m; i++) {
        if(!dp[i]) continue;
        for(auto& w : wordDict) {
            int wl = w.size();
            if(i+wl<=m && s.substr(i,wl)==w) dp[i+wl]=true;
        }
    }
    return dp[m];
}
```

### Template H — allConstruct / Word Break II Pattern

```cpp
// Return ALL combinations
vector<vector<string>> word_break_all(string& s, vector<string>& wordDict) {
    int m = s.size();
    vector<vector<vector<string>>> dp(m+1);
    dp[0] = {{}};
    for(int i=0; i<=m; i++) {
        if(dp[i].empty()) continue;
        for(auto& w : wordDict) {
            int wl = w.size();
            if(i+wl<=m && s.substr(i,wl)==w) {
                for(auto& way : dp[i]) {
                    auto nw = way;
                    nw.push_back(w);
                    dp[i+wl].push_back(nw);
                }
            }
        }
    }
    return dp[m];
}
```

---

## PART 18 — Bonus: DP Patterns NOT in Our Docs (Brief Overview)

```
These appear in interviews and extend the 8 patterns you know:

LCS — Longest Common Subsequence (LC 1143)
  dp[i][j] = LCS length of s1[0..i-1] and s2[0..j-1]
  if(s1[i-1]==s2[j-1]): dp[i][j] = dp[i-1][j-1] + 1
  else: dp[i][j] = max(dp[i-1][j], dp[i][j-1])
  O(m×n) time, O(m×n) space → O(n) optimized

LIS — Longest Increasing Subsequence (LC 300)
  dp[i] = LIS ending at index i
  dp[i] = max(dp[j]+1) for all j<i where nums[j]<nums[i]
  O(n²) → O(n log n) with patience sorting/binary search

Edit Distance (LC 72)
  dp[i][j] = min edits to convert s1[0..i-1] to s2[0..j-1]
  if(s1[i-1]==s2[j-1]): dp[i][j] = dp[i-1][j-1]
  else: dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])
                          ↑ delete      ↑ insert      ↑ replace

Palindrome Substrings / Longest Palindromic Subsequence
  dp[i][j] = answer for substring s[i..j]
  Expand from center / interval DP

Matrix Chain Multiplication / Burst Balloons
  Interval DP: dp[i][j] = answer for subarray arr[i..j]
  for(length 2 to n): for each window: try each split point k

These all follow the same MEMOIZATION → TABULATION progression.
Once you master the 8 patterns, these are natural extensions.
```

---

## PART 19 — Quick Pre-Interview Checklist

```
THINGS TO CONFIRM BEFORE CODING:
  □ Can items be reused? (unbounded vs 0/1 knapsack)
  □ Does order matter? (ordered vs unordered counting)
  □ What does dp[0] mean? (always verify base case with example)
  □ Is the answer dp[target] or min of last two or something else?
  □ What is the target? (sum? string length? grid cell?)
  □ Can the answer be 0 / false / impossible? How to return that?
  □ Could the count overflow int? (use long long if unsure)
  □ Should I optimize space after the main solution?

THINGS TO SAY IN INTERVIEW:
  "I notice this has overlapping subproblems — I'll add memoization."
  "I can convert this to tabulation to avoid the call stack."
  "The time complexity is O(n×m) because there are m unique subproblems,
   each trying n choices."
  "I can further optimize space from O(m) to O(1) by using two variables."
  "This is similar to Coin Change but with strings instead of numbers."
  "Order matters here because... / Order doesn't matter because..."

THINGS THAT IMPRESS:
  → Recognizing the pattern before writing code.
  → Explaining why memoization doesn't improve allConstruct.
  → Knowing the ordered vs unordered loop trick.
  → Knowing the backward loop for 0/1 knapsack.
  → Space-optimizing without being asked.
```

---

## Quick Reference — The 8 Patterns Summary

```
┌──────┬──────────────────┬───────────────────────────────────────────────────┐
│  #   │ Pattern Name     │ One-line: "What it does"                          │
├──────┼──────────────────┼───────────────────────────────────────────────────┤
│  1   │ Fibonacci        │ Current = sum/min/max of 1-3 previous values      │
│  2   │ Grid Traveler    │ Cell = sum/min/max of cell above + cell to left   │
│  3   │ canSum           │ "Can I reach target?" → dp[j]=true, spread        │
│  4   │ howSum           │ "Give me any path" → carry vector back up stack   │
│  5   │ bestSum          │ "Shortest path" → overwrite if shorter             │
│  6   │ canConstruct     │ canSum but with string prefix matching             │
│  7   │ countConstruct   │ "How many ways?" → dp[j]+=dp[i], accumulate       │
│  8   │ allConstruct     │ "All ways" → extend each combo, O(n^m) always     │
└──────┴──────────────────┴───────────────────────────────────────────────────┘

THE GOLDEN RULE:
  Same problem → same DP structure, whether numbers or strings.
  canSum ↔ canConstruct    (bool)
  howSum ↔ (howConstruct)  (any one combo)
  bestSum ↔ Coin Change    (minimum)
  countSum ↔ countConstruct (count ways)
  allSum ↔ allConstruct    (all combos)
```

---

*All patterns covered in detail in the individual technique documents in this folder.*
*Reference: fibonacci_techniques.md, grid_traveler_techniques.md, can_sum_techniques.md,*
*how_sum_techniques.md, best_sum_techniques.md, can_construct_techniques.md,*
*count_construct_techniques.md, all_construct_techniques.md*
