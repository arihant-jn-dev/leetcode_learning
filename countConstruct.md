# countConstruct Problem — Three Techniques
## Recursion vs Memoization vs Tabulation in C++

> **Problem:** Given a `target` string and an array of strings `wordBank`,
> return the **number of ways** the `target` can be constructed by concatenating
> words from `wordBank`.
> You may reuse words from `wordBank` as many times as you like.
> All characters are lowercase English letters.

---

## countConstruct vs canConstruct — One Word Changes Everything

```
canConstruct("purple", ["purp","p","ur","le","purpl"])   → true / false
  "CAN we build the target?"
  Returns: boolean — just YES or NO
  Short-circuits: returns true as soon as the FIRST valid path is found.
  Operator: OR  (any one path working is enough)

countConstruct("purple", ["purp","p","ur","le","purpl"]) → 2
  "HOW MANY WAYS can we build the target?"
  Returns: integer — the exact count of valid constructions
  NO short-circuit: MUST explore ALL paths to count them all.
  Operator: + (SUM up results from every branch)

THE ONE-LINE DIFFERENCE:
  canConstruct:    if(cc(suffix, wb))   return true;     ← OR logic
  countConstruct:  total += cc(suffix, wb);              ← ADD logic

ANALOGY WITH NUMBERS:
  canSum:    → true/false    "can we reach target?"
  howSum:    → vector        "give me any one combination"
  countSum:  → integer       "how many combinations exist?"
  ↕ same relationship ↕
  canConstruct:   → true/false   "can we build target?"
  countConstruct: → integer      "how many ways to build it?"
```

---

## Quick Comparison (Read This First)

```
TECHNIQUE       TIME            SPACE         STYLE
──────────────────────────────────────────────────────────────────────
Recursion       O(n^m × m)      O(m²)         Top-down
Memoization     O(n × m²)       O(m²)         Top-down + cache
Tabulation      O(n × m²)       O(m)          Bottom-up

Where: m = target.length,  n = wordBank.size()

Same complexities as canConstruct.
The only difference: storing int counts instead of booleans.
Tabulation space advantage (O(m) vs O(m²)) is preserved.
```

---

## The Three Key Substitutions

```
Going from canConstruct → countConstruct requires exactly 3 changes:

CHANGE 1: Return type
  canConstruct:   bool → true / false
  countConstruct: int  → 0, 1, 2, 3, ...

CHANGE 2: Base case
  canConstruct:   if(target.empty()) return true;
  countConstruct: if(target.empty()) return 1;
                                           ↑
                              "1 way" to build an empty string:
                              pick nothing! That's 1 valid way.

CHANGE 3: How branch results are combined
  canConstruct:   if(cc(suffix, wb)) return true;   ← OR: stop at first success
  countConstruct: total += cc(suffix, wb);           ← ADD: accumulate all paths
                  return total;

TABULATION CHANGE (bonus):
  canConstruct:   dp[i + wlen] = true;        ← SET once
  countConstruct: dp[i + wlen] += dp[i];      ← ACCUMULATE
```

---

## The Problem Visualized

```
countConstruct("purple", ["purp","p","ur","le","purpl"])

All valid constructions:
  PATH 1: "purp" + "le"          = "purpl" + "e"? no... "purp"(4) + "le"(2) = "purple" ✓
  PATH 2: "p" + "ur" + "p" + "le" = "p"(1)+"ur"(2)+"p"(1)+"le"(2) = "purple" ✓

What about "purpl" + "e"? "e" not in wordBank → dead end.
What about "p" + "ur" + "ple"? "ple" not in wordBank → dead end.

ANSWER: 2 valid ways ✓

────────────────────────────────────────────────────────────────────────

countConstruct("abcdef", ["ab","abc","cd","def","abcd"])

Paths:
  "ab" + "cd" + ???   → after "abcd", need "ef" — no word matches "ef..." ✗
  "abc" + "def"       = "abcdef" ✓
  "abcd" + ???        → need "ef" — dead end ✗
  "ab" + "cdef"?      → "cdef" not constructible ✗

ANSWER: 1 valid way

────────────────────────────────────────────────────────────────────────

countConstruct("enterapotentpot", ["a","p","ent","enter","ot","o","t"])

"enter"(5) → "apotentpot"
  "a"(1)   → "potentpot"
    "p"(1) → "otentpot"
      "ot"(2) → "entpot"
        "ent"(3) → "pot"
          "p"(1)  → "ot"
            "ot"(2) → "" = ✓ PATH 1
            "o"(1)  → "t"
              "t"(1) → "" = ✓ PATH 2
      "o"(1) → "tentpot"
        "t"(1) → "entpot"
          "ent"(3) → "pot"
            "p"(1)  → "ot"
              "ot"(2) → "" = ✓ PATH 3
              "o"(1)  → "t"
                "t"(1) → "" = ✓ PATH 4
"ent"(3) → "erapotentpot" → no word starts with 'r' here → 0 ways

TOTAL = 4 ways
```

**Test Cases for Reference:**

```
countConstruct("purple",         ["purp","p","ur","le","purpl"])         = 2
countConstruct("abcdef",         ["ab","abc","cd","def","abcd"])          = 1
countConstruct("skateboard",     ["bo","rd","ate","t","ska","sk","boar"]) = 0
countConstruct("enterapotentpot",["a","p","ent","enter","ot","o","t"])    = 4
countConstruct("",               ["cat","dog","horse"])                   = 1  (base!)
countConstruct("eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeef",
               ["e","ee","eee","eeee","eeeee","eeeeee"])                  = 0
```

---

## The Recursive Formula

```
countConstruct(target, wordBank):

  BASE CASE: target is empty string ("")
    → return 1   (found ONE complete valid way — the empty construction)

  totalCount = 0      ← accumulates count across ALL branches

  FOR EACH word in wordBank:
    IF target STARTS WITH word:            ← prefix check (same as canConstruct)
      suffix = target AFTER removing word  ← slice off matched prefix
      totalCount += countConstruct(suffix, wordBank)   ← ADD (not OR!)

  return totalCount   ← 0 if nothing worked; ≥1 if some paths found

CRITICAL DIFFERENCE FROM canConstruct:
  canConstruct:   if(cc(suffix, wb)) return true;    ← stop at first success
  countConstruct: total += cc(suffix, wb);            ← never stop, always add

WHY WE CAN'T STOP EARLY:
  Stopping at the first success would give count = 1 (at most).
  We need to count ALL successful paths → must explore every branch.
  This is identical to why bestSum can't short-circuit.
```

---

## Technique 1 — Recursion (Naive)

### The Idea

At each call, try every word as a prefix match.
For every match, recurse on the suffix and ADD the result.
Never return early — always complete the full loop.
Return the accumulated total count.

### How the Count Builds Through the Call Stack

```
countConstruct("purple", ["purp","p","ur","le","purpl"])

Step-by-step accumulation (top level):

  Try "purp": matches → cc("le")
    cc("le"):
      Try "purp": "le"<len("purp") → skip
      Try "p":    "l"≠"p" ✗
      Try "ur":   "le"≠"ur" ✗
      Try "le":   "le"="le" ✓ → cc("") = 1  total+=1
      Try "purpl":"le"<len("purpl") → skip
      return 1
    total += 1   ← "purp"+"le" is ONE valid path

  Try "p": matches → cc("urple")
    cc("urple"):
      Try "purp": "urpl"≠"purp" ✗
      Try "p":    "u"≠"p" ✗
      Try "ur":   "ur"="ur" ✓ → cc("ple")
        cc("ple"):
          Try "purp": too long ✗
          Try "p":    "p"="p" ✓ → cc("le") = 1  total+=1
          Try "ur":   "pl"≠"ur" ✗
          Try "le":   "pl"≠"le" ✗
          Try "purpl":too long ✗
          return 1
        total += 1
      Try "le":   "ur"≠"le" ✗
      Try "purpl":"urple"≠"purpl" ✗
      return 1
    total += 1   ← "p"+"ur"+"p"+"le" is another valid path

  Try "ur":    "pu"≠"ur" ✗
  Try "le":    "pu"≠"le" ✗
  Try "purpl": matches → cc("e")
    cc("e"):
      No word matches 'e' prefix → return 0
    total += 0   ← "purpl"+"e"... "e" can't be built → dead end

  FINAL total = 1 + 1 + 0 = 2 ✓
```

### Recursion Tree for countConstruct("purple", ["purp","p","ur","le","purpl"])

```
                     cc("purple") = ?
                   /         |          \
            cc("le")=1  cc("urple")=1  cc("e")=0
           [purp matched] [p matched] [purpl matched]
               ↓               ↓
           cc("")=1  ←    cc("ple")=1
          [le matched]   [ur matched]
           BASE CASE!         ↓
                          cc("le")=1
                         [p matched]
                              ↓
                          cc("")=1
                         [le matched]
                          BASE CASE!

EACH LEAF "cc("")" contributes 1 to the total count.
EACH INTERNAL NODE accumulates by adding child results.

RESULTS BUBBLE UP BY ADDITION:
  cc("le")    = 1  (from cc("") via "le")
  cc("ple")   = 1  (from cc("le") via "p")
  cc("urple") = 1  (from cc("ple") via "ur")
  cc("e")     = 0  (nothing matches)
  cc("purple")= 1+1+0 = 2

REPEATED SUBPROBLEM:
  cc("le") is computed TWICE:
    Once from cc("purple") via "purp" → cc("le")
    Once from cc("ple")   via "p"    → cc("le")

  cc("le") = 1 in BOTH cases.
  → Memoization will cache and reuse this!
```

### Recursion Tree for countConstruct("abcdef", ["ab","abc","cd","def","abcd"])

```
                       cc("abcdef") = ?
             ┌──────────┼──────────┬──────────────┐
         cc("cdef")  cc("def")  cc("ef")       (cd,def don't match at 0)
         [ab mtch]   [abc mtch] [abcd mtch]
             │            │         │
        cc("ef")=0    cc("")=1  cc("ef")=0
       [cd matched]  [def mtch]
          BASE!

Wait: from cc("abcdef"):
  "ab"   matches → cc("cdef"):
    "cd" matches → cc("ef"):
      nothing matches 'e' prefix → 0
    others don't match → return 0
  "abc"  matches → cc("def"):
    "def" matches → cc("") = 1 → return 1
  "abcd" matches → cc("ef"):
    nothing matches → 0

cc("abcdef") = 0 + 1 + 0 = 1 ✓

NOTICE: cc("ef") computed TWICE (from "ab"→"cd" and from "abcd").
        Both times = 0. Memoization caches it once.
```

### C++ Code

```cpp
#include <iostream>
#include <vector>
#include <string>
using namespace std;

// ─────────────────────────────────────────────────────────────
// TECHNIQUE 1: RECURSION (Naive)
// Time:  O(n^m × m)
//   n^m recursive calls (branching n, depth m).
//   Each call: O(m) for substr() to create suffix + comparison.
//   No short-circuit → full tree always explored.
// Space: O(m²)
//   Call stack depth: m frames.
//   Each frame holds a suffix string of up to O(m) chars.
//   m frames × O(m) string = O(m²) total.
// ─────────────────────────────────────────────────────────────

int countConstruct_recursive(const string& target, const vector<string>& wordBank) {

    // BASE CASE: empty string — exactly 1 way to build it (pick nothing)
    if (target.empty()) return 1;

    int totalCount = 0;    // accumulate count from ALL branches

    for (const string& word : wordBank) {
        // Check if word matches the front of target
        if (target.length() >= word.length() &&
            target.substr(0, word.length()) == word) {

            string suffix = target.substr(word.length());

            // ADD the number of ways to build the suffix
            totalCount += countConstruct_recursive(suffix, wordBank);
            // NOTE: no "return" here — we must try ALL words!
        }
    }

    return totalCount;    // 0 if no word works; ≥1 if paths exist
}

int main() {
    cout << "countConstruct (Recursion)" << endl;
    cout << "──────────────────────────────────────────────────────" << endl;

    cout << "cc(\"purple\",         [purp,p,ur,le,purpl]) = "
         << countConstruct_recursive("purple", {"purp","p","ur","le","purpl"}) << endl;

    cout << "cc(\"abcdef\",         [ab,abc,cd,def,abcd]) = "
         << countConstruct_recursive("abcdef", {"ab","abc","cd","def","abcd"}) << endl;

    cout << "cc(\"skateboard\",     [bo,rd,ate,t,ska,sk,boar]) = "
         << countConstruct_recursive("skateboard", {"bo","rd","ate","t","ska","sk","boar"}) << endl;

    cout << "cc(\"\",               [cat,dog]) = "
         << countConstruct_recursive("", {"cat","dog"}) << endl;

    cout << "cc(\"abcdef\",         [ab,cd,abcg]) = "
         << countConstruct_recursive("abcdef", {"ab","cd","abcg"}) << endl;

    // WARNING: Do NOT run the "eeeeef" test without memoization!
    // Exponential branches all ending in 'f' (unmatchable) → hangs!

    return 0;
}
```

**Output:**
```
countConstruct (Recursion)
──────────────────────────────────────────────────────
cc("purple",         [purp,p,ur,le,purpl]) = 2
cc("abcdef",         [ab,abc,cd,def,abcd]) = 1
cc("skateboard",     [bo,rd,ate,t,ska,sk,boar]) = 0
cc("",               [cat,dog]) = 1
cc("abcdef",         [ab,cd,abcg]) = 0
```

### Complexity Analysis — Recursion

```
TIME COMPLEXITY: O(n^m × m)
────────────────────────────
Same reasoning as canConstruct:
  Branching factor = n (try each word), depth = m (1 char min consumed per level).
  Total nodes: n^m
  Per node: substr() + comparison → O(m)
  TOTAL: O(n^m × m)

SAME AS canConstruct? Yes — asymptotically.
But countConstruct is ALWAYS SLOWER IN PRACTICE:
  canConstruct: short-circuits at first success → many branches never explored
  countConstruct: never short-circuits → entire n^m tree always explored

SPACE COMPLEXITY: O(m²)
────────────────────────
Same reasoning as canConstruct:
  m stack frames × suffix string of O(m) chars per frame = O(m²).
  (Each frame stores a local int totalCount → O(1), but also the suffix string)
```

---

## Technique 2 — Memoization (Top-Down DP)

### The Idea

**Cache the COUNT for each unique suffix string, so it is computed only once.**

Key insight: `countConstruct("le", wordBank)` returns the same count
regardless of how we arrived at the suffix `"le"`.
So once computed, store it — add from cache instead of recomputing the whole subtree.

```cpp
unordered_map<string, int> memo;
// Key   = remaining suffix string
// Value = number of ways to construct this suffix from wordBank
```

### How the Count is Cached and Reused

```
WITHOUT MEMO — cc("le") computed TWICE:

  cc("purple"):
    "purp" → cc("le") = 1    ← computed here (full subtree)
    "p"    → cc("urple")
               "ur" → cc("ple")
                        "p" → cc("le") = 1  ← computed AGAIN!

WITH MEMO:
  cc("le") computed FIRST from "purp" branch:
    "le" matches "le" → cc("") = 1
    return 1
    memo["le"] = 1 ✅ STORED

  cc("le") needed AGAIN from "p"→"ur"→"p" path:
    🎯 CACHE HIT: memo["le"] = 1 → return instantly!
    Entire subtree of cc("le") SKIPPED.

MEMO CONTENTS after cc("purple"):
  memo[""]      = 1   ← base case (actually not stored, returned directly)
  memo["le"]    = 1   ← "le" → 1 way
  memo["ple"]   = 1   ← "p"+"le" → 1 way
  memo["urple"] = 1   ← "ur"+"p"+"le" → 1 way
  memo["e"]     = 0   ← "e" can't be built from wordBank
  memo["purple"]= 2   ← 2 ways total
```

### C++ Code

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <unordered_map>
using namespace std;

// ─────────────────────────────────────────────────────────────
// TECHNIQUE 2: MEMOIZATION (Top-Down DP)
// Time:  O(n × m²)
//   (m+1) unique suffixes × n words × O(m) per substr/compare
// Space: O(m²)
//   memo: up to m+1 string keys of up to m chars each → O(m²)
//   call stack: m frames × O(m) suffix strings → O(m²)
//   Total: O(m²)
// ─────────────────────────────────────────────────────────────

int countConstruct_memo(const string& target,
                        const vector<string>& wordBank,
                        unordered_map<string, int>& memo) {

    // STEP 1: Cache check — return stored count if already computed
    if (memo.count(target)) {
        return memo[target];    // CACHE HIT — instant return
    }

    // STEP 2: Base case — one way to construct an empty string
    if (target.empty()) return 1;

    int totalCount = 0;

    // STEP 3: Try each word — NO early return, explore ALL branches
    for (const string& word : wordBank) {
        if (target.length() >= word.length() &&
            target.substr(0, word.length()) == word) {

            string suffix = target.substr(word.length());
            totalCount += countConstruct_memo(suffix, wordBank, memo);
        }
    }

    // STEP 4: Cache the total count for this suffix and return
    return memo[target] = totalCount;
}

int main() {
    cout << "countConstruct (Memoization)" << endl;
    cout << "──────────────────────────────────────────────────────" << endl;

    auto run = [](const string& t, vector<string> wb) {
        unordered_map<string, int> memo;
        return countConstruct_memo(t, wb, memo);
    };

    cout << "cc(\"purple\",          [purp,p,ur,le,purpl]) = "
         << run("purple", {"purp","p","ur","le","purpl"}) << endl;

    cout << "cc(\"abcdef\",          [ab,abc,cd,def,abcd]) = "
         << run("abcdef", {"ab","abc","cd","def","abcd"}) << endl;

    cout << "cc(\"skateboard\",      [bo,rd,ate,t,ska,sk,boar]) = "
         << run("skateboard", {"bo","rd","ate","t","ska","sk","boar"}) << endl;

    cout << "cc(\"enterapotentpot\", [a,p,ent,enter,ot,o,t]) = "
         << run("enterapotentpot", {"a","p","ent","enter","ot","o","t"}) << endl;

    cout << "cc(\"\",                [cat,dog]) = "
         << run("", {"cat","dog"}) << endl;

    cout << "cc(\"eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeef\", [e,ee,eee,eeee,eeeee,eeeeee]) = "
         << run("eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeef",
                {"e","ee","eee","eeee","eeeee","eeeeee"}) << endl;
    // ↑ Instant! Without memo: would hang (counts exponential 'e' partitions)

    return 0;
}
```

**Output:**
```
countConstruct (Memoization)
──────────────────────────────────────────────────────
cc("purple",          [purp,p,ur,le,purpl]) = 2
cc("abcdef",          [ab,abc,cd,def,abcd]) = 1
cc("skateboard",      [bo,rd,ate,t,ska,sk,boar]) = 0
cc("enterapotentpot", [a,p,ent,enter,ot,o,t]) = 4
cc("",                [cat,dog]) = 1
cc("eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeef",[e,ee,...]) = 0
```

### Step-by-Step Trace for countConstruct("purple", ["purp","p","ur","le","purpl"])

```
cc = countConstruct, 🎯 = cache hit, ✅ = stored in memo

cc("purple"): not in memo
  try "purp": "purple" starts with "purp" ✓
    suffix = "le"
    cc("le"): not in memo
      try "purp": len("le")=2 < len("purp")=4 → skip
      try "p":    "l"≠"p" ✗
      try "ur":   "le"≠"ur" ✗
      try "le":   "le"="le" ✓
        suffix = ""
        cc("") → return 1  (BASE CASE — not stored in memo)
        total += 1
      try "purpl": len("le")=2 < 5 → skip
      return 1 → memo["le"] = 1 ✅
    totalCount += 1   ← "purp"+"le" path
  try "p": "purple" starts with "p" ✓
    suffix = "urple"
    cc("urple"): not in memo
      try "purp": "urpl"≠"purp" ✗
      try "p":    "u"≠"p" ✗
      try "ur":   "ur"="ur" ✓
        suffix = "ple"
        cc("ple"): not in memo
          try "purp": too short ✗
          try "p":    "p"="p" ✓
            suffix = "le"
            cc("le"): 🎯 CACHE HIT → return 1 (instant!)
            total += 1
          try "ur":   "pl"≠"ur" ✗
          try "le":   "pl"≠"le" ✗
          try "purpl":too long ✗
          return 1 → memo["ple"] = 1 ✅
        total += 1
      try "le":   "ur"≠"le" ✗
      try "purpl":"urple"[0..4]="urple"≠"purpl" ✗
      return 1 → memo["urple"] = 1 ✅
    totalCount += 1   ← "p"+"ur"+"p"+"le" path
  try "ur":    "pu"≠"ur" ✗
  try "le":    "pu"≠"le" ✗
  try "purpl": "purple" starts with "purpl" ✓
    suffix = "e"
    cc("e"): not in memo
      try all words: none match 'e' prefix ✗
      return 0 → memo["e"] = 0 ✅
    totalCount += 0   ← "purpl"+"e" dead end (e can't be built)
  return 2 → memo["purple"] = 2 ✅

FINAL: 2 ✓

MEMO CONTENTS:
  "le"     → 1    (1 way to build "le")
  "ple"    → 1    (1 way: "p"+"le")
  "urple"  → 1    (1 way: "ur"+"p"+"le")
  "e"      → 0    (no way to build "e")
  "purple" → 2    (2 ways total)

CACHE HIT SAVED: cc("le") was skipped the SECOND time it appeared
                 (from the "p"→"ur"→"p"→"le" path)
```

### Trace for countConstruct("abcdef", ["ab","abc","def"])

```
cc("abcdef"): not in memo
  try "ab": "ab" matches ✓ → cc("cdef")
    cc("cdef"): not in memo
      try "ab":  "cd"≠"ab" ✗
      try "abc": "cde"≠"abc" ✗
      try "def": "cde"≠"def" ✗
      return 0 → memo["cdef"] = 0 ✅
    totalCount += 0
  try "abc": "abc" matches ✓ → cc("def")
    cc("def"): not in memo
      try "ab":  "de"≠"ab" ✗
      try "abc": "def"≠"abc" ✗
      try "def": "def"="def" ✓ → cc("") = 1
        totalCount += 1
      return 1 → memo["def"] = 1 ✅
    totalCount += 1
  try "def": "abc"≠"def" ✗
  return 1 → memo["abcdef"] = 1 ✅

ANSWER: 1 way ("abc"+"def") ✓

MEMO CONTENTS:
  "cdef"   → 0    (no valid construction)
  "def"    → 1    ("def" alone)
  "abcdef" → 1    ("abc"+"def")
```

### Complexity Analysis — Memoization

```
TIME COMPLEXITY: O(n × m²)
───────────────────────────
Unique suffixes (unique subproblems): m+1
Each suffix computed EXACTLY ONCE.
Per suffix: try n words → n iterations.
Per iteration: substr + comparison → O(m).

→ (m+1) × n × O(m) = O(n × m²)

For cc("eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeef", 6 words):
  Recursion:    n^m × m = 6^32 × 32 ≈ 10^26 → infeasible
  Memoization:  n × m² = 6 × 32² ≈ 6,144 → instant! ✅

  Special for countConstruct "eeee...ef" case:
    Without memo: count the number of ways to partition 31 'e's — exponential.
    With memo:    memo["f"]=0, memo["ef"]=0, memo["eef"]=0, ...
                  Each suffix computed once → 32 entries, 6 words each = 192 ops.

SPACE COMPLEXITY: O(m²)
─────────────────────────
memo map: up to m+1 entries, each key is a suffix string up to m chars.
  → (m+1) × m = O(m²) for stored keys.
  → Values are ints → O(1) each.
call stack: m frames, each holds a suffix string up to m chars.
  → O(m²) stack space.
Total: O(m²)
```

---

## Technique 3 — Tabulation (Bottom-Up DP)

### The Idea

**Build an integer array `dp` of size `m+1` where `dp[i]` = number of ways
to construct `target[0..i-1]` from `wordBank`.**

```
dp[0] = 1   ← 1 way to construct "" (the empty prefix — pick nothing)
dp[i] = 0   ← for all i > 0 initially

SPREAD RULE (key change from canConstruct):
  canConstruct:   if(match) dp[i + wlen] = true;       ← boolean SET
  countConstruct: if(match) dp[i + wlen] += dp[i];     ← integer ACCUMULATE

WHY += dp[i]?
  If there are dp[i] ways to reach position i,
  and word matches at position i,
  then each of those dp[i] ways can be extended by this word
  to reach position i + word.length().
  → dp[i + word.length()] gets dp[i] MORE paths added to it.
```

### How the Table is Filled — countConstruct("purple", ["purp","p","ur","le","purpl"])

```
target   = "purple"   (m = 6)
wordBank = ["purp","p","ur","le","purpl"]

INITIAL TABLE: all 0, except dp[0] = 1

Index:    0      1      2      3      4      5      6
Char:    [·]     p      u      r      p      l      e
dp:      [ 1 ]  [ 0 ]  [ 0 ]  [ 0 ]  [ 0 ]  [ 0 ]  [ 0 ]
           ↑
       1 way to build "" (base case)

──────────────────────────────────────────────────────────────────

i=0: dp[0] = 1 → check each word starting at position 0
  "purp" (4): target[0..3] = "purp" = "purp" ✓
              dp[0+4] = dp[4] += dp[0] = dp[4] += 1 → dp[4] = 1
  "p"    (1): target[0..0] = "p"    = "p"    ✓
              dp[0+1] = dp[1] += dp[0] = dp[1] += 1 → dp[1] = 1
  "ur"   (2): target[0..1] = "pu"   ≠ "ur"   ✗
  "le"   (2): target[0..1] = "pu"   ≠ "le"   ✗
  "purpl"(5): target[0..4] = "purpl"= "purpl" ✓
              dp[0+5] = dp[5] += dp[0] = dp[5] += 1 → dp[5] = 1

Index:    0      1      2      3      4      5      6
dp:      [ 1 ]  [ 1 ]  [ 0 ]  [ 0 ]  [ 1 ]  [ 1 ]  [ 0 ]
                  ↑                    ↑      ↑
               "p" hit           "purp" hit  "purpl" hit

──────────────────────────────────────────────────────────────────

i=1: dp[1] = 1 → check from position 1
  "purp" (4): target[1..4] = "urpl" ≠ "purp" ✗
  "p"    (1): target[1..1] = "u"    ≠ "p"    ✗
  "ur"   (2): target[1..2] = "ur"   = "ur"   ✓
              dp[1+2] = dp[3] += dp[1] = dp[3] += 1 → dp[3] = 1
  "le"   (2): target[1..2] = "ur"   ≠ "le"   ✗
  "purpl"(5): target[1..5] = "urple"≠ "purpl" ✗

Index:    0      1      2      3      4      5      6
dp:      [ 1 ]  [ 1 ]  [ 0 ]  [ 1 ]  [ 1 ]  [ 1 ]  [ 0 ]
                               ↑
                          "ur" from pos 1 hit → dp[3]=1

──────────────────────────────────────────────────────────────────

i=2: dp[2] = 0 → SKIP (0 ways to reach position 2 → nothing to propagate)

──────────────────────────────────────────────────────────────────

i=3: dp[3] = 1 → check from position 3
  "purp" (4): 3+4=7 > 6 → out of bounds, skip
  "p"    (1): target[3..3] = "p"  = "p"    ✓
              dp[3+1] = dp[4] += dp[3] = dp[4] += 1 → dp[4] = 2
  "ur"   (2): target[3..4] = "pl" ≠ "ur"   ✗
  "le"   (2): target[3..4] = "pl" ≠ "le"   ✗
  "purpl"(5): 3+5=8 > 6 → skip

Index:    0      1      2      3      4      5      6
dp:      [ 1 ]  [ 1 ]  [ 0 ]  [ 1 ]  [ 2 ]  [ 1 ]  [ 0 ]
                                       ↑
                    dp[4] is now 2! (two ways to reach position 4:
                    1. "purp"    from position 0
                    2. "p"+"ur"+"p" from position 0→1→3→4)

──────────────────────────────────────────────────────────────────

i=4: dp[4] = 2 → check from position 4
  "purp" (4): 4+4=8 > 6 → skip
  "p"    (1): target[4..4] = "l"  ≠ "p"    ✗
  "ur"   (2): target[4..5] = "le" ≠ "ur"   ✗
  "le"   (2): target[4..5] = "le" = "le"   ✓
              dp[4+2] = dp[6] += dp[4] = dp[6] += 2 → dp[6] = 2
  "purpl"(5): 4+5=9 > 6 → skip

Index:    0      1      2      3      4      5      6
dp:      [ 1 ]  [ 1 ]  [ 0 ]  [ 1 ]  [ 2 ]  [ 1 ]  [ 2 ]
                                                      ↑
                                          dp[6] = 2! TARGET REACHED!
                                          "le" from position 4 contributed dp[4]=2

──────────────────────────────────────────────────────────────────

i=5: dp[5] = 1 → check from position 5
  "purp" (4): 5+4=9 > 6 → skip
  "p"    (1): target[5..5] = "e"  ≠ "p"    ✗
  "ur"   (2): 5+2=7 > 6 → skip
  "le"   (2): 5+2=7 > 6 → skip
  "purpl"(5): 5+5=10> 6 → skip
  (nothing can extend from position 5 within target bounds)

i=6: loop ends

FINAL TABLE:
Index:    0      1      2      3      4      5      6
dp:      [ 1 ]  [ 1 ]  [ 0 ]  [ 1 ]  [ 2 ]  [ 1 ]  [ 2 ]
                                                      ↑
                                      Answer: dp[6] = 2 ✓

READING THE TABLE:
  dp[0] = 1  → 1 way to build ""
  dp[1] = 1  → 1 way to build "p"         via "p"
  dp[2] = 0  → 0 ways to build "pu"       (no word is prefix of "pu")
  dp[3] = 1  → 1 way to build "pur"       via "p"+"ur"
  dp[4] = 2  → 2 ways to build "purp"     via "purp"  OR  "p"+"ur"+"p"
  dp[5] = 1  → 1 way to build "purpl"     via "purpl"
  dp[6] = 2  → 2 ways to build "purple"   via "purp"+"le"  OR  "p"+"ur"+"p"+"le"

WHY dp[4]=2 GIVES dp[6]=2:
  Both paths to "purp" can each be extended by "le" to reach "purple".
  dp[6] += dp[4] = += 2 → adds BOTH paths in one operation.
  This is the power of tabulation: dp[i] carries count of ALL paths to i.
```

### How the Table is Filled — countConstruct("abcdef", ["ab","abc","cd","def","abcd"])

```
target = "abcdef" (m=6), wordBank = ["ab","abc","cd","def","abcd"]

Start: dp = [1, 0, 0, 0, 0, 0, 0]

i=0: dp[0]=1
  "ab"(2):  "ab"="ab" ✓   → dp[2] += 1 → dp[2]=1
  "abc"(3): "abc"="abc" ✓ → dp[3] += 1 → dp[3]=1
  "cd"(2):  "ab"≠"cd" ✗
  "def"(3): "abc"≠"def" ✗
  "abcd"(4):"abcd"="abcd" ✓ → dp[4] += 1 → dp[4]=1

dp = [1, 0, 1, 1, 1, 0, 0]

i=2: dp[2]=1
  "ab"(2):  "cd"≠"ab" ✗
  "abc"(3): "cde"≠"abc" ✗
  "cd"(2):  "cd"="cd" ✓    → dp[4] += 1 → dp[4]=2
  "def"(3): "cde"≠"def" ✗
  "abcd"(4):"cdef"≠"abcd" ✗

dp = [1, 0, 1, 1, 2, 0, 0]
           ← dp[4] is now 2: via "abcd" or via "ab"+"cd" →

i=3: dp[3]=1
  "ab"(2):  "de"≠"ab" ✗
  "abc"(3): "def"≠"abc" ✗
  "cd"(2):  "de"≠"cd" ✗
  "def"(3): "def"="def" ✓  → dp[6] += dp[3] = dp[6] += 1 → dp[6]=1
  "abcd"(4):3+4=7>6, skip

dp = [1, 0, 1, 1, 2, 0, 1]

i=4: dp[4]=2
  "ab"(2):  "ef"≠"ab" ✗
  "abc"(3): 4+3>6, skip
  "cd"(2):  "ef"≠"cd" ✗
  "def"(3): 4+3>6, skip
  "abcd"(4):4+4>6, skip

dp unchanged.

i=5: dp[5]=0 → skip
i=6: loop ends.

FINAL: dp = [1, 0, 1, 1, 2, 0, 1]
Answer: dp[6] = 1 ✓

dp[4]=2 means "abcd" reachable in 2 ways: via "abcd" OR via "ab"+"cd"
But only ONE of those reaches dp[6]: via dp[3]→"def"→dp[6]
dp[6] = 1: only "abc"+"def" works.
```

### C++ Code

```cpp
#include <iostream>
#include <vector>
#include <string>
using namespace std;

// ─────────────────────────────────────────────────────────────
// TECHNIQUE 3: TABULATION (Bottom-Up DP)
// Time:  O(n × m²)
//   m+1 positions × n words × O(m) per substr comparison
// Space: O(m)
//   Only the integer dp array of size m+1.
//   NO suffix strings stored. NO call stack at all.
//   SAME space advantage as canConstruct tabulation!
// ─────────────────────────────────────────────────────────────

int countConstruct_tabulation(const string& target, const vector<string>& wordBank) {

    int m = target.length();

    // STEP 1: Create integer dp array, all 0 (except dp[0] = 1)
    vector<int> dp(m + 1, 0);

    // STEP 2: Base case — 1 way to construct the empty string
    dp[0] = 1;

    // STEP 3: Scan each position in the target
    for (int i = 0; i <= m; i++) {
        if (dp[i] == 0) continue;    // 0 ways to reach position i → skip

        // From position i, try each word
        for (const string& word : wordBank) {
            int wlen = word.length();

            if (i + wlen > m) continue;    // out of bounds → skip

            // Check if word matches target starting at position i
            if (target.substr(i, wlen) == word) {
                // KEY LINE: += instead of = true (the only tabulation change)
                dp[i + wlen] += dp[i];
            }
        }
    }

    // STEP 4: Answer is the count at the final position
    return dp[m];
}

// ─────────────────────────────────────────────────────────────
// BONUS: Print the dp table step-by-step (for learning)
// ─────────────────────────────────────────────────────────────
void printTable(const string& target, const vector<string>& wordBank) {
    int m = target.length();
    vector<int> dp(m + 1, 0);
    dp[0] = 1;

    cout << "\nBuilding dp table for countConstruct(\"" << target << "\", [";
    for (int i = 0; i < (int)wordBank.size(); i++) {
        cout << wordBank[i];
        if (i < (int)wordBank.size()-1) cout << ",";
    }
    cout << "]):" << endl;

    cout << "Index: ";
    for (int i = 0; i <= m; i++) cout << "[" << i << "]  ";
    cout << endl;

    cout << "Start: ";
    for (int i = 0; i <= m; i++) cout << " " << dp[i] << "    ";
    cout << endl;

    for (int i = 0; i <= m; i++) {
        if (dp[i] == 0) continue;
        bool changed = false;
        for (const string& word : wordBank) {
            int wlen = word.length();
            if (i + wlen <= m && target.substr(i, wlen) == word) {
                dp[i + wlen] += dp[i];
                changed = true;
            }
        }
        if (changed) {
            cout << "i=" << i << ":   ";
            for (int j = 0; j <= m; j++) cout << " " << dp[j] << "    ";
            cout << endl;
        }
    }

    cout << "Answer: dp[" << m << "] = " << dp[m] << endl;
}

int main() {
    cout << "countConstruct (Tabulation)" << endl;
    cout << "──────────────────────────────────────────────────────" << endl;

    cout << "cc(\"purple\",          [purp,p,ur,le,purpl]) = "
         << countConstruct_tabulation("purple", {"purp","p","ur","le","purpl"}) << endl;

    cout << "cc(\"abcdef\",          [ab,abc,cd,def,abcd]) = "
         << countConstruct_tabulation("abcdef", {"ab","abc","cd","def","abcd"}) << endl;

    cout << "cc(\"skateboard\",      [bo,rd,ate,t,ska,sk,boar]) = "
         << countConstruct_tabulation("skateboard", {"bo","rd","ate","t","ska","sk","boar"}) << endl;

    cout << "cc(\"enterapotentpot\", [a,p,ent,enter,ot,o,t]) = "
         << countConstruct_tabulation("enterapotentpot", {"a","p","ent","enter","ot","o","t"}) << endl;

    cout << "cc(\"\",                [cat,dog]) = "
         << countConstruct_tabulation("", {"cat","dog"}) << endl;

    cout << "cc(\"eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeef\", [e,ee,...]) = "
         << countConstruct_tabulation("eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeef",
                                      {"e","ee","eee","eeee","eeeee","eeeeee"}) << endl;

    printTable("purple", {"purp","p","ur","le","purpl"});
    printTable("abcdef", {"ab","abc","cd","def","abcd"});

    return 0;
}
```

**Output:**
```
countConstruct (Tabulation)
──────────────────────────────────────────────────────
cc("purple",          [purp,p,ur,le,purpl]) = 2
cc("abcdef",          [ab,abc,cd,def,abcd]) = 1
cc("skateboard",      [bo,rd,ate,t,ska,sk,boar]) = 0
cc("enterapotentpot", [a,p,ent,enter,ot,o,t]) = 4
cc("",                [cat,dog]) = 1
cc("eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeef",[e,...]) = 0

Building dp table for countConstruct("purple", [purp,p,ur,le,purpl]):
Index: [0]  [1]  [2]  [3]  [4]  [5]  [6]
Start:  1    0    0    0    0    0    0
i=0:    1    1    0    0    1    1    0    ← "p"→dp[1], "purp"→dp[4], "purpl"→dp[5]
i=1:    1    1    0    1    1    1    0    ← "ur"→dp[3]
i=3:    1    1    0    1    2    1    0    ← "p"→dp[4] (dp[4]:1→2)
i=4:    1    1    0    1    2    1    2    ← "le"→dp[6] += dp[4]=2
Answer: dp[6] = 2

Building dp table for countConstruct("abcdef", [ab,abc,cd,def,abcd]):
Index: [0]  [1]  [2]  [3]  [4]  [5]  [6]
Start:  1    0    0    0    0    0    0
i=0:    1    0    1    1    1    0    0    ← "ab","abc","abcd" matched
i=2:    1    0    1    1    2    0    0    ← "cd"→dp[4] += 1 (dp[4]:1→2)
i=3:    1    0    1    1    2    0    1    ← "def"→dp[6] += dp[3]=1
Answer: dp[6] = 1
```

### Complexity Analysis — Tabulation

```
TIME COMPLEXITY: O(n × m²)
────────────────────────────
Outer loop: i from 0 to m         → m+1 iterations
Inner loop: for each word         → n iterations per outer step
Each inner step: substr + compare → O(m)
Total: (m+1) × n × O(m) = O(n × m²)

Same as memoization. No recursion overhead. Faster in practice.

SPACE COMPLEXITY: O(m)   ← BEST (same advantage as canConstruct)
────────────────────────────────────────────────────────────────
dp array: m+1 integers → O(m)
No suffix strings stored anywhere.
No call stack (no recursion).

COMPARE:
  Memoization: O(m²) — stores suffix STRING keys (expensive)
  Tabulation:  O(m)  — stores only INTEGERS (cheap!)

NOTE: dp values can be very large (many ways to partition 'e's).
For "eeeeeeeeeeeeeeee" type inputs dp values grow exponentially.
Use long long (64-bit int) for large inputs to avoid overflow.
```

---

## All Three Together — One Complete File

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <unordered_map>
using namespace std;

// ════════════════════════════════════════════════════════════
// TECHNIQUE 1: RECURSION
// Time: O(n^m × m)    Space: O(m²)
// ════════════════════════════════════════════════════════════
int cc_recursive(const string& t, const vector<string>& wb) {
    if (t.empty()) return 1;
    int total = 0;
    for (const string& w : wb)
        if (t.length() >= w.length() && t.substr(0, w.length()) == w)
            total += cc_recursive(t.substr(w.length()), wb);
    return total;
}

// ════════════════════════════════════════════════════════════
// TECHNIQUE 2: MEMOIZATION (Top-Down DP)
// Time: O(n × m²)    Space: O(m²)
// ════════════════════════════════════════════════════════════
int cc_memo(const string& t, const vector<string>& wb,
            unordered_map<string, int>& memo) {
    if (memo.count(t)) return memo[t];
    if (t.empty()) return 1;
    int total = 0;
    for (const string& w : wb)
        if (t.length() >= w.length() && t.substr(0, w.length()) == w)
            total += cc_memo(t.substr(w.length()), wb, memo);
    return memo[t] = total;
}

// ════════════════════════════════════════════════════════════
// TECHNIQUE 3: TABULATION (Bottom-Up DP)
// Time: O(n × m²)    Space: O(m) — NO call stack, NO strings
// ════════════════════════════════════════════════════════════
int cc_tabulation(const string& t, const vector<string>& wb) {
    int m = t.length();
    vector<int> dp(m + 1, 0);
    dp[0] = 1;
    for (int i = 0; i <= m; i++) {
        if (!dp[i]) continue;
        for (const string& w : wb) {
            int wl = w.length();
            if (i + wl <= m && t.substr(i, wl) == w)
                dp[i + wl] += dp[i];    // the ONE change from canConstruct
        }
    }
    return dp[m];
}

// ════════════════════════════════════════════════════════════
// MAIN — Compare all techniques
// ════════════════════════════════════════════════════════════
int main() {
    cout << "target               Recursive  Memoized  Tabulation" << endl;
    cout << "──────────────────────────────────────────────────────" << endl;

    struct TC { string t; vector<string> wb; };
    vector<TC> tests = {
        {"purple",         {"purp","p","ur","le","purpl"}},
        {"abcdef",         {"ab","abc","cd","def","abcd"}},
        {"skateboard",     {"bo","rd","ate","t","ska","sk","boar"}},
        {"enterapotentpot",{"a","p","ent","enter","ot","o","t"}},
        {"",               {"cat","dog"}},
    };

    for (auto& [t, wb] : tests) {
        unordered_map<string, int> memo;
        cout << "\"" << (t.length() > 14 ? t.substr(0,11)+"..." : t) << "\"";
        cout << string(max(1, 20-(int)min(t.length()+2,(size_t)16)), ' ');
        cout << cc_recursive(t, wb)   << "          "
             << cc_memo(t, wb, memo) << "         "
             << cc_tabulation(t, wb) << endl;
    }

    cout << "\nLarge targets (memo + tabulation only):" << endl;
    cout << "────────────────────────────────────────────────────" << endl;
    vector<TC> large = {
        {"eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeef", {"e","ee","eee","eeee","eeeee","eeeeee"}},
        {"eeeeeeeeeeeeeeeeeeeeeeeeeeeeeee",  {"e","ee","eee","eeee","eeeee","eeeeee"}},
    };
    for (auto& [t, wb] : large) {
        unordered_map<string, int> memo;
        cout << "\"" << t.substr(0,10) << "...\"(len=" << t.length() << "):  ";
        cout << "Memo=" << cc_memo(t, wb, memo)
             << "  Tab=" << cc_tabulation(t, wb) << endl;
    }

    return 0;
}
```

**Output:**
```
target               Recursive  Memoized  Tabulation
──────────────────────────────────────────────────────
"purple"             2          2         2
"abcdef"             1          1         1
"skateboard"         0          0         0
"enterapotentpot"    4          4         4
""                   1          1         1

Large targets (memo + tabulation only):
────────────────────────────────────────────────────
"eeeeeeeeee..."(len=32):  Memo=0  Tab=0
"eeeeeeeeee..."(len=31):  Memo=<large number>  Tab=<large number>
```

---

## Complexity Summary — Side by Side

```
┌──────────────────┬─────────────────┬────────────────────────────────────────────┐
│ Technique        │      TIME       │                 SPACE                      │
├──────────────────┼─────────────────┼────────────────────────────────────────────┤
│ Recursion        │  O(n^m × m)     │ O(m²) — suffix strings on call stack       │
│                  │                 │ WARNING: Exponential. Never short-circuits. │
├──────────────────┼─────────────────┼────────────────────────────────────────────┤
│ Memoization      │  O(n × m²)      │ O(m²) — string keys + call stack           │
│ (Top-Down DP)    │                 │ Stores int per suffix (not bool like can)   │
├──────────────────┼─────────────────┼────────────────────────────────────────────┤
│ Tabulation       │  O(n × m²)      │ O(m)  — integer dp array only!             │
│ (Bottom-Up DP)   │                 │ NO call stack, NO strings, SAFEST          │
└──────────────────┴─────────────────┴────────────────────────────────────────────┘

n = wordBank.size(),  m = target.length()
```

---

## The One-Line Difference — canConstruct vs countConstruct

```
These two problems are IDENTICAL except for these substitutions:

RECURSION:
  canConstruct:    if(cc(suffix, wb)) return true;
  countConstruct:  total += cc(suffix, wb);

  canConstruct:    return false;
  countConstruct:  return total;    (0 = no ways, ≥1 = some ways)

MEMOIZATION:
  canConstruct:    unordered_map<string, bool>  memo;
  countConstruct:  unordered_map<string, int>   memo;

  canConstruct:    return memo[t] = true / false;
  countConstruct:  return memo[t] = total;

TABULATION:
  canConstruct:    dp[i + wlen] = true;
  countConstruct:  dp[i + wlen] += dp[i];

  canConstruct:    vector<bool> dp(m+1, false);
  countConstruct:  vector<int>  dp(m+1, 0);

  canConstruct:    dp[0] = true;
  countConstruct:  dp[0] = 1;

BASE CASE:
  canConstruct:    return true;   (found 1 valid construction — stop!)
  countConstruct:  return 1;      (found 1 valid construction — keep counting!)
```

---

## Understanding WHY the Complexities Are What They Are

### Why Recursion is Always Slower in Practice (same Big-O as canConstruct)

```
ASYMPTOTIC: O(n^m × m) — SAME as canConstruct

PRACTICE: countConstruct is drastically slower because:

canConstruct:
  At any node, if ONE child returns true → immediately return true.
  Best case (target directly in wordBank): O(m) total calls.
  Average case: much less than O(n^m) due to short-circuiting.

countConstruct:
  At EVERY node, must visit ALL n children — no early exit.
  Worst case is the SAME as average case: always O(n^m).
  Even when we know the answer early, we continue exploring.

ANALOGY: Counting votes vs asking if anyone voted yes.
  "Did anyone vote yes?" → stop asking as soon as one "yes" found.
  "How many voted yes?" → must ask everyone, every time.
```

### Why dp[i + wlen] += dp[i] is the Correct Rule

```
INTUITION: Paths multiply.

dp[i] = number of ways to reach position i.

If 3 paths reach position i, and word "abc" matches at position i,
then each of those 3 paths can be extended by "abc" to reach i+3.
→ 3 MORE paths reach position i+3.
→ dp[i+3] += 3 = dp[i+3] += dp[i].

EXAMPLE from "purple" trace:
  dp[4] = 2 (two paths reach position 4: "purp" and "p"+"ur"+"p")
  "le" matches at position 4 (target[4..5]="le")
  → both paths can be extended by "le" to reach position 6
  → dp[6] += dp[4] = += 2
  → dp[6] = 2 ✓ (two ways to build "purple")

COMPARE WITH canConstruct:
  canConstruct: dp[4] = true → dp[6] = true (just SET once)
  countConstruct: dp[4] = 2  → dp[6] += 2  (accumulate the count)
```

### Why Tabulation Space is O(m) While Memoization is O(m²)

```
Same reason as canConstruct:

MEMOIZATION stores suffix STRINGS as keys:
  m+1 entries × up to m chars each = O(m²) for keys alone.

TABULATION stores only INTEGERS in the dp array:
  m+1 entries × 4 bytes each (int) = O(m) space.

The values change:
  canConstruct:   bool (1 bit / 1 byte per entry) → O(m) for dp
  countConstruct: int  (4 bytes per entry)        → still O(m) for dp!
  (Both are O(m) — the element type doesn't change the asymptotic class)

NO suffix strings ever stored in tabulation → O(m) space confirmed.
```

---

## Visualizing "Count" Spread — Tabulation Intuition

```
Think of dp[i] as the NUMBER OF ROADS reaching junction i.
Multiple roads can reach the same junction.
Each road can be extended by a matching word.

countConstruct("purple", ["purp","p","ur","le","purpl"]):

Position: 0    1    2    3    4    5    6
          1    0    0    0    0    0    0    Start: 1 road at position 0

After i=0 ("p"→1, "purp"→4, "purpl"→5):
          1    1    0    0    1    1    0    Counts of roads

After i=1 ("ur"→3):
          1    1    0    1    1    1    0    1 road now reaches position 3

After i=3 ("p"→4):
          1    1    0    1    2    1    0    Position 4 now has 2 roads!
                                            (one via "purp", one via "p"+"ur"+"p")

After i=4 ("le"→6, adding dp[4]=2):
          1    1    0    1    2    1    2    Position 6 has 2 roads = 2 ways ✓

Each "road" from position 0 to position 6 is one valid construction.
dp[i] counts the roads. += propagates all roads forward simultaneously.
```

---

## canConstruct vs countConstruct — Full Comparison

```
┌─────────────────────┬─────────────────────────┬──────────────────────────────┐
│                     │    canConstruct          │     countConstruct           │
├─────────────────────┼─────────────────────────┼──────────────────────────────┤
│ Question            │ Is there ≥1 way?         │ How many ways exactly?       │
│ Return type         │ bool                     │ int                          │
│ Base case return    │ true                     │ 1                            │
│ Dead end return     │ false                    │ 0 (auto — no explicit check) │
│ Branch combination  │ OR (short-circuit)       │ + (accumulate, no exit)      │
│ Short-circuit?      │ YES (stop at first true) │ NO (always explore all)      │
│                     │                          │                              │
│ Recursion return    │ return true;             │ return total;                │
│ Recursion combo     │ if(cc) return true;      │ total += cc(suffix);         │
│                     │                          │                              │
│ Memo value type     │ bool                     │ int                          │
│ Memo store          │ memo[t] = true/false     │ memo[t] = total count        │
│                     │                          │                              │
│ Tab array type      │ vector<bool>             │ vector<int>                  │
│ Tab base            │ dp[0] = true             │ dp[0] = 1                    │
│ Tab update          │ dp[j] = true;            │ dp[j] += dp[i];              │
│                     │                          │                              │
│ Recursion TC        │ O(n^m × m)               │ O(n^m × m)                   │
│ Recursion SC        │ O(m²)                    │ O(m²)                        │
│ Memo TC             │ O(n × m²)                │ O(n × m²)                    │
│ Memo SC             │ O(m²)                    │ O(m²)                        │
│ Tab TC              │ O(n × m²)                │ O(n × m²)                    │
│ Tab SC              │ O(m)                     │ O(m)                         │
├─────────────────────┼─────────────────────────┼──────────────────────────────┤
│ "purple" result     │ true                     │ 2                            │
│ "skateboard" result │ false                    │ 0                            │
│ "" result           │ true                     │ 1                            │
└─────────────────────┴─────────────────────────┴──────────────────────────────┘
```

---

## The Interview Answer — What to Say and When

```
INTERVIEWER: "Count the number of ways to construct target from wordBank."

STEP 1: Connect to canConstruct
  "This is canConstruct but instead of returning true/false,
   we count ALL valid constructions.
   Same structure, same base cases, same prefix-matching logic.
   Only three things change: return type, base case value, and how
   we combine results — OR becomes addition, true becomes 1."

STEP 2: Write recursive solution
  "Base case: empty string → return 1 (one way: pick nothing).
   For each word matching the prefix: suffix = target.substr(wlen).
   total += countConstruct(suffix, wordBank).
   Return total."
  → Write ~8 lines. Emphasize: NO early return, full loop always.

  "Time: O(n^m × m) — worse than canConstruct in practice
   because we can never short-circuit."

STEP 3: Identify repeated subproblems
  "Same suffix 'le' appears from different paths — computed multiple times.
   Cache the COUNT per suffix, not just true/false."

STEP 4: Memoize
  "Change map<string,bool> to map<string,int>.
   Cache and return total instead of true/false.
   Time: O(n × m²), Space: O(m²)."

STEP 5: Tabulate
  "dp[i] = number of ways to construct first i characters.
   dp[0] = 1 (base). For each true dp[i] and each matching word:
   dp[i + wlen] += dp[i]  ← the += is the entire difference from canConstruct.
   Time: O(n × m²), Space: O(m) — no strings stored."
```

---

## Quick Cheat Sheet

```
PROBLEM: countConstruct(target, wordBank)
         Count the number of ways to build target by concatenating words.
         Words reusable. Return 0 if impossible.

─────────────────────────────────────────────────────────────────

RECURSION:
  int cc(const string& t, const vector<string>& wb) {
    if(t.empty()) return 1;
    int total = 0;
    for(const string& w : wb)
      if(t.length()>=w.length() && t.substr(0,w.length())==w)
        total += cc(t.substr(w.length()), wb);   // ADD, no return!
    return total;
  }
  Time: O(n^m × m)   Space: O(m²)

─────────────────────────────────────────────────────────────────

MEMOIZATION:
  // Extra param: unordered_map<string, int>& memo
  if(memo.count(t)) return memo[t];
  if(t.empty()) return 1;
  int total = 0;
  for(const string& w : wb)
    if(t.length()>=w.length() && t.substr(0,w.length())==w)
      total += cc_memo(t.substr(w.length()), wb, memo);
  return memo[t] = total;

  Time: O(n×m²)   Space: O(m²)

─────────────────────────────────────────────────────────────────

TABULATION:
  vector<int> dp(m+1, 0);   // int, not bool
  dp[0] = 1;                // 1, not true
  for(int i=0; i<=m; i++) {
    if(!dp[i]) continue;
    for(const string& w : wb) {
      int wl = w.length();
      if(i+wl<=m && target.substr(i,wl)==w)
        dp[i+wl] += dp[i];  // +=, not =true  ← THE ONLY CHANGE
    }
  }
  return dp[m];

  Time: O(n×m²)   Space: O(m)  ← best!

─────────────────────────────────────────────────────────────────

ANSWERS:
  cc("purple",         [purp,p,ur,le,purpl])  = 2
  cc("abcdef",         [ab,abc,cd,def,abcd])  = 1
  cc("skateboard",     [bo,rd,ate,t,...])      = 0
  cc("enterapotentpot",[a,p,ent,enter,ot,o,t])= 4
  cc("",               [cat,dog])              = 1    ← base case!
  cc("eee...ef",       [e,ee,eee,...])         = 0    ← 'f' unmatchable

CANSTRUCT VS COUNTCONSTRUCT SUBSTITUTIONS:
  bool  →  int
  true  →  1
  false →  0   (naturally, when total stays 0)
  ||    →  +=
  return true  →  total += ...
  vector<bool> →  vector<int>
  dp[j] = true →  dp[j] += dp[i]
  dp[0] = true →  dp[0] = 1
```

---

*Compile and run: `g++ -std=c++17 -o countconstruct count_construct.cpp && ./countconstruct`*
*(C++17 needed for structured bindings in main — core logic works with C++11/14)*
