# Permutations, Combinations & Time Complexity
## How to Count, Think, and Calculate TC While Coding

> The secret to understanding Time Complexity is **counting**.
> Before you can count algorithm steps, you need to understand HOW to count.
> This guide builds from basic counting → P&C → TC patterns used daily in coding.

---

## Table of Contents

```
1. The Fundamental Counting Principles
2. Factorial — The Building Block
3. Permutations (Order MATTERS)
4. Combinations (Order DOESN'T Matter)
5. Pascal's Triangle — Combinations Visualized
6. How P&C Connect to Time Complexity
7. The "How Many Choices × How Many Steps" Framework
8. Big-O Cheat Sheet — All Common Complexities
9. How to Calculate TC While Coding — Step by Step
10. Recursion Tree Method — The Most Powerful Tool
11. Real Code Examples — TC Breakdown
12. Space Complexity Patterns
13. Quick Reference — TC of Common Algorithms
```

---

## 1. The Fundamental Counting Principles

### The Multiplication Rule — "AND"

**If event A can happen in `m` ways AND event B can happen in `n` ways,**
**then both together can happen in `m × n` ways.**

```
Example: You have 3 shirts and 4 pants.
  Outfits = 3 × 4 = 12 total combinations.

  Shirt: [Red, Blue, Green]  → 3 choices
  Pants: [Black, White, Grey, Brown] → 4 choices

  Red+Black, Red+White, Red+Grey, Red+Brown   → 4
  Blue+Black, Blue+White, Blue+Grey, Blue+Brown → 4
  Green+Black, Green+White, Green+Grey, Green+Brown → 4
  Total = 12

CODING ANALOGY:
  Nested loops — each loop "AND"s its choices:

  for i in 0..n:         ← n choices
    for j in 0..n:       ← n choices (for EACH i)
      doWork()           ← runs n × n = n² times → O(n²)
```

### The Addition Rule — "OR"

**If event A can happen in `m` ways OR event B can happen in `n` ways (but not both),**
**then either can happen in `m + n` ways.**

```
Example: You pick ONE thing — either a fruit (5 options) or a vegetable (8 options).
  Total choices = 5 + 8 = 13

CODING ANALOGY:
  Separate sequential code blocks — each block "OR"s its steps:

  for i in 0..n:        ← runs n times
    doWork()
                        ← "OR" (one or the other runs, sequentially)
  for j in 0..m:        ← runs m times
    doWork()

  Total = n + m → O(n + m) → usually simplified to O(n) if n dominates
```

### Rule of Dominant Terms in Big-O

```
When adding complexities, only the FASTEST GROWING term matters:

O(n³ + n² + n + 1)  →  O(n³)   (n³ grows fastest)
O(2^n + n!)          →  O(n!)   (n! grows faster than 2^n)
O(n log n + n)       →  O(n log n)

Why? As n → ∞, smaller terms become negligible:
  n=1000: n³=10⁹, n²=10⁶, n=1000 → n³ dominates by 1000×
```

---

## 2. Factorial — The Building Block

### What is Factorial?

```
n! = n × (n-1) × (n-2) × ... × 2 × 1

0! = 1    (by definition)
1! = 1
2! = 2 × 1 = 2
3! = 3 × 2 × 1 = 6
4! = 4 × 3 × 2 × 1 = 24
5! = 5 × 4 × 3 × 2 × 1 = 120
10! = 3,628,800
20! = 2,432,902,008,176,640,000  (2.4 quintillion!)

INTUITION:
  n! = "how many ways can I arrange n distinct objects?"
  
  3 books [A, B, C] on a shelf:
    Position 1: 3 choices (A or B or C)
    Position 2: 2 choices (whatever's left)
    Position 3: 1 choice  (only one left)
    Total = 3 × 2 × 1 = 6 arrangements

  [A,B,C]  [A,C,B]  [B,A,C]  [B,C,A]  [C,A,B]  [C,B,A]
```

### Factorial Grows INCREDIBLY Fast

```
n       n!              Approximate
─────────────────────────────────────────────
1       1               1
2       2               2
3       6               6
4       24              24
5       120             120
6       720             720
7       5,040           5 thousand
8       40,320          40 thousand
9       362,880         360 thousand
10      3,628,800       3.6 million
12      479,001,600     480 million
15      1.3 trillion    1.3 × 10¹²
20      2.4 quintillion 2.4 × 10¹⁸

THIS IS WHY O(n!) algorithms are only feasible for n ≤ 10-12.
```

---

## 3. Permutations — Order MATTERS

### Definition

**A permutation is an arrangement of items where the ORDER matters.**

```
"Permute" = rearrange.
If you pick 3 people from 5 for 1st, 2nd, 3rd place — ORDER MATTERS.
(Alice 1st, Bob 2nd) ≠ (Bob 1st, Alice 2nd)  → different permutations.
```

### Formula

```
P(n, r) = n! / (n - r)!

Where:
  n = total number of items
  r = how many you're choosing/arranging

P(n, n) = n! / 0! = n!   ← arranging ALL n items
P(n, 1) = n              ← choosing 1 from n
P(n, 0) = 1              ← choosing 0 from n (empty arrangement)
```

### Why This Formula Works

```
Example: 5 runners, pick 1st/2nd/3rd place medal winners. P(5, 3) = ?

Position 1 (Gold):   5 choices (any runner)
Position 2 (Silver): 4 choices (the 4 remaining runners)
Position 3 (Bronze): 3 choices (the 3 remaining runners)

By Multiplication Rule: 5 × 4 × 3 = 60

Using formula: P(5, 3) = 5! / (5-3)! = 5!/2! = 120/2 = 60 ✅

THINK OF IT AS:
  5 × 4 × 3 × 2 × 1     ← that's 5!
  ─────────────────── 
       2 × 1             ← that's (n-r)! = 2! — cancels the tail
  
  = 5 × 4 × 3 = 60      ← only the "front r terms" survive
```

### Examples

```
P(4, 2) — 4 letters {A,B,C,D}, pick 2, order matters:
  P(4,2) = 4!/(4-2)! = 4!/2! = 24/2 = 12
  Arrangements: AB AC AD BA BC BD CA CB CD DA DB DC → 12 ✅

P(3, 3) — arrange all 3: P(3,3) = 3!/0! = 6/1 = 6
  [ABC] [ACB] [BAC] [BCA] [CAB] [CBA] → 6 ✅

P(10, 3) — 10 items, pick 3 in order:
  P(10,3) = 10!/7! = 10 × 9 × 8 = 720

CODING CONNECTION:
  Generating all permutations of n elements → O(n!) time
  Because there are n! permutations, and each takes O(n) to output.
  Total: O(n × n!) which is often written O(n!)
```

### Code: Generating All Permutations (C++)

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

// Generate all permutations of array
// Time Complexity: O(n × n!)
//   - n! permutations exist
//   - Each permutation takes O(n) to construct/print
// Space Complexity: O(n) — call stack depth

void permute(vector<int>& arr, int start) {
    if (start == arr.size()) {
        // Print this permutation (O(n) work)
        for (int x : arr) cout << x << " ";
        cout << endl;
        return;
    }

    for (int i = start; i < arr.size(); i++) {
        swap(arr[start], arr[i]);          // fix element at 'start' position
        permute(arr, start + 1);           // arrange the rest
        swap(arr[start], arr[i]);          // backtrack (restore)
    }
}

int main() {
    vector<int> v = {1, 2, 3};
    cout << "All permutations of [1,2,3]:" << endl;
    permute(v, 0);
    // Output: 6 permutations (3! = 6)
    return 0;
}

/*
Recursion tree for permute([1,2,3], 0):

                  [1,2,3]
                  /  |  \
               fix1 fix2 fix3
              [1,..][2,..][3,..]
             /   \  ...   ...
          [1,2,3][1,3,2]  ...
            ↑      ↑
          print  print
          
Total leaves = 3! = 6 (each leaf = one permutation)
*/
```

---

## 4. Combinations — Order DOESN'T Matter

### Definition

**A combination is a selection of items where ORDER does NOT matter.**

```
"Combine" = group together without caring about order.
Picking 3 people for a team from 5 — ORDER DOESN'T MATTER.
{Alice, Bob, Carol} = {Bob, Alice, Carol}  → same team, same combination.
```

### Formula

```
C(n, r) = n! / (r! × (n-r)!)

Also written as:  ⁿCᵣ  or  C(n,r)  or  (n choose r)  or  nCr

Where:
  n = total number of items
  r = how many you're choosing

C(n, 0) = 1    ← choose 0 from n = 1 way (choose nothing)
C(n, 1) = n    ← choose 1 from n = n ways
C(n, n) = 1    ← choose all n from n = 1 way (take everything)
```

### Why Divide by r! Compared to Permutation?

```
Permutation counts [A,B,C] and [B,A,C] as DIFFERENT (order matters).
Combination counts [A,B,C] and [B,A,C] as THE SAME (order doesn't matter).

How many ways to ORDER r items? → r! ways

So: Combination = Permutation / r!
    C(n,r) = P(n,r) / r! = [n!/(n-r)!] / r! = n! / (r! × (n-r)!)

Example: C(5,3) — choose 3 from {A,B,C,D,E}

Permutations of 3 from 5 = P(5,3) = 60
But each group of 3 (like {A,B,C}) can be arranged 3! = 6 ways:
  [A,B,C] [A,C,B] [B,A,C] [B,C,A] [C,A,B] [C,B,A] ← all same combination!

So: C(5,3) = P(5,3) / 3! = 60 / 6 = 10 ✅
Formula:    C(5,3) = 5! / (3! × 2!) = 120 / (6 × 2) = 120/12 = 10 ✅
```

### Examples

```
C(4, 2) — pick 2 from {A,B,C,D}:
  C(4,2) = 4!/(2!×2!) = 24/(2×2) = 6
  Groups: {AB} {AC} {AD} {BC} {BD} {CD} → 6 ✅

C(5, 0) = 5!/(0!×5!) = 120/(1×120) = 1    ← empty set, 1 way
C(5, 5) = 5!/(5!×0!) = 120/(120×1) = 1    ← take all, 1 way
C(10, 3) = 10!/(3!×7!) = (10×9×8)/(3×2×1) = 720/6 = 120
C(52, 5) = 2,598,960  ← 5-card poker hands from 52-card deck!

SHORTCUT MENTAL MATH:
  C(n, r) = (n × (n-1) × ... × (n-r+1)) / r!
  
  C(10,3) = (10 × 9 × 8) / (3 × 2 × 1) = 720/6 = 120
             ←── r=3 terms ──→   ← divide by 3! →

CODING CONNECTION:
  Generating all subsets of r elements from n → O(C(n,r)) subsets
  Generating ALL subsets (any size) → 2^n subsets total
  Because: C(n,0) + C(n,1) + ... + C(n,n) = 2^n
```

### Key Identities You MUST Know

```
1. Symmetry:      C(n,r) = C(n, n-r)
   Example:       C(10,3) = C(10,7) = 120
   REASON: Choosing 3 to INCLUDE = choosing 7 to EXCLUDE.

2. Pascal's Rule: C(n,r) = C(n-1, r-1) + C(n-1, r)
   This gives us Pascal's Triangle! (see next section)

3. Sum of all subsets: C(n,0) + C(n,1) + ... + C(n,n) = 2^n
   REASON: Each of n items has 2 choices: IN or OUT of the subset.
   → 2 × 2 × ... × 2 (n times) = 2^n total subsets.

4. C(n, 2) = n(n-1)/2  ← number of pairs from n items
   This is where O(n²) comes from in pair-counting problems!
```

---

## 5. Pascal's Triangle — Combinations Visualized

```
        n=0:           C(0,0)                     = 1
        n=1:         C(1,0) C(1,1)                = 1  1
        n=2:       C(2,0)C(2,1)C(2,2)             = 1  2  1
        n=3:     C(3,0)C(3,1)C(3,2)C(3,3)         = 1  3  3  1
        n=4:   C(4,0)C(4,1)C(4,2)C(4,3)C(4,4)     = 1  4  6  4  1
        n=5: C(5,0)...                             = 1  5 10 10  5  1

PATTERN: Each entry = sum of two entries directly above it.
  C(4,2) = C(3,1) + C(3,2) = 3 + 3 = 6 ✅  (Pascal's Rule!)

ROW SUMS = powers of 2:
  n=0: 1          = 2⁰ = 1
  n=1: 1+1        = 2¹ = 2
  n=2: 1+2+1      = 2² = 4
  n=3: 1+3+3+1    = 2³ = 8
  n=4: 1+4+6+4+1  = 2⁴ = 16
  
  This proves: sum of all subsets of n items = 2^n

WHERE YOU SAW THIS IN CODE:
  - Grid Traveler dp table diagonals = values from Pascal's Triangle!
  - gridTraveler(m,n) = C(m+n-2, m-1)
  - Binomial expansion (a+b)^n — coefficients are Pascal's Triangle row n
```

---

## 6. How Permutations & Combinations Connect to Time Complexity

```
COUNTING OUTPUTS = LOWER BOUND ON TIME COMPLEXITY

If your algorithm must GENERATE or EXAMINE every possible arrangement,
the minimum time is proportional to how many those arrangements there are.

ARRANGEMENTS → TIME COMPLEXITY:

All arrangements of n items (permutations):
  Count = n!
  → O(n!) time minimum
  Example: string permutations, TSP brute force

All subsets of n items (combinations of all sizes):
  Count = 2^n  (each item is IN or OUT — 2 choices)
  → O(2^n) time minimum
  Example: all subsequences, power set, 0/1 knapsack brute force

All pairs from n items:
  Count = C(n,2) = n(n-1)/2 ≈ n²/2
  → O(n²) time
  Example: bubble sort, naive two-sum, naive duplicate detection

All triples from n items:
  Count = C(n,3) = n(n-1)(n-2)/6 ≈ n³/6
  → O(n³) time
  Example: naive matrix multiplication, 3-sum brute force

Arrangements of r items from n (partial permutations):
  Count = P(n,r) = n!/(n-r)!
  → O(n^r) if r is fixed, O(n!) if r=n
```

---

## 7. The "How Many Choices × How Many Steps" Framework

### The Core Mental Model

```
TIME COMPLEXITY = (choices at each step) ^ (number of steps)

OR for non-uniform steps:

TIME COMPLEXITY = step1_choices × step2_choices × step3_choices × ...
```

### Building Intuition — The Decision Tree View

```
Every algorithm makes a sequence of DECISIONS.
Draw the decision tree. Count the nodes (or leaves). That's your TC.

For a LOOP:
  for i in 0..n:         ← n choices at "step" level
    doWork()             ← O(1) work per choice
  → n × O(1) = O(n)

For NESTED LOOPS:
  for i in 0..n:         ← n choices
    for j in 0..n:       ← n choices FOR EACH outer choice
      doWork()           ← n × n × O(1) = O(n²)

For RECURSION with 2 branches, depth n:
  f(n):
    f(n-1)    ← branch 1
    f(n-1)    ← branch 2
  Tree depth = n, branching factor = 2
  Total nodes = 2^n → O(2^n)

For RECURSION with n branches, depth m:
  f(m, array[n]):
    for each of n items:
      f(m-1, array[n])
  Tree depth = m, branching factor = n
  Total nodes = n^m → O(n^m)
  ← This is exactly canSum's recursion!
```

### The Decision Tree for Common Problems

```
PROBLEM                DECISION TREE               COMPLEXITY
──────────────────────────────────────────────────────────────

Linear search:
  [item1][item2]...[itemN]  → check each   O(n)
  1 decision per item, n items total

Binary search:
       [mid]
      /     \
  [left]   [right]          O(log₂ n)
  Each step cuts n in half
  How many times can we halve n until n=1? → log₂(n) times

Bubble sort:
  For i: For j:
    compare pair (i,j)       O(n²)
  n choices × n choices = n² pairs

Subsets (power set):
  Each item: [IN] or [OUT]   O(2^n)
  n binary decisions → 2^n leaves

Permutations:
  Pick 1st: n choices
  Pick 2nd: n-1 choices
  Pick 3rd: n-2 choices       O(n!)
  ...
  Total = n × (n-1) × ... × 1 = n!

String matching (naive):
  n positions × m-length match check  O(n×m)

Fibonacci (naive):
  f(n) → f(n-1) + f(n-2)             O(2^n)
  2 branches, depth n
```

---

## 8. Big-O Cheat Sheet — All Common Complexities

### Growth Rate Comparison

```
For n = 10, 100, 1000:

Complexity  n=10        n=100              n=1000
──────────────────────────────────────────────────────────
O(1)        1           1                  1
O(log n)    3           7                  10
O(√n)       3           10                 32
O(n)        10          100                1,000
O(n log n)  33          664                9,966
O(n²)       100         10,000             1,000,000
O(n³)       1,000       1,000,000          10^9  ← 1 billion
O(2^n)      1,024       10^30  ← HUGE      10^301 ← impossible
O(n!)       3,628,800   10^157 ← INSANE    ...

FEASIBILITY (assuming 10^8 operations/second):
  O(n)        n=10^8    → 1 second    ← fine
  O(n log n)  n=10^6    → ~0.02 sec   ← fine
  O(n²)       n=10^4    → ~1 second   ← borderline
  O(n³)       n=10^2    → ~1 second   ← small inputs only
  O(2^n)      n=25      → ~0.3 sec    ← only tiny n
  O(n!)       n=12      → ~0.5 sec    ← only very tiny n
```

### Complexity Ranking (Slowest to Fastest)

```
SLOWEST (worst)
     O(n!)          — factorial: all permutations
     O(n^n)         — n choices at each of n steps (rare)
     O(2^n)         — exponential: all subsets
     O(n³)          — cubic: triple nested loops
     O(n²)          — quadratic: double nested loops
     O(n^1.5)       — sometimes seen in graph algorithms
     O(n log n)     — linearithmic: merge sort, heap sort
     O(n)           — linear: single pass
     O(√n)          — square root: some prime/factor algorithms
     O(log n)       — logarithmic: binary search
     O(1)           — constant: hash lookup, array index
FASTEST (best)
```

---

## 9. How to Calculate TC While Coding — Step by Step

### Method 1: Count the Loops

```
RULE: Count how many times the innermost operation runs.

SINGLE LOOP:
  for (int i = 0; i < n; i++)      ← runs n times
    doWork();                       → O(n)

LOOP NOT STARTING FROM 0:
  for (int i = 5; i < n; i++)      ← runs n-5 times ≈ n times
    doWork();                       → O(n)  (constants don't matter)

LOOP WITH STEP:
  for (int i = 0; i < n; i += 2)   ← runs n/2 times
    doWork();                       → O(n/2) = O(n)  (drop constant)

LOOP HALVING EACH TIME:
  for (int i = n; i > 0; i /= 2)   ← runs log₂(n) times
    doWork();                       → O(log n)  ← KEY PATTERN!

WHY log? Ask: "How many times can I divide n by 2 before reaching 1?"
  n → n/2 → n/4 → n/8 → ... → 1
  Answer: log₂(n) steps.
```

### Method 2: Count Nested Loops — Multiply

```
NESTED SAME-RANGE LOOPS:
  for (int i = 0; i < n; i++)         ← n times
    for (int j = 0; j < n; j++)       ← n times (for each i)
      doWork();                        → n × n = O(n²)

NESTED DIFFERENT-RANGE LOOPS:
  for (int i = 0; i < m; i++)         ← m times
    for (int j = 0; j < n; j++)       ← n times
      doWork();                        → m × n = O(m×n)

NESTED WITH DEPENDENCY (j starts from i):
  for (int i = 0; i < n; i++)
    for (int j = i; j < n; j++)       ← runs n-i times
      doWork();

  Total iterations = (n) + (n-1) + (n-2) + ... + 1
                   = n(n+1)/2  ≈  n²/2
                   → O(n²)  (drop constant 1/2)

  WHY? This is the formula for sum of 1 to n: Σk = n(n+1)/2
  This counts all pairs (i,j) where j ≥ i = C(n,2) + n ≈ n²/2

TRIPLE NESTED LOOPS:
  for i in 0..n:
    for j in 0..n:
      for k in 0..n:
        doWork();       → O(n³)
```

### Method 3: Recursion — The Recurrence Relation

```
STEP 1: Write the recurrence (how many calls, with what input size).
STEP 2: Solve it (using tree method or master theorem).

FORM: T(n) = a × T(n/b) + f(n)
  a = number of recursive calls
  n/b = size of each subproblem (how much input shrinks)
  f(n) = work done outside the recursive calls (combining/splitting)

EXAMPLES:

T(n) = T(n-1) + O(1)     ← one call, shrinks by 1, O(1) work
  Tree: n → n-1 → n-2 → ... → 0  (depth n, O(1) each)
  Total = O(n)
  [factorial recursion, linear search]

T(n) = T(n/2) + O(1)     ← one call, halves, O(1) work
  Tree: n → n/2 → n/4 → ... → 1  (depth log n, O(1) each)
  Total = O(log n)
  [binary search]

T(n) = 2T(n/2) + O(n)    ← two calls, halves, O(n) merge work
  Tree: depth log n, level k has 2^k nodes each of size n/2^k
        work at each level = 2^k × n/2^k = n (constant per level)
        log n levels × n work = O(n log n)
  [merge sort, quicksort average]

T(n) = 2T(n-1) + O(1)    ← two calls, shrinks by 1, O(1) work
  Tree: depth n, 2 branches → 2^n nodes
  Total = O(2^n)
  [naive Fibonacci, all subsets]

T(n) = n × T(n-1) + O(1) ← n calls, shrinks by 1, O(1) work
  Level 0: 1 node   (n calls)
  Level 1: n nodes  (each makes n-1 calls)
  Level 2: n×(n-1) nodes
  ...
  Total = n × (n-1) × (n-2) × ... = n!  → O(n!)
  [generating all permutations]
```

### Method 4: The Master Theorem (Quick Reference)

```
For recurrences of form: T(n) = a·T(n/b) + O(n^c)

Compare log_b(a) with c:

CASE 1: log_b(a) > c   →  T(n) = O(n^(log_b(a)))
CASE 2: log_b(a) = c   →  T(n) = O(n^c × log n)
CASE 3: log_b(a) < c   →  T(n) = O(n^c)

EXAMPLES:
  Merge Sort: T(n) = 2T(n/2) + O(n)
    a=2, b=2, c=1
    log_2(2) = 1 = c  →  CASE 2  →  O(n¹ × log n) = O(n log n) ✅

  Binary Search: T(n) = T(n/2) + O(1)
    a=1, b=2, c=0
    log_2(1) = 0 = c  →  CASE 2  →  O(n⁰ × log n) = O(log n) ✅

  Simple recursion: T(n) = 4T(n/2) + O(n)
    a=4, b=2, c=1
    log_2(4) = 2 > c=1  →  CASE 1  →  O(n²)
```

---

## 10. Recursion Tree Method — The Most Powerful Tool

### How to Draw a Recursion Tree

```
STEP 1: Write the recursive call at the root.
STEP 2: Draw branches for each recursive call.
STEP 3: Label each node with the WORK done at that level (excluding recursive calls).
STEP 4: Count total work across ALL levels.

TOTAL TIME = SUM OF WORK AT EACH LEVEL × NUMBER OF LEVELS

OR: COUNT ALL NODES if each node does O(1) work.
```

### Example 1: Merge Sort — T(n) = 2T(n/2) + O(n)

```
f(n=8):  ←────────────────── work = n = 8 ──────────────────────────
            /                                        \
         f(4)                                       f(4)
     ←── work=4 ──→                             ←── work=4 ──→
       /        \                                   /        \
    f(2)        f(2)                            f(2)        f(2)
  ← w=2 →    ← w=2 →                         ← w=2 →    ← w=2 →
   /   \      /   \                            /   \      /   \
f(1)  f(1)  f(1) f(1)                       f(1) f(1)  f(1) f(1)

WORK PER LEVEL:
  Level 0 (root):  1 node  × n work  = n
  Level 1:         2 nodes × n/2     = n
  Level 2:         4 nodes × n/4     = n
  Level 3:         8 nodes × n/8     = n   (leaves, O(1) each)

TOTAL LEVELS = log₂(n)    [since we halve each time: n→n/2→n/4→...→1]
WORK PER LEVEL = n        [constant! always sums to n]

TOTAL = log₂(n) levels × n work per level = O(n log n) ✅

KEY INSIGHT: Work per level is CONSTANT (n) because
  doubling the nodes × halving the work = same total.
```

### Example 2: Fibonacci — T(n) = 2T(n-1) + O(1)

```
f(5):
         f(5)                                          level 0: 1 node
        /     \
      f(4)    f(4)                                     level 1: 2 nodes
     /   \   /   \
   f(3) f(3)f(3) f(3)                                 level 2: 4 nodes
   / \  ...                                            level 3: 8 nodes
 f(2)f(2)                                             ...

NODES PER LEVEL: 1, 2, 4, 8, ..., 2^(n-1)
DEPTH OF TREE: n (decreasing by 1 each level)
TOTAL NODES: 1+2+4+...+2^(n-1) = 2^n - 1 = O(2^n)
WORK PER NODE: O(1)

TOTAL: O(2^n) × O(1) = O(2^n) ✅

DUPLICATE WORK:
  f(3) appears 4 times at level 2!
  f(2) appears 8 times at level 3!
  Memoization caches each unique value → only n unique values → O(n).
```

### Example 3: canSum — T(m) = n × T(m-1) + O(1)

```
canSum(4, [1,2,3]):   n=3 numbers, m=4 targetSum

           cs(4)                         branching factor = 3
          / | \
       cs(3)cs(2)cs(1)                  level 1: 3 nodes
       /|\  /|\  /|\
    cs(2).. cs(1).. cs(0)               level 2: 9 nodes
    /|\                ↑
  ...                TRUE               level 3: 27 nodes
                                        level 4: 81 nodes

DEPTH: m = 4 (targetSum — we subtract at least 1 each time with [1,...])
BRANCHING FACTOR: n = 3 (one branch per number in array)
TOTAL NODES: n^m = 3^4 = 81 → O(n^m) ✅

WITH MEMO:
  Unique targetSum values: 0, 1, 2, 3, 4 → only 5 unique values.
  Each computed once, tries n=3 numbers.
  Total: n × m = 3 × 4 = 12 operations → O(n × m) ✅
```

---

## 11. Real Code Examples — TC Breakdown

### Example A: Two Sum

```cpp
// APPROACH 1: Brute Force — check all pairs
// Time: O(n²)  Space: O(1)
int twoSum_brute(vector<int>& nums, int target) {
    for (int i = 0; i < n; i++)               // ← n choices for i
        for (int j = i+1; j < n; j++)         // ← n-i choices for j
            if (nums[i] + nums[j] == target)   // ← O(1) check
                return {i, j};

    // Total pairs checked: C(n,2) = n(n-1)/2 ≈ n²/2 → O(n²)
}

// APPROACH 2: Hash Map — check complement in O(1)
// Time: O(n)   Space: O(n)
int twoSum_hash(vector<int>& nums, int target) {
    unordered_map<int, int> seen;
    for (int i = 0; i < n; i++) {             // ← n iterations
        int complement = target - nums[i];
        if (seen.count(complement))            // ← O(1) hash lookup
            return {seen[complement], i};
        seen[nums[i]] = i;                    // ← O(1) hash insert
    }
    // Total: n × O(1) = O(n)
}

// WHY THE DIFFERENCE?
// Brute force: "Try ALL pairs"  → C(n,2) pairs → O(n²)
// Hash map:    "Check one complement per element" → n elements → O(n)
```

### Example B: Binary Search

```cpp
// Time: O(log n)   Space: O(1)
int binarySearch(vector<int>& arr, int target) {
    int left = 0, right = arr.size() - 1;

    while (left <= right) {                    // ← how many times?
        int mid = left + (right - left) / 2;
        if (arr[mid] == target)  return mid;
        if (arr[mid] < target)   left = mid + 1;   // discard left half
        else                     right = mid - 1;  // discard right half
    }
    return -1;
}

// TC ANALYSIS:
// Each iteration: search space HALVES (n → n/2 → n/4 → ...)
// Ask: "How many times can I halve n before reaching 1?"
//
// n / 2^k = 1  →  2^k = n  →  k = log₂(n)
//
// Loop runs log₂(n) times.  Each iteration: O(1) work.
// Total: log₂(n) × O(1) = O(log n)
//
// n=1000:   log₂(1000) ≈ 10 iterations  (not 1000!)
// n=10^6:   log₂(10⁶)  = 20 iterations
// n=10^9:   log₂(10⁹)  = 30 iterations
// → Binary search is INCREDIBLY efficient.
```

### Example C: Merge Sort

```cpp
// Time: O(n log n)   Space: O(n)
void mergeSort(vector<int>& arr, int left, int right) {
    if (left >= right) return;

    int mid = left + (right - left) / 2;
    mergeSort(arr, left, mid);              // T(n/2) — sort left half
    mergeSort(arr, mid+1, right);           // T(n/2) — sort right half
    merge(arr, left, mid, right);           // O(n)   — merge halves
}

// RECURRENCE: T(n) = 2T(n/2) + O(n)
// See recursion tree above → O(n log n)

// INTUITION:
// log n levels of recursion (halving each time)
// n total work per level (merging)
// → n log n total
```

### Example D: All Subsets (Power Set)

```cpp
// Time: O(2^n × n)   Space: O(2^n × n) for storing all subsets
// or O(n) for the recursion stack alone

void subsets(vector<int>& nums, int i, vector<int>& current,
             vector<vector<int>>& result) {
    if (i == nums.size()) {
        result.push_back(current);         // ← O(n) to copy
        return;
    }

    // CHOICE 1: INCLUDE nums[i]
    current.push_back(nums[i]);
    subsets(nums, i+1, current, result);   // recurse with inclusion
    current.pop_back();

    // CHOICE 2: EXCLUDE nums[i]
    subsets(nums, i+1, current, result);   // recurse without inclusion
}

// TC ANALYSIS:
// At each of n elements: 2 choices (include or exclude)
// By multiplication rule: 2 × 2 × ... × 2 (n times) = 2^n
// Total subsets = 2^n
// Each subset: O(n) to copy into result
// Total: O(2^n × n)
//
// Recursion tree:
//   [include/exclude nums[0]]
//           /              \
//   [incl/excl nums[1]]  [incl/excl nums[1]]
//       /       \              /       \
//       ...                   ...
//   n levels, 2 branches → 2^n leaves
```

### Example E: String Permutations

```cpp
// Time: O(n! × n)  Space: O(n) stack + O(n! × n) for storing
void permutations(string s, int start, vector<string>& result) {
    if (start == s.size()) {
        result.push_back(s);               // ← O(n) to copy
        return;
    }

    for (int i = start; i < s.size(); i++) {
        swap(s[start], s[i]);              // fix character at position 'start'
        permutations(s, start+1, result); // arrange rest
        swap(s[start], s[i]);              // backtrack
    }
}

// TC ANALYSIS:
// Positions: n, n-1, n-2, ..., 1   (choices at each level)
// By multiplication rule: n × (n-1) × (n-2) × ... × 1 = n!
// Leaves of recursion tree = n! (one per permutation)
// Each leaf: O(n) work to copy string
// Total: O(n! × n)
```

### Example F: Finding Duplicates — Three Approaches

```cpp
// APPROACH 1: Brute Force — O(n²)
bool hasDuplicate_brute(vector<int>& arr) {
    for (int i = 0; i < n; i++)
        for (int j = i+1; j < n; j++)     // C(n,2) pairs = O(n²)
            if (arr[i] == arr[j]) return true;
    return false;
}

// APPROACH 2: Sort First — O(n log n)
bool hasDuplicate_sort(vector<int>& arr) {
    sort(arr.begin(), arr.end());          // O(n log n)
    for (int i = 0; i+1 < n; i++)         // O(n) — check adjacent
        if (arr[i] == arr[i+1]) return true;
    return false;
    // Total: O(n log n) + O(n) = O(n log n)
}

// APPROACH 3: Hash Set — O(n)
bool hasDuplicate_hash(vector<int>& arr) {
    unordered_set<int> seen;
    for (int x : arr) {                    // O(n) loop
        if (seen.count(x)) return true;   // O(1) check
        seen.insert(x);                   // O(1) insert
    }
    return false;
    // Total: O(n)
}
```

---

## 12. Space Complexity Patterns

```
Space = memory used by your algorithm (excluding input)

O(1)        CONSTANT — fixed variables, no matter input size
  int a = 0, b = 0;              ← O(1) always

O(log n)    LOGARITHMIC — recursion stack that halves each time
  Binary search (recursive):     ← log n call stack frames

O(n)        LINEAR — array/stack proportional to input
  Recursion depth n (linear):    ← n call stack frames
  Storing one thing per element: ← n items in map/set/array

O(n²)       QUADRATIC — 2D table proportional to n×n
  canSum tabulation (m×n table): ← m×n entries
  Grid traveler (m×n table):     ← m×n entries

O(2^n)      EXPONENTIAL — storing all subsets
  Storing all subsets of n items: ← 2^n items

KEY SPACE PATTERNS:
  Call stack depth = recursion depth (not total nodes!)
  Memoization = number of unique subproblems
  Tabulation = size of dp array/table
```

### Recursion Stack Space Explained

```
IMPORTANT: Stack space = DEPTH of recursion (not total nodes!)

At any moment, only ONE path from root to current node is on the stack.

      f(5)  ←── currently executing
     /
   f(4)     ←── waiting for f(3) to return
   /
 f(3)       ←── waiting for f(2) to return
 /
f(2)        ←── waiting for f(1) to return
/
f(1)        ←── base case, about to return

Stack at this moment: [f(5), f(4), f(3), f(2), f(1)] = 5 frames
Maximum stack depth = n = O(n) space

When f(1) returns, f(2) resumes and calls f(0) (its other branch).
The total tree has 2^n nodes, but only n are active simultaneously.

THEREFORE:
  Fibonacci (recursive): O(n) space, NOT O(2^n)
  canSum (recursive):    O(m) space, NOT O(n^m)  (m = targetSum)
  Permutations:          O(n) space for stack
```

---

## 13. Quick Reference — TC of Common Algorithms

```
ALGORITHM                  TIME COMPLEXITY   SPACE COMPLEXITY
─────────────────────────────────────────────────────────────

SEARCHING
  Linear Search            O(n)              O(1)
  Binary Search            O(log n)          O(1) iter / O(log n) rec
  Hash Table Lookup        O(1) avg          O(n)

SORTING
  Bubble Sort              O(n²)             O(1)
  Selection Sort           O(n²)             O(1)
  Insertion Sort           O(n²)             O(1)
  Merge Sort               O(n log n)        O(n)
  Quick Sort               O(n log n) avg    O(log n)
  Heap Sort                O(n log n)        O(1)
  Counting Sort            O(n + k)          O(k)

ARRAYS / BASIC
  Access                   O(1)
  Insert (end)             O(1) amortized
  Insert (middle)          O(n)
  Delete                   O(n)

STRINGS
  String comparison        O(n)
  All substrings           O(n²)
  All permutations         O(n! × n)
  KMP pattern match        O(n + m)

DP PROBLEMS
  Fibonacci (naive)        O(2^n)            O(n)
  Fibonacci (memo/tab)     O(n)              O(n)
  Grid Traveler (naive)    O(2^(m+n))        O(m+n)
  Grid Traveler (DP)       O(m×n)            O(m×n)
  canSum (naive)           O(n^m)            O(m)
  canSum (DP)              O(n×m)            O(m)
  0/1 Knapsack             O(n×W)            O(n×W)
  Longest Common Subseq    O(m×n)            O(m×n)
  Coin Change              O(n×amount)       O(amount)

GRAPHS
  BFS                      O(V + E)          O(V)
  DFS                      O(V + E)          O(V)
  Dijkstra                 O((V+E) log V)    O(V)
  Floyd-Warshall           O(V³)             O(V²)

TREES
  Binary Tree Traversal    O(n)              O(h)  [h=height]
  BST Insert/Search        O(h)  O(log n) balanced
  Building a Heap          O(n)              O(1)

COMBINATORIAL
  All subsets (power set)  O(2^n × n)        O(2^n × n)
  All permutations         O(n! × n)         O(n! × n)
  Backtracking (general)   O(b^d)            O(d)  [b=branch,d=depth]
```

---

## P&C Formulas at a Glance

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FACTORIAL:         n! = n × (n-1) × ... × 1       0! = 1

PERMUTATION:       P(n,r) = n! / (n-r)!
  (order MATTERS)  P(n,n) = n!

COMBINATION:       C(n,r) = n! / (r! × (n-r)!)
  (order doesn't)  C(n,r) = C(n, n-r)    [symmetry]
                   C(n,0) = C(n,n) = 1
                   C(n,2) = n(n-1)/2 ← counts all pairs

ALL SUBSETS:       2^n  [each element is IN or OUT]
                   = C(n,0) + C(n,1) + ... + C(n,n)

PASCAL'S RULE:     C(n,r) = C(n-1,r-1) + C(n-1,r)

ALL PAIRS:         C(n,2) = n(n-1)/2 ≈ n²/2   → O(n²)
ALL TRIPLES:       C(n,3) = n(n-1)(n-2)/6      → O(n³)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

QUICK MAPPING: Algorithm Structure → Time Complexity

  Single loop over n            →  O(n)
  Loop halving each step        →  O(log n)
  Two nested loops              →  O(n²)
  Three nested loops            →  O(n³)
  Recursion: 2 branches, n deep →  O(2^n)
  Recursion: n branches, n deep →  O(n^n)  [very rare]
  Recursion: n branches, m deep →  O(n^m)
  Generate all permutations     →  O(n!)
  Generate all subsets          →  O(2^n)
  Divide and conquer (halving)  →  O(n log n)
  Memoization (unique inputs)   →  O(unique_states × work_per_state)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

*Practice calculating TC before looking it up — the intuition builds with repetition.*
*The "how many choices at each step?" question will solve 90% of TC problems.*
