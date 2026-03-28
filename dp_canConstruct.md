# canConstruct Problem — Three Techniques
## Recursion vs Memoization vs Tabulation in C++

> **Problem:** Given a `target` string and an array of strings `wordBank`,
> return `true` if the `target` can be constructed by concatenating words from `wordBank`.
> You may reuse words from `wordBank` as many times as you like.
> All characters are lowercase English letters.

---

## canConstruct vs canSum — The Same Problem in a Different Dimension

```
canSum(7, [2, 3])          →  true / false
  "Can we REACH target by ADDING numbers?"
  Operation: subtraction   (target - num)
  Base case: target == 0   (hit zero exactly)

canConstruct("abcdef", ["ab","def"]) →  true / false
  "Can we BUILD target by JOINING strings?"
  Operation: prefix removal  (target minus matching prefix word)
  Base case: target == ""  (string fully consumed)

THEY ARE THE SAME PATTERN:
  canSum:       pick a number  → subtract from target → recurse on remainder
  canConstruct: pick a word    → slice off from front → recurse on suffix

ANALOGY TABLE:
  canSum          canConstruct          What it does
  ─────────────────────────────────────────────────────────────
  targetSum       target string         What we're trying to reach
  numbers array   wordBank array        Tools we can use (reusable)
  num             word                  One tool picked per step
  target - num    target.substr(len)    What's left after picking
  target == 0     target.empty()        Base case: done!
  target < 0      (no prefix match)     Dead end: can't use this word
```

---

## Quick Comparison (Read This First)

```
TECHNIQUE       TIME            SPACE         STYLE       BEST FOR
──────────────────────────────────────────────────────────────────────
Recursion       O(n^m × m)      O(m²)         Top-down    Understanding
Memoization     O(n × m²)       O(m²)         Top-down    Recursive + cache
Tabulation      O(n × m²)       O(m)          Bottom-up   BEST — no stack!

Where: m = target.length,  n = wordBank.size()

KEY INSIGHT: Tabulation space is O(m) while Memoization is O(m²).
This is because:
  Memoization stores strings as keys → up to m strings of up to m length = O(m²)
  Tabulation stores only a boolean array of size m+1 → O(m)
```

---

## New Terms You Need for This Problem

```
TERM          MEANING                              EXAMPLE (target = "abcdef")
──────────────────────────────────────────────────────────────────────────────
Prefix        Characters at the START of a string  "abc" is a prefix of "abcdef"
              (matches from position 0)

Suffix        Characters at the END of a string    "def" is a suffix of "abcdef"
              (what remains after removing prefix)

Substring     Any contiguous slice of a string     "bcde" is a substring of "abcdef"

target[i..j]  Substring from index i to j          target[2..4] = "cde"

substr(i,len) C++ function: slice string at         "abcdef".substr(2,3) = "cde"
              position i with length len             position 2, take 3 chars

Prefix match  Does word match at the FRONT          "abc".startsWith("ab")? YES
              of the current target/suffix?          "abc".startsWith("bc")? NO

"eats" prefix When word matches, we "consume" it:  "abcdef" - "abc" = "def"
              remove it from the front of target    remaining suffix = "def"
```

---

## The Problem Visualized

```
canConstruct("abcdef", ["ab","abc","cd","def","abcd"])

THINK OF IT LIKE CUTTING A RIBBON:
  Target string:  a  b  c  d  e  f
                  ├──┤              "ab"   → leaves "cdef"
                  ├────┤            "abc"  → leaves "def"
                  ├────────┤        "abcd" → leaves "ef"

  After picking "abc", try cutting "def":
  Remaining:  d  e  f
              ├────┤                "def" → leaves ""

  "" is empty → BASE CASE → return true ✅
  PATH: "abc" + "def" = "abcdef" ✓

──────────────────────────────────────────────────────────────────

canConstruct("skateboard", ["bo","rd","ate","t","ska","sk","boar"])

  s  k  a  t  e  b  o  a  r  d
  ├────┤                          "sk" → leaves "ateboard"
  ├──────┤                        "ska" → leaves "teboard"

  Try from "teboard":
  "bo": "te"≠"bo" ✗
  "rd": "te"≠"rd" ✗
  "ate": "teb"≠"ate" ✗  ← "t" ≠ "a" at position 0
  "t": "t"="t" ✓ → leaves "eboard"

  Try from "eboard":
  None of ["bo","rd","ate","t","ska","sk","boar"] start with 'e' → ALL fail
  → false — backtrack all the way → false ✗

  All paths fail → canConstruct("skateboard") = false ✗

──────────────────────────────────────────────────────────────────

canConstruct("enterapotentpot", ["a","p","ent","enter","ot","o","t"])

  PATH: "enter" + "a" + "pot" ... wait: "pot" not in bank. Try:
  "ent" + "er" ... "er" not in bank. Try:
  "enter" (5 chars) → leaves "apotentpot"
    "a" (1 char)    → leaves "potentpot"
      "p" (1 char)  → leaves "otentpot"
        "ot" (2)    → leaves "entpot"
          "ent" (3) → leaves "pot"
            "p"(1)  → leaves "ot"
              "ot"(2) → leaves ""  → true ✅

  PATH: enter + a + p + ot + ent + p + ot
  Full: "enter" + "a" + "p" + "ot" + "ent" + "p" + "ot" = "enterapotentpot" ✓
```

**Test Cases for Reference:**

```
canConstruct("abcdef",         ["ab","abc","cd","def","abcd"])  = true  ("abc"+"def")
canConstruct("skateboard",     ["bo","rd","ate","t","ska","sk","boar"]) = false
canConstruct("enterapotentpot",["a","p","ent","enter","ot","o","t"])    = true
canConstruct("",               ["cat","dog","horse"])                    = true  (base!)
canConstruct("eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeef",
             ["e","ee","eee","eeee","eeeee","eeeeee"])                  = false
             ← "f" at the end can never be matched by any word in bank
             ← Without memoization, this is catastrophically slow!
```

---

## The Recursive Formula

```
canConstruct(target, wordBank):

  BASE CASE: target is empty string ("")
    → return true   (we've fully consumed the target — success!)

  FOR EACH word in wordBank:
    IF target STARTS WITH word:         ← prefix check
      suffix = target AFTER removing word from front
      IF canConstruct(suffix, wordBank) == true:
        return true                     ← this path works!

  return false                          ← no word helped from here

INTUITION (mirrors canSum exactly):
  canSum("can we reach 7?"):
    try num=3: "can we reach 7-3=4?" → recurse on 4
    if canSum(4) is true → return true

  canConstruct("can we build 'abcdef'?"):
    try word="abc": "can we build 'def'?" → recurse on "def"
    if canConstruct("def") is true → return true

THE KEY OPERATION:
  canSum:       remainder  = targetSum - num
  canConstruct: suffix     = target.substr(word.length())
  Both: shrink the problem, recurse, check if base case is reached.
```

---

## Technique 1 — Recursion (Naive)

### The Idea

At each call, scan through every word in `wordBank`.
If the word matches the beginning of `target` (prefix match):
  - Slice it off → get the `suffix`
  - Recurse on the suffix
  - If recursion returns `true` → return `true` immediately (short-circuit)
If no word works → return `false`.

### How Prefix Matching Works

```
target = "abcdef"

Checking word = "ab":
  target.substr(0, 2) = "ab"
  "ab" == "ab" → MATCH ✓
  suffix = target.substr(2) = "cdef"

Checking word = "abc":
  target.substr(0, 3) = "abc"
  "abc" == "abc" → MATCH ✓
  suffix = target.substr(3) = "def"

Checking word = "cd":
  target.substr(0, 2) = "ab"
  "ab" == "cd" → NO MATCH ✗

Checking word = "abcg":
  target.substr(0, 4) = "abcd"
  "abcd" == "abcg" → NO MATCH ✗  ('d' ≠ 'g')

IMPORTANT: We only check if word matches at position 0 (the FRONT).
           The suffix carries forward what's left unmatched.
           substr() in C++ costs O(m) — it creates a new string object.
```

### Recursion Tree for canConstruct("abcdef", ["ab","abc","cd","def","abcd"])

```
                      cc("abcdef")
                     /       |         \
              cc("cdef")  cc("def")  cc("ef")
              [ab matched] [abc matched] [abcd matched]
             /      \         /    \
        cc("ef") cc("f")  cc("")  cc("ef")
       [cd mtch] [no mtch] ↑        [def≠cd ✗]
           |               |
     no match           BASE CASE!
       false             true ✅

→ cc("def") branch finds: "def" matches → cc("") → true!
→ true propagates up to cc("abcdef") → return true ✅

LABELS:
  cc = canConstruct (shortened)
  Branches labeled with which word was matched to get there
  "no match" = no word in wordBank is a prefix of this suffix

SHORT-CIRCUIT: Once cc("def") → true is found, cc("abcdef") returns immediately.
               cc("ef") (from "abcd") is never explored!
```

### Recursion Tree for canConstruct("eeeef", ["e","ee","eee","eeee"]) — REPEATED WORK

```
                           cc("eeeef")
                  ┌──────────┼──────────┬────────────┐
               cc("eeef")  cc("eef")  cc("ef")  cc("f")
              /  |  |  \    ...         ...        ✗
        cc("eef") ...  ✗
        /  |  \
   cc("ef") ...  ✗
       ✗

REPEATED SUBPROBLEMS:
  cc("eef") computed from cc("eeeef") branch AND cc("eeef") branch!
  cc("ef")  computed MULTIPLE times from different parents!
  cc("f")   computed MANY times — always returns false (no word matches "f...")

Each unique suffix should be computed EXACTLY ONCE.
Memoization will fix this!

WHY THIS IS CATASTROPHIC:
  Target: "eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeef" (31 chars, all 'e', ends with 'f')
  Every 'e' prefix matches — branches at every position.
  Final 'f' always fails — but recursion discovers this only after exploring
  every possible partition of the 'e's (exponential combinations!).
  Result: hangs without memoization on this 31-character string.
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
//   n^m recursive calls (branching factor n, depth m).
//   Each call: O(m) for substr() to create the suffix string.
// Space: O(m²)
//   Call stack depth: at most m frames.
//   Each frame holds a suffix string of up to O(m) characters.
//   m frames × O(m) string = O(m²) total.
// ─────────────────────────────────────────────────────────────

bool canConstruct_recursive(const string& target, const vector<string>& wordBank) {

    // BASE CASE: empty string — target fully consumed, valid construction!
    if (target.empty()) return true;

    // Try each word as a potential prefix match
    for (const string& word : wordBank) {

        // Check: does 'word' match the beginning of 'target'?
        if (target.length() >= word.length() &&
            target.substr(0, word.length()) == word) {

            // Slice off the matched prefix → get the suffix
            string suffix = target.substr(word.length());

            // Recursively check if suffix can be constructed
            if (canConstruct_recursive(suffix, wordBank)) {
                return true;    // found a valid path — short-circuit!
            }
        }
    }

    // No word in wordBank could start a valid construction from here
    return false;
}

int main() {
    cout << "canConstruct (Recursion)" << endl;
    cout << "──────────────────────────────────────────────────────" << endl;

    cout << boolalpha;

    cout << "cc(\"abcdef\",    [ab,abc,cd,def,abcd])  = "
         << canConstruct_recursive("abcdef", {"ab","abc","cd","def","abcd"}) << endl;

    cout << "cc(\"skateboard\",[bo,rd,ate,t,ska,sk,boar]) = "
         << canConstruct_recursive("skateboard", {"bo","rd","ate","t","ska","sk","boar"}) << endl;

    cout << "cc(\"\",          [cat,dog])             = "
         << canConstruct_recursive("", {"cat","dog"}) << endl;

    cout << "cc(\"abcdef\",    [ab,cd,abcg])          = "
         << canConstruct_recursive("abcdef", {"ab","cd","abcg"}) << endl;

    // WARNING: Do NOT run the "eeeef" example without memoization!
    // canConstruct_recursive("eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeef", {...})
    // → Will hang — exponential calls all ending in "f" which never matches.

    return 0;
}
```

**Output:**
```
canConstruct (Recursion)
──────────────────────────────────────────────────────
cc("abcdef",    [ab,abc,cd,def,abcd])  = true
cc("skateboard",[bo,rd,ate,t,ska,sk,boar]) = false
cc("",          [cat,dog])             = true
cc("abcdef",    [ab,cd,abcg])          = false
```

### Complexity Analysis — Recursion

```
TIME COMPLEXITY: O(n^m × m)
────────────────────────────
PART 1 — Number of recursive calls: O(n^m)
  At each call, we try n words (one per wordBank entry).
  Depth of recursion: at most m (each step removes ≥ 1 character).
  Total nodes in tree: n^m   (same reasoning as canSum)

PART 2 — Work per call: O(m)
  substr(0, word.length()) creates a new string → O(m) in worst case.
  String comparison == word:                    → O(m) in worst case.
  TOTAL per call: O(m)

TOTAL: O(n^m) calls × O(m) per call = O(n^m × m)

For canConstruct("eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeef", 6 words):
  n=6, m=32 → 6^32 ≈ 10^24 calls → utterly infeasible!

SPACE COMPLEXITY: O(m²)
────────────────────────
TWO sources:

1. CALL STACK:
   Recursion can go m levels deep (1 char consumed per level worst case).
   → At most m frames simultaneously on the stack.

2. SUFFIX STRINGS:
   Each stack frame holds a suffix string.
   Suffix at depth d has length: m - d   (shrinks by word length each level)
   Strings at depths 0,1,...,m have total size: m + (m-1) + ... + 1 = O(m²)
   But only m are on stack at once: sizes m, m-w1, m-w1-w2, ...
   In the worst case (word length = 1): frames hold m, m-1, m-2, ..., 1
   → Total at once: O(m + m-1 + ... + 1) ÷ m per frame ≈ O(m) per frame
   The SINGLE DEEPEST PATH holds m strings summing to O(m²).

TOTAL: O(m²) — worse than canSum's O(m) because strings (not ints) are copied.

Compare with canSum recursion space:
  canSum:       O(m)  — just integers on the call stack
  canConstruct: O(m²) — string objects on the call stack
```

---

## Technique 2 — Memoization (Top-Down DP)

### The Idea

**Cache the result for each unique suffix string, so it is computed only once.**

Key insight: `canConstruct("def", wordBank)` gives the SAME answer regardless of
how we arrived at the suffix `"def"`. So once computed, store it — never recompute it.

Use `unordered_map<string, bool>` as the cache.

```cpp
unordered_map<string, bool> memo;

// Key = the suffix string at this point in the recursion
// Value = true (suffix CAN be constructed) or false (cannot)
```

### Why the Suffix String is a Unique Cache Key

```
EVERY unique suffix of "abcdef" represents a UNIQUE SUBPROBLEM:
  ""       → trivially true (base case)
  "f"      → depends on whether any word is a prefix of "f"
  "ef"     → depends on "f" and ""
  "def"    → depends on "ef" and "f" and ""
  "cdef"   → depends on "def" and others
  "bcdef"  → depends on "cdef" and others
  "abcdef" → the original problem

Total unique suffixes = m+1 (one for each starting position in target, plus "").
Once any suffix is computed, its result is stored.
Any future call with the SAME suffix → instant cache hit.

HOW MANY UNIQUE SUFFIXES EXIST?
  target = "abcdef" (length 6)
  Suffixes: "", "f", "ef", "def", "cdef", "bcdef", "abcdef"
  Count: m+1 = 7
  Each unique suffix computed ONCE → at most m+1 entries in memo.
```

### How Memoization Prunes the Tree

```
WITHOUT MEMO — canConstruct("eeeef", ["e","ee"]) explores "eef" multiple times:

                         cc("eeeef")
                        /            \
                  cc("eeef")       cc("eef")   ← explored from ROOT
                  /     \
            cc("eef")  cc("ef")    ← cc("eef") explored AGAIN from cc("eeef")!
            ...

WITH MEMO:
  cc("eef") is computed ONCE (from cc("eeef")).
  Result: cc("eef") = false.
  memo["eef"] = false ✅ stored.

  When cc("eeeef") tries its second branch (ee → suffix "eef"):
  → 🎯 CACHE HIT: memo["eef"] = false → return immediately.
     Entire subtree of cc("eef") is SKIPPED.

MEMO CONTENTS after canConstruct("eeeef", ["e","ee"]):
  memo["eef"]   = false ✅
  memo["ef"]    = false ✅
  memo["f"]     = false ✅  ← "f" matches nothing — cached fast
  memo["eeef"]  = false ✅
  memo["eeeef"] = false ✅

All return false because 'f' at the end can never be matched.
Each false is computed ONCE and reused.
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
//   memo: up to m+1 string keys, each up to m characters → O(m²)
//   call stack: O(m) depth
//   Total: O(m²)
// ─────────────────────────────────────────────────────────────

bool canConstruct_memo(const string& target,
                       const vector<string>& wordBank,
                       unordered_map<string, bool>& memo) {

    // STEP 1: Cache check — if this suffix was already computed, return it
    if (memo.count(target)) {
        return memo[target];    // CACHE HIT — instant return
    }

    // STEP 2: Base case
    if (target.empty()) return true;

    // STEP 3: Try each word as a prefix match
    for (const string& word : wordBank) {
        if (target.length() >= word.length() &&
            target.substr(0, word.length()) == word) {

            string suffix = target.substr(word.length());

            if (canConstruct_memo(suffix, wordBank, memo)) {
                return memo[target] = true;    // cache true and return
            }
        }
    }

    // STEP 4: No word worked — cache false and return
    return memo[target] = false;
}

int main() {
    cout << "canConstruct (Memoization)" << endl;
    cout << "──────────────────────────────────────────────────────" << endl;

    cout << boolalpha;

    auto run = [](const string& t, vector<string> wb) {
        unordered_map<string, bool> memo;
        return canConstruct_memo(t, wb, memo);
    };

    cout << "cc(\"abcdef\",    [ab,abc,cd,def,abcd])      = "
         << run("abcdef", {"ab","abc","cd","def","abcd"}) << endl;

    cout << "cc(\"skateboard\",[bo,rd,ate,t,ska,sk,boar]) = "
         << run("skateboard", {"bo","rd","ate","t","ska","sk","boar"}) << endl;

    cout << "cc(\"enterapotentpot\", [a,p,ent,enter,ot,o,t]) = "
         << run("enterapotentpot", {"a","p","ent","enter","ot","o","t"}) << endl;

    cout << "cc(\"\", [cat,dog])                            = "
         << run("", {"cat","dog"}) << endl;

    cout << "cc(\"eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeef\", [e,ee,eee,eeee,eeeee,eeeeee]) = "
         << run("eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeef",
                {"e","ee","eee","eeee","eeeee","eeeeee"}) << endl;
    // ↑ Instant with memoization! (was ~10^24 calls without memo)

    return 0;
}
```

**Output:**
```
canConstruct (Memoization)
──────────────────────────────────────────────────────
cc("abcdef",    [ab,abc,cd,def,abcd])      = true
cc("skateboard",[bo,rd,ate,t,ska,sk,boar]) = false
cc("enterapotentpot", [a,p,ent,enter,ot,o,t]) = true
cc("", [cat,dog])                            = true
cc("eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeef", [e,ee,...]) = false
```

### Step-by-Step Trace for canConstruct("abcdef", ["ab","abc","def"]) with Memo

```
cc = canConstruct (shortened), 🎯 = cache hit, ✅ = stored in memo

cc("abcdef"): not in memo
  try "ab": "abcdef" starts with "ab" ✓
    suffix = "cdef"
    cc("cdef"): not in memo
      try "ab":  "cdef" starts with "ab"? "cd"≠"ab" ✗
      try "abc": "cde"≠"abc" ✗
      try "def": "cde"≠"def" ✗  ("cdef"[0..2]="cde", not "def")
      → all fail → memo["cdef"] = false ✅  return false
    ← false
  try "abc": "abcdef" starts with "abc" ✓
    suffix = "def"
    cc("def"): not in memo
      try "ab":  "def" starts with "ab"? "de"≠"ab" ✗
      try "abc": "def"≠"abc" ✗
      try "def": "def"="def" ✓
        suffix = ""
        cc(""): target.empty() → return true!  ← BASE CASE
      → memo["def"] = true ✅  return true
    ← true
  → memo["abcdef"] = true ✅  return true

FINAL MEMO TABLE:
  "cdef"   → false   (computed and cached — no word can construct "cdef")
  "def"    → true    (constructed by "def" alone)
  "abcdef" → true    (constructed by "abc" + "def")

BENEFIT: If any subsequent call needed cc("cdef") or cc("def"),
         it returns instantly from the memo cache.
```

### Trace for the "EEEEF" Problem — Seeing Memo in Action

```
cc("eeeef", ["e","ee"]):

cc("eeeef"): not in memo
  try "e": suffix = "eeef"
    cc("eeef"): not in memo
      try "e": suffix = "eef"
        cc("eef"): not in memo
          try "e": suffix = "ef"
            cc("ef"): not in memo
              try "e": suffix = "f"
                cc("f"): not in memo
                  try "e": "f"≠"e" ✗
                  try "ee": "f" too short ✗
                  → memo["f"] = false ✅  return false
              try "ee": "ef"[0..1]="ef"≠"ee" ✗
              → memo["ef"] = false ✅  return false
          try "ee": suffix = "f"
            cc("f"): 🎯 CACHE HIT → false (instant!)
          → memo["eef"] = false ✅  return false
        cc("eef") = false  ← computed above
      try "ee": suffix = "ef"
        cc("ef"): 🎯 CACHE HIT → false (instant!)
      → memo["eeef"] = false ✅  return false
  try "ee": suffix = "eef"
    cc("eef"): 🎯 CACHE HIT → false (instant!)
  → memo["eeeef"] = false ✅  return false

CACHE HITS SAVED: cc("f") once, cc("ef") once, cc("eef") once
WITHOUT MEMO: exponential repeated computation of these same suffixes
WITH MEMO:    each suffix computed ONCE, reused via cache
```

### Complexity Analysis — Memoization

```
TIME COMPLEXITY: O(n × m²)
───────────────────────────
Unique suffixes (unique subproblems): m+1 (one per starting position + "")
Each unique suffix computed EXACTLY ONCE.
For each suffix, we try n words → n prefix checks.
Each prefix check: substr(0, word.length()) comparison → O(m) in worst case.

→ Total: (m+1) states × n words × O(m) per check
       = O(n × m × m) = O(n × m²)

For canConstruct("eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeef", 6 words):
  Recursion:    n^m = 6^32 ≈ 10^24 → infeasible
  Memoization:  n × m² = 6 × 32² = 6,144 operations → instant! ✅

SPACE COMPLEXITY: O(m²)
─────────────────────────
1. MEMO HASH MAP:
   At most m+1 entries.
   Each KEY is a suffix string of length up to m.
   Storage: m+1 entries × up to m chars each → O(m²) for keys.
   (Values are booleans → O(1) each)
   Total memo space: O(m²)

2. CALL STACK:
   At most m frames deep.
   Each frame holds a suffix string of up to m chars.
   Total stack space: O(m²)   (sum of suffix lengths: m + m-1 + ... ≈ m²/2)

Total: O(m²) + O(m²) = O(m²)

COMPARE WITH RECURSION:
  Both memo and recursion have O(m²) space.
  But memo time is O(n×m²) vs recursion O(n^m × m).
  The time difference is dramatic; space is the same order.
```

---

## Technique 3 — Tabulation (Bottom-Up DP)

### The Idea

**Build a boolean array `dp` of size `m+1` from the bottom up.**

Instead of working with suffix STRINGS (like recursion/memo), we work with
INDICES into the target string.

```
dp[i] = "Can target[0..i-1] be constructed from wordBank?"

Equivalently: "Can the first i characters of target be constructed?"

dp[0] = true   ← empty prefix "" is always constructible (base case)
dp[i] = false  ← for all i > 0 initially

SPREAD RULE:
  If dp[i] = true AND word matches target starting at position i:
    → dp[i + word.length()] = true
    (Because: first i chars are constructible + word fills the next stretch)
```

### The Index-to-String Connection (Critical to Understand)

```
target = "abcdef"

Index i    What dp[i] means                   Equivalent suffix
─────────────────────────────────────────────────────────────────
dp[0]      Can ""         be constructed?  ←→ suffix starting at 0 = "abcdef"
dp[1]      Can "a"        be constructed?  ←→ suffix starting at 1 = "bcdef"
dp[2]      Can "ab"       be constructed?  ←→ suffix starting at 2 = "cdef"
dp[3]      Can "abc"      be constructed?  ←→ suffix starting at 3 = "def"
dp[4]      Can "abcd"     be constructed?  ←→ suffix starting at 4 = "ef"
dp[5]      Can "abcde"    be constructed?  ←→ suffix starting at 5 = "f"
dp[6]      Can "abcdef"   be constructed?  ←→ ""  (fully consumed)

ANSWER: dp[m] = dp[6] = "Can the full target be constructed?"

INDEX-BASED CHECK:
  Instead of: suffix.substr(0, word.length()) == word
  We check:   target.substr(i, word.length()) == word

  "Does word match target STARTING AT POSITION i?"
```

### How the Table is Filled — canConstruct("abcdef", ["ab","abc","cd","def","abcd"])

```
target   = "abcdef"   (m = 6)
wordBank = ["ab","abc","cd","def","abcd"]

INITIAL TABLE: all false, except dp[0] = true

Index:    0      1      2      3      4      5      6
Char:    [start]  a      b      c      d      e      f
dp:      [ T ]  [ F ]  [ F ]  [ F ]  [ F ]  [ F ]  [ F ]
           ↑
        Always true: empty prefix "" is constructible

──────────────────────────────────────────────────────────────────

i=0: dp[0] = true → check each word starting at position 0
  "ab"   (len 2): target[0..1] = "ab"   == "ab"   ✓ → dp[0+2] = dp[2] = true
  "abc"  (len 3): target[0..2] = "abc"  == "abc"  ✓ → dp[0+3] = dp[3] = true
  "cd"   (len 2): target[0..1] = "ab"   ≠  "cd"   ✗
  "def"  (len 3): target[0..2] = "abc"  ≠  "def"  ✗
  "abcd" (len 4): target[0..3] = "abcd" == "abcd" ✓ → dp[0+4] = dp[4] = true

Index:    0      1      2      3      4      5      6
dp:      [ T ]  [ F ]  [ T ]  [ T ]  [ T ]  [ F ]  [ F ]
                         ↑      ↑      ↑
                        "ab"  "abc"  "abcd" matched from position 0

──────────────────────────────────────────────────────────────────

i=1: dp[1] = false → SKIP (can't construct "a" from wordBank — no word is "a")

──────────────────────────────────────────────────────────────────

i=2: dp[2] = true → check each word starting at position 2
  (dp[2]=true means "ab" was constructible, so we can continue from here)
  "ab"   (len 2): target[2..3] = "cd"   ≠  "ab"   ✗
  "abc"  (len 3): target[2..4] = "cde"  ≠  "abc"  ✗
  "cd"   (len 2): target[2..3] = "cd"   == "cd"   ✓ → dp[2+2] = dp[4] = true (already T)
  "def"  (len 3): target[2..4] = "cde"  ≠  "def"  ✗
  "abcd" (len 4): target[2..5] = "cdef" ≠  "abcd" ✗

Index:    0      1      2      3      4      5      6
dp:      [ T ]  [ F ]  [ T ]  [ T ]  [ T ]  [ F ]  [ F ]  (no change)

──────────────────────────────────────────────────────────────────

i=3: dp[3] = true → check each word starting at position 3
  (dp[3]=true means "abc" was constructible, continue from here)
  "ab"   (len 2): target[3..4] = "de"   ≠  "ab"   ✗
  "abc"  (len 3): target[3..5] = "def"  ≠  "abc"  ✗
  "cd"   (len 2): target[3..4] = "de"   ≠  "cd"   ✗
  "def"  (len 3): target[3..5] = "def"  == "def"  ✓ → dp[3+3] = dp[6] = true ← TARGET!
  "abcd" (len 4): 3+4=7 > 6 → OUT OF BOUNDS, skip

Index:    0      1      2      3      4      5      6
dp:      [ T ]  [ F ]  [ T ]  [ T ]  [ T ]  [ F ]  [ T ]
                                                      ↑
                                            dp[6] = true ✅ TARGET REACHED!
                                            "abc" (0→3) + "def" (3→6)

──────────────────────────────────────────────────────────────────

i=4: dp[4] = true → check from position 4
  "ab"   (len 2): target[4..5] = "ef"   ≠  "ab"   ✗
  "abc"  (len 3): 4+3=7 > 6 → skip
  "cd"   (len 2): target[4..5] = "ef"   ≠  "cd"   ✗
  "def"  (len 3): 4+3=7 > 6 → skip
  "abcd" (len 4): 4+4=8 > 6 → skip
  (dp[6] is already true from i=3 — nothing new to set)

i=5: dp[5] = false → SKIP
i=6: loop ends (we've filled the table)

FINAL TABLE:
Index:    0      1      2      3      4      5      6
dp:      [ T ]  [ F ]  [ T ]  [ T ]  [ T ]  [ F ]  [ T ]
                                                      ↑
                                    Answer: dp[6] = true ✅

dp[1] = false: "a" can't be constructed (no word "a" in bank)
dp[5] = false: "abcde" can't be constructed (no way to reach 'e')
dp[6] = true:  "abcdef" = "abc" + "def" ✓
```

### How the Table is Filled — FALSE Case: canConstruct("abcdef", ["ab","abc","cd","abcg"])

```
target   = "abcdef"   (m = 6)
wordBank = ["ab","abc","cd","abcg"]
("def" is missing — last 3 chars can't be matched)

INITIAL:
Index:    0      1      2      3      4      5      6
dp:      [ T ]  [ F ]  [ F ]  [ F ]  [ F ]  [ F ]  [ F ]

i=0: dp[0]=T
  "ab"  (2): target[0..1]="ab"="ab"     ✓ → dp[2]=T
  "abc" (3): target[0..2]="abc"="abc"   ✓ → dp[3]=T
  "cd"  (2): target[0..1]="ab"≠"cd"    ✗
  "abcg"(4): target[0..3]="abcd"≠"abcg" ✗  ('d' ≠ 'g' at position 3)

Index:    0      1      2      3      4      5      6
dp:      [ T ]  [ F ]  [ T ]  [ T ]  [ F ]  [ F ]  [ F ]

i=2: dp[2]=T
  "ab"  (2): target[2..3]="cd"≠"ab"    ✗
  "abc" (3): target[2..4]="cde"≠"abc"  ✗
  "cd"  (2): target[2..3]="cd"="cd"    ✓ → dp[4]=T
  "abcg"(4): target[2..5]="cdef"≠"abcg" ✗

Index:    0      1      2      3      4      5      6
dp:      [ T ]  [ F ]  [ T ]  [ T ]  [ T ]  [ F ]  [ F ]

i=3: dp[3]=T
  "ab"  (2): target[3..4]="de"≠"ab"    ✗
  "abc" (3): target[3..5]="def"≠"abc"  ✗
  "cd"  (2): target[3..4]="de"≠"cd"    ✗
  "abcg"(4): 3+4=7 > 6, skip

NO CHANGE: nothing can bridge from position 3 onward with these words.

i=4: dp[4]=T
  "ab"  (2): target[4..5]="ef"≠"ab"    ✗
  "abc" (3): 4+3=7>6, skip
  "cd"  (2): target[4..5]="ef"≠"cd"    ✗
  "abcg"(4): 4+4=8>6, skip

NO CHANGE.

i=5: dp[5]=F → skip
i=6: loop ends

FINAL TABLE:
Index:    0      1      2      3      4      5      6
dp:      [ T ]  [ F ]  [ T ]  [ T ]  [ T ]  [ F ]  [ F ]
                                                      ↑
                                    Answer: dp[6] = false ✗

dp[5] = false: no word bridges to 'f' from any reachable position
dp[6] = false: can't construct full "abcdef" without "def" in wordBank
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
//   Only the dp boolean array of size m+1.
//   NO suffix strings stored, NO call stack at all.
//   This is BETTER than memoization's O(m²) space!
// ─────────────────────────────────────────────────────────────

bool canConstruct_tabulation(const string& target, const vector<string>& wordBank) {

    int m = target.length();

    // STEP 1: Create dp array of size m+1, all false
    vector<bool> dp(m + 1, false);

    // STEP 2: Base case — empty prefix "" is always constructible
    dp[0] = true;

    // STEP 3: Scan each position in the target
    for (int i = 0; i <= m; i++) {
        if (!dp[i]) continue;    // can't construct target[0..i-1] → skip

        // From position i, try each word
        for (const string& word : wordBank) {
            int wlen = word.length();

            // Check if word fits within remaining target
            if (i + wlen > m) continue;

            // Check if word matches target starting at position i
            if (target.substr(i, wlen) == word) {
                dp[i + wlen] = true;    // we can construct target[0..i+wlen-1]
            }
        }
    }

    // STEP 4: Answer is at index m
    return dp[m];
}

// ─────────────────────────────────────────────────────────────
// BONUS: Print the dp table row-by-row (for learning)
// ─────────────────────────────────────────────────────────────
void printTable(const string& target, const vector<string>& wordBank) {
    int m = target.length();
    vector<bool> dp(m + 1, false);
    dp[0] = true;

    cout << "\nBuilding dp table for canConstruct(\"" << target << "\", [";
    for (int i = 0; i < (int)wordBank.size(); i++) {
        cout << wordBank[i];
        if (i < (int)wordBank.size()-1) cout << ",";
    }
    cout << "]):" << endl;

    cout << "Index: ";
    for (int i = 0; i <= m; i++) cout << "[" << i << "] ";
    cout << endl;

    cout << "Start: ";
    for (int i = 0; i <= m; i++) cout << (dp[i] ? " T   " : " F   ");
    cout << endl;

    for (int i = 0; i <= m; i++) {
        if (!dp[i]) continue;
        bool changed = false;
        for (const string& word : wordBank) {
            int wlen = word.length();
            if (i + wlen <= m && target.substr(i, wlen) == word) {
                if (!dp[i + wlen]) { dp[i + wlen] = true; changed = true; }
            }
        }
        if (changed) {
            cout << "i=" << i << ":   ";
            for (int j = 0; j <= m; j++) cout << (dp[j] ? " T   " : " F   ");
            cout << endl;
        }
    }

    cout << "Answer: dp[" << m << "] = " << (dp[m] ? "true" : "false") << endl;
}

int main() {
    cout << "canConstruct (Tabulation)" << endl;
    cout << "──────────────────────────────────────────────────────" << endl;

    cout << boolalpha;

    cout << "cc(\"abcdef\",    [ab,abc,cd,def,abcd])  = "
         << canConstruct_tabulation("abcdef", {"ab","abc","cd","def","abcd"}) << endl;

    cout << "cc(\"skateboard\",[bo,rd,ate,t,ska,sk,boar]) = "
         << canConstruct_tabulation("skateboard", {"bo","rd","ate","t","ska","sk","boar"}) << endl;

    cout << "cc(\"enterapotentpot\", [a,p,ent,enter,ot,o,t]) = "
         << canConstruct_tabulation("enterapotentpot", {"a","p","ent","enter","ot","o","t"}) << endl;

    cout << "cc(\"\",           [cat,dog])             = "
         << canConstruct_tabulation("", {"cat","dog"}) << endl;

    cout << "cc(\"eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeef\", [e,ee,eee,eeee,eeeee,eeeeee]) = "
         << canConstruct_tabulation("eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeef",
                                    {"e","ee","eee","eeee","eeeee","eeeeee"}) << endl;

    printTable("abcdef", {"ab","abc","cd","def","abcd"});
    printTable("abcdef", {"ab","abc","cd","abcg"});

    return 0;
}
```

**Output:**
```
canConstruct (Tabulation)
──────────────────────────────────────────────────────
cc("abcdef",    [ab,abc,cd,def,abcd])  = true
cc("skateboard",[bo,rd,ate,t,ska,sk,boar]) = false
cc("enterapotentpot", [a,p,ent,enter,ot,o,t]) = true
cc("",           [cat,dog])             = true
cc("eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeef",[e,...]) = false

Building dp table for canConstruct("abcdef", [ab,abc,cd,def,abcd]):
Index: [0] [1] [2] [3] [4] [5] [6]
Start:  T    F    F    F    F    F    F
i=0:    T    F    T    T    T    F    F     ← "ab","abc","abcd" matched
i=2:    T    F    T    T    T    F    F     ← "cd" matched but dp[4] already T
i=3:    T    F    T    T    T    F    T     ← "def" matched → dp[6] set!
Answer: dp[6] = true

Building dp table for canConstruct("abcdef", [ab,abc,cd,abcg]):
Index: [0] [1] [2] [3] [4] [5] [6]
Start:  T    F    F    F    F    F    F
i=0:    T    F    T    T    F    F    F
i=2:    T    F    T    T    T    F    F
Answer: dp[6] = false
```

### Complexity Analysis — Tabulation

```
TIME COMPLEXITY: O(n × m²)
────────────────────────────
Outer loop: i from 0 to m         → m+1 iterations
Inner loop: for each word         → n iterations per outer step
Each inner step:
  target.substr(i, wlen) == word  → O(m) for substr + comparison

Total: (m+1) × n × O(m) = O(n × m²)

Same time complexity as memoization.
BUT: No recursion overhead, no hash map lookups.
→ Tabulation is faster in practice by a constant factor.

SPACE COMPLEXITY: O(m)   ← BETTER than memoization!
────────────────────────────────────────────────────
dp array:        m+1 booleans → O(m)
No suffix strings stored anywhere.
No call stack (no recursion).
No hash map.

COMPARE:
  Memoization: O(m²) — stores suffix strings as keys
  Tabulation:  O(m)  — stores only booleans!

This is the MAIN SPACE ADVANTAGE of tabulation for canConstruct.
For large targets, this difference is significant.
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
// Time: O(n^m × m)    Space: O(m²) — suffix strings on stack
// ════════════════════════════════════════════════════════════
bool cc_recursive(const string& t, const vector<string>& wb) {
    if (t.empty()) return true;
    for (const string& w : wb)
        if (t.length() >= w.length() && t.substr(0, w.length()) == w)
            if (cc_recursive(t.substr(w.length()), wb)) return true;
    return false;
}

// ════════════════════════════════════════════════════════════
// TECHNIQUE 2: MEMOIZATION (Top-Down DP)
// Time: O(n × m²)    Space: O(m²)
// ════════════════════════════════════════════════════════════
bool cc_memo(const string& t, const vector<string>& wb,
             unordered_map<string, bool>& memo) {
    if (memo.count(t)) return memo[t];
    if (t.empty()) return true;
    for (const string& w : wb)
        if (t.length() >= w.length() && t.substr(0, w.length()) == w)
            if (cc_memo(t.substr(w.length()), wb, memo))
                return memo[t] = true;
    return memo[t] = false;
}

// ════════════════════════════════════════════════════════════
// TECHNIQUE 3: TABULATION (Bottom-Up DP)
// Time: O(n × m²)    Space: O(m) — just the dp array!
// ════════════════════════════════════════════════════════════
bool cc_tabulation(const string& t, const vector<string>& wb) {
    int m = t.length();
    vector<bool> dp(m + 1, false);
    dp[0] = true;
    for (int i = 0; i <= m; i++) {
        if (!dp[i]) continue;
        for (const string& w : wb) {
            int wl = w.length();
            if (i + wl <= m && t.substr(i, wl) == w)
                dp[i + wl] = true;
        }
    }
    return dp[m];
}

// ════════════════════════════════════════════════════════════
// MAIN — Compare all techniques
// ════════════════════════════════════════════════════════════
int main() {
    cout << boolalpha;
    cout << "target                 Recursive  Memoized  Tabulation" << endl;
    cout << "──────────────────────────────────────────────────────" << endl;

    struct TC { string target; vector<string> wb; };
    vector<TC> tests = {
        {"abcdef",         {"ab","abc","cd","def","abcd"}},
        {"skateboard",     {"bo","rd","ate","t","ska","sk","boar"}},
        {"enterapotentpot",{"a","p","ent","enter","ot","o","t"}},
        {"",               {"cat","dog"}},
        {"abcdef",         {"ab","abc","cd","abcg"}},
    };

    for (auto& [t, wb] : tests) {
        unordered_map<string, bool> memo;
        cout << "\"" << t.substr(0, 12) << (t.length()>12?"...":"") << "\"";
        cout << string(max(0, 22-(int)min(t.length()+2, (size_t)14)), ' ');
        cout << cc_recursive(t, wb)   << "       "
             << cc_memo(t, wb, memo) << "       "
             << cc_tabulation(t, wb) << endl;
    }

    // Large cases — only memo + tabulation
    cout << "\nLarge targets (memo + tabulation only):" << endl;
    cout << "────────────────────────────────────────────────────" << endl;
    vector<TC> large = {
        {"eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeef", {"e","ee","eee","eeee","eeeee","eeeeee"}},
        {"eeeeeeeeeeeeeeeeeeeeeeeeeeeeeee",  {"e","ee","eee","eeee","eeeee","eeeeee"}},
    };
    for (auto& [t, wb] : large) {
        unordered_map<string, bool> memo;
        cout << "\"" << t.substr(0,10) << "...\" (len=" << t.length() << "):  ";
        cout << "Memo=" << cc_memo(t, wb, memo)
             << "  Tab=" << cc_tabulation(t, wb) << endl;
    }

    return 0;
}
```

**Output:**
```
target                 Recursive  Memoized  Tabulation
──────────────────────────────────────────────────────
"abcdef"               true       true       true
"skateboard"           false      false      false
"enterapotentpot"      true       true       true
""                     true       true       true
"abcdef"               false      false      false

Large targets (memo + tabulation only):
────────────────────────────────────────────────────
"eeeeeeeeee..." (len=32):  Memo=false  Tab=false
"eeeeeeeeee..." (len=31):  Memo=true   Tab=true
```

---

## Complexity Summary — Side by Side

```
┌──────────────────┬─────────────────┬────────────────────────────────────────────┐
│ Technique        │      TIME       │                 SPACE                      │
├──────────────────┼─────────────────┼────────────────────────────────────────────┤
│ Recursion        │  O(n^m × m)     │ O(m²) — suffix strings on call stack       │
│                  │                 │ WARNING: Exponential — hangs on hard cases  │
├──────────────────┼─────────────────┼────────────────────────────────────────────┤
│ Memoization      │  O(n × m²)      │ O(m²) — memo map stores suffix strings      │
│ (Top-Down DP)    │                 │ O(m²) — call stack with suffix strings      │
│                  │                 │ Total: O(m²)                                │
├──────────────────┼─────────────────┼────────────────────────────────────────────┤
│ Tabulation       │  O(n × m²)      │ O(m)  — boolean dp array only!             │
│ (Bottom-Up DP)   │                 │ NO call stack, NO strings stored            │
│                  │                 │ BEST SPACE — same as canSum!               │
└──────────────────┴─────────────────┴────────────────────────────────────────────┘

n = wordBank.size(),  m = target.length()

KEY SPACE OBSERVATION:
  canSum      tabulation: O(m)  memo: O(m)   ← same (integers cached)
  canConstruct tabulation: O(m)  memo: O(m²)  ← tabulation wins! (no strings)
```

---

## Understanding WHY the Complexities Are What They Are

### Why Recursion is O(n^m × m) with O(m²) Space

```
TIME: O(n^m × m)
─────────────────
At each recursive call, we try n words (one per wordBank entry).
Depth of recursion = at most m (each step removes ≥ 1 character from target).
Total nodes in recursion tree: n^m

At EACH node: substr(0, w.length()) creates a new string → O(m) work.
TOTAL: O(n^m) nodes × O(m) substr = O(n^m × m)

SPACE: O(m²)
──────────────
The call stack holds m frames at once.
Each frame stores the current suffix string.
Suffix lengths at each depth: m, m-w1, m-w1-w2, ...
Total strings on stack at maximum depth: sum ≈ m + (m-1) + ... ≈ m²/2 = O(m²)

COMPARE WITH canSum SPACE:
  canSum recursion: O(m) — each frame stores just an integer (4 bytes)
  canConstruct rec: O(m²) — each frame stores a suffix string (up to m bytes)
  → Strings are expensive! This is the STRING TAX.
```

### Why Memo Time is O(n × m²) — The Two Sources of m²

```
SOURCE 1: m unique subproblems × n words = O(n × m)  ← like canSum

SOURCE 2: Each operation costs O(m) for string work   ← the STRING TAX

  target.substr(0, w.length()) == w:
    → Creates a new string of length w.length() → O(m) worst case
    → Character-by-character comparison → O(m) worst case

TOTAL: O(n × m) × O(m) = O(n × m²)

Compare with canSum memo:
  canSum:       O(n × m) — operations are O(1) (integer arithmetic)
  canConstruct: O(n × m²) — operations are O(m) (string operations)

THE STRING TAX: Every string operation (copy, compare, substr)
costs O(m) where m = string length.
This multiplies the canSum complexity by an extra factor of m.
```

### Why Tabulation Space is Only O(m) When Memo is O(m²)

```
MEMOIZATION stores:
  Keys:   suffix strings (up to m chars each × up to m+1 entries = O(m²))
  Values: booleans (trivial)
  Stack:  suffix strings at each depth = O(m²)

TABULATION stores:
  dp array: m+1 booleans = O(m)
  No strings stored anywhere.
  No recursion → no call stack at all.

WHY CAN TABULATION AVOID STORING STRINGS?
  Instead of suffix = "def" (a string object)...
  Tabulation uses index i = 3 (just an integer).
  "def" starts at index 3 in "abcdef" — that IS the same information,
  but stored as a 4-byte integer instead of a 3-byte string object + overhead.

  The comparison target.substr(i, wlen) == word still costs O(m),
  but the result is immediately used and the string is NOT stored.
  → Time is still O(m), but no O(m²) space accumulation!

CONCLUSION: Tabulation's O(m) space vs Memoization's O(m²) space
            is a REAL advantage for large inputs.
```

### The "EEEEEF" Problem — Why Memoization is Critical

```
canConstruct("eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeef", ["e","ee","eee","eeee"])

WITHOUT MEMOIZATION:
  Every 'e' can be matched by "e", "ee", "eee", or "eeee" (4 choices).
  The "f" at the end NEVER matches anything — but recursion finds this
  only after exploring every combination of 'e' splits.
  Number of ways to partition n 'e's: exponential (related to compositions)
  For 31 'e's: over 4^31 ≈ 10^18 calls — takes years to compute!

WITH MEMOIZATION:
  cc("f")   = false → cached
  cc("ef")  = false → cached (trying e→"f" → cache hit: false)
  cc("eef") = false → cached (trying e→"ef" → cache hit, ee→"f" → cache hit)
  cc("eeef") = false → cached
  ...and so on.
  Each new suffix is computed in O(n) time using cached results.
  Total: O(n × m) = O(4 × 32) = 128 operations → instant!

WITH TABULATION:
  dp[31] = false (the 'f' position was never set)
  dp[32] = false (never reachable from any true dp[i])
  Total: 32 × 4 = 128 operations → instant!

BOTH optimized approaches reduce catastrophic exponential to instant.
```

---

## Visualizing the "Reachability" Spread — Tabulation Intuition

```
Think of dp as positions on the TARGET STRING.
dp[i] = "Can I arrive at position i using words from wordBank?"
dp[0] = true  (start at the beginning — always valid).

From each reachable position, if a word matches, we "jump" forward.

canConstruct("abcdef", ["ab","abc","cd","def","abcd"]):

Positions: 0   1   2   3   4   5   6
           ↑   a   b   c   d   e   f
           🟢  ⚫  ⚫  ⚫  ⚫  ⚫  ⚫

From 0, "ab" jumps +2, "abc" jumps +3, "abcd" jumps +4:
           🟢  ⚫  🟢  🟢  🟢  ⚫  ⚫

From 2 ("ab" landed), "cd" jumps +2 → position 4 (already lit):
           🟢  ⚫  🟢  🟢  🟢  ⚫  ⚫  (no change)

From 3 ("abc" landed), "def" jumps +3 → position 6:
           🟢  ⚫  🟢  🟢  🟢  ⚫  🟢  ← position 6 lit! ✅

Position 6 is lit → canConstruct = true ✅
Position 1 never lit → "a" can't be constructed from wordBank
Position 5 never lit → no way to reach position 5 from any reachable position

canConstruct("abcdef", ["ab","abc","cd","abcg"]):
Positions: 0   1   2   3   4   5   6

From 0: "ab"→2, "abc"→3
           🟢  ⚫  🟢  🟢  ⚫  ⚫  ⚫

From 2: "cd"→4
           🟢  ⚫  🟢  🟢  🟢  ⚫  ⚫

From 3,4: no word matches "de..","ef.." → stuck!
           🟢  ⚫  🟢  🟢  🟢  ⚫  ⚫  ← position 6 stays dark → false ✗
```

---

## canConstruct vs canSum — Complete Comparison

```
┌─────────────────────┬──────────────────────────┬──────────────────────────────┐
│                     │         canSum            │       canConstruct           │
├─────────────────────┼──────────────────────────┼──────────────────────────────┤
│ Domain              │ Integer addition          │ String concatenation         │
│ Target              │ Integer (e.g. 7)          │ String (e.g. "abcdef")       │
│ Tools               │ Array of integers         │ Array of strings (wordBank)  │
│ Operation           │ target - num              │ target.substr(word.len())    │
│ Match condition     │ any num ≤ target          │ word matches prefix of target│
│ Base case           │ target == 0               │ target.empty()               │
│ Dead end            │ target < 0                │ no word matches prefix       │
│ Return type         │ bool                      │ bool                         │
│                     │                           │                              │
│ Recursion time      │ O(n^m)                    │ O(n^m × m)   +m for substr   │
│ Recursion space     │ O(m)                      │ O(m²)        strings on stack│
│                     │                           │                              │
│ Memo time           │ O(n × m)                  │ O(n × m²)    +m for strings  │
│ Memo space          │ O(m)                      │ O(m²)        string keys     │
│                     │                           │                              │
│ Tab time            │ O(n × m)                  │ O(n × m²)    +m for substr   │
│ Tab space           │ O(m)                      │ O(m)   ← SAME! booleans only│
├─────────────────────┼──────────────────────────┼──────────────────────────────┤
│ Tabulation code     │ if(dp[i])                 │ if(dp[i])                    │
│ difference          │   dp[i+num]=true;         │   if(target.substr(i,wl)==w) │
│                     │                           │     dp[i+wl]=true;           │
└─────────────────────┴──────────────────────────┴──────────────────────────────┘

THE EXTRA ×m FACTOR IN canConstruct:
  Every string operation (substr, ==) costs O(m) instead of O(1).
  This is the "String Tax" — strings are expensive compared to integers.
  canSum O(n×m) × O(1) per op = O(n×m)
  canConstruct O(n×m) × O(m) per op = O(n×m²)
```

---

## The Interview Answer — What to Say and When

```
INTERVIEWER: "Can you construct target string from words in wordBank?"

STEP 1: Recognize it as canSum with strings
  "This is analogous to canSum, but instead of numbers adding to a target,
   we have strings concatenating to form the target.
   The operation is: check prefix match → recurse on remaining suffix."

STEP 2: Write recursive solution
  "Base case: empty target → return true.
   For each word: if word matches the start of target,
   recurse on the rest. If any recursion returns true, return true."
  → Write ~7 lines of recursive code.

  "Time: O(n^m × m) — exponential, plus O(m) per call for substr.
   Space: O(m²) — suffix strings accumulate on the call stack.
   Fails on hard cases like 'eeeeeeef' — hangs indefinitely."

STEP 3: Identify overlapping subproblems
  "The same suffix (e.g. 'def') can appear from multiple paths.
   Once we know 'def' is constructible, we shouldn't recompute it.
   Cache results using the suffix string as the key."

STEP 4: Add memoization
  "Add unordered_map<string, bool> memo.
   Before computing, check memo. After computing, store in memo.
   Time: O(n × m²). Space: O(m²) — strings stored as keys."

STEP 5: Convert to tabulation
  "Replace suffix strings with integer indices into target.
   dp[i] = can target[0..i-1] be constructed?
   dp[0] = true. For each true dp[i] and each word:
   if word matches at position i, set dp[i+word.length()] = true.
   Time: O(n × m²). Space: O(m) — no strings stored, just booleans!"

  "Key insight: tabulation is better in SPACE for canConstruct —
   O(m) vs O(m²) for memoization, because we don't store suffix strings."
```

---

## Quick Cheat Sheet

```
PROBLEM: canConstruct(target, wordBank)
         Return true if target can be built by concatenating words from wordBank.
         Words can be reused.

ANALOGY: Same as canSum but with strings instead of numbers.
         target - word  becomes  target.substr(word.length())
         target == 0   becomes  target.empty()

─────────────────────────────────────────────────────────────────

RECURSION:
  bool cc(const string& t, const vector<string>& wb) {
    if(t.empty()) return true;
    for(const string& w : wb)
      if(t.length()>=w.length() && t.substr(0,w.length())==w)
        if(cc(t.substr(w.length()), wb)) return true;
    return false;
  }
  Time: O(n^m × m)   Space: O(m²)

─────────────────────────────────────────────────────────────────

MEMOIZATION:
  // Extra param: unordered_map<string,bool>& memo
  if(memo.count(t)) return memo[t];
  if(t.empty()) return true;
  for(const string& w : wb)
    if(t.length()>=w.length() && t.substr(0,w.length())==w)
      if(cc_memo(t.substr(w.length()), wb, memo))
        return memo[t]=true;
  return memo[t]=false;

  Time: O(n×m²)   Space: O(m²)

─────────────────────────────────────────────────────────────────

TABULATION:
  vector<bool> dp(m+1, false);
  dp[0] = true;
  for(int i=0; i<=m; i++) {
    if(!dp[i]) continue;
    for(const string& w : wb) {
      int wl = w.length();
      if(i+wl<=m && target.substr(i,wl)==w)
        dp[i+wl] = true;
    }
  }
  return dp[m];

  Time: O(n×m²)   Space: O(m)  ← BEST SPACE! No strings stored.

─────────────────────────────────────────────────────────────────

ANSWERS:
  cc("abcdef",    [ab,abc,cd,def,abcd]) = true   ("abc"+"def")
  cc("skateboard",[bo,rd,ate,t,ska,...])= false   (no valid partition)
  cc("enterapotentpot",[a,p,ent,...])   = true    ("enter"+"a"+...)
  cc("",          [cat,dog])            = true    (base case: empty!)
  cc("eee...ef",  [e,ee,eee,...])       = false   ('f' unmatchable)

COMPLEXITY:
                    TIME          SPACE
  Recursion:      O(n^m × m)    O(m²)
  Memoization:    O(n × m²)     O(m²)
  Tabulation:     O(n × m²)     O(m)   ← wins on space

STRING TAX: Every string op (substr, ==) costs O(m), not O(1).
  canSum:        O(n×m)  time  (ops are integers: O(1))
  canConstruct:  O(n×m²) time  (ops are strings: O(m))
```

---

*Compile and run: `g++ -std=c++17 -o canconstruct can_construct.cpp && ./canconstruct`*
*(C++17 needed only for structured bindings `auto [t,wb] = ...` in main — the core logic works with C++11/14)*
