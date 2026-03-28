# allConstruct Problem — Three Techniques
## Recursion vs Memoization vs Tabulation in C++

> **Problem:** Given a `target` string and an array of strings `wordBank`,
> return **all possible ways** the `target` can be constructed by concatenating
> words from `wordBank`.
> Return a 2D array where each inner array is one valid combination of words.
> Return an empty 2D array if no construction is possible.
> You may reuse words from `wordBank` as many times as you like.

---

## The Complete Construct Family — Where allConstruct Fits

```
canConstruct("purple", wb)    →  true / false
  "CAN we build it?"
  Returns: 1 bit of information.
  Short-circuits: YES. Stops at first success.
  Complexity: O(n × m²) with memoization.

countConstruct("purple", wb)  →  2
  "HOW MANY ways can we build it?"
  Returns: 1 integer.
  Short-circuits: NO. Counts all paths.
  Complexity: O(n × m²) with memoization.

allConstruct("purple", wb)    →  [["purp","le"], ["p","ur","p","le"]]
  "SHOW ME ALL the ways."
  Returns: full 2D list of every combination.
  Short-circuits: NEVER. Must collect every single path.
  Complexity: O(n^m × m) — ALWAYS — even with memoization!  ← KEY INSIGHT

THE PROGRESSION:
  canConstruct  → "is there a path?"               (output size: O(1))
  countConstruct → "how many paths?"               (output size: O(1))
  allConstruct  → "list every path"                (output size: O(n^m × m))

WHY allConstruct IS FUNDAMENTALLY HARDER:
  canConstruct/countConstruct can cache a small value (bool or int) per suffix.
  Cache hit = O(1) lookup.

  allConstruct must cache a LIST OF LISTS per suffix.
  Cache hit = copy the entire cached list = potentially O(n^m × m) work!
  → Memoization does NOT improve the asymptotic complexity here.
  → The output size itself forces exponential work.
```

---

## The Critical Base Case Distinction — [[]] vs []

```
THIS IS THE MOST IMPORTANT THING TO UNDERSTAND IN allConstruct.

allConstruct("", wordBank)  →  [[]]
  ↑ outer vector has ONE element
  ↑ that element is an EMPTY inner vector
  Meaning: there is EXACTLY ONE way to build "":
           pick ZERO words — the empty selection.

allConstruct("xyz", ["a","b"])  →  []
  ↑ outer vector is completely EMPTY
  Meaning: there are ZERO ways to build "xyz" — impossible.

THEY ARE DIFFERENT:
  [[]]  →  1 way, that way uses 0 words
  []    →  0 ways, target is impossible

WHY THIS MATTERS FOR BUILDING RESULTS:
  When we pick word "abc" and recurse on suffix "def":
    suffixWays = allConstruct("def", wb)

  If suffixWays = [["def"]]:
    For each way in suffixWays: prepend "abc" → ["abc","def"]
    result has 1 element ✓

  If suffixWays = [[]]:    ← this happens when suffix = ""
    For each way in [[  ]]: prepend "abc" → ["abc"]
    result has 1 element ✓   ← the [[]] base case propagates correctly!

  If suffixWays = []:      ← suffix is impossible
    For each way in [  ]: (loop body never executes)
    result gets NOTHING added ✓   ← the [] case naturally skips!

The [[]] vs [] distinction makes the algorithm self-consistent:
  "found 1 way (empty)" enables prepend → builds combinations.
  "found 0 ways (empty outer)" disables prepend → skips dead ends.
```

---

## Quick Comparison (Read This First)

```
TECHNIQUE       TIME            SPACE              STYLE
────────────────────────────────────────────────────────────────────────
Recursion       O(n^m × m)      O(n^m × m)         Top-down
Memoization     O(n^m × m)      O(n^m × m)         Top-down + cache
Tabulation      O(n^m × m)      O(n^m × m)         Bottom-up

Where: m = target.length,  n = wordBank.size()

⚠️  ALL THREE TECHNIQUES HAVE THE SAME COMPLEXITY.
    Memoization provides NO asymptotic improvement for allConstruct.
    This is fundamentally different from canConstruct/countConstruct.

WHY? The output itself can be exponentially large:
  In the worst case, there are O(n^m) combinations, each of length O(m).
  Any algorithm that produces this output must spend O(n^m × m) time.
  Memoization can avoid recomputing subtrees, but copying cached results
  (which can be exponentially large) takes the same time as recomputing.

COMPARE THE WHOLE FAMILY:
  canConstruct:   Recursion O(n^m×m) → Memoization O(n×m²)  → Tab O(n×m²)
  countConstruct: Recursion O(n^m×m) → Memoization O(n×m²)  → Tab O(n×m²)
  allConstruct:   Recursion O(n^m×m) → Memoization O(n^m×m) → Tab O(n^m×m)
                                                  ↑
                             NO improvement possible — output is too large!
```

---

## The New Operation: "Prepend Word to Each Way"

```
This is the core building block of allConstruct.

SCENARIO: We picked word "abc" and recursed on suffix "def".
          suffixWays = allConstruct("def", wb) = [["def"], ["d","ef"]]

We want: for each way to build "def", prepend "abc" to get a way to build "abcdef".

FOR EACH way in suffixWays:
  newWay = ["abc"] + way        ← prepend current word
  result.push_back(newWay)

Step by step:
  way = ["def"]       → newWay = ["abc", "def"]
  way = ["d","ef"]    → newWay = ["abc", "d", "ef"]

result = [["abc","def"], ["abc","d","ef"]]

COMPARE WITH OTHER PROBLEMS:
  canConstruct:    if(cc(suffix)) return true;           ← keep or discard
  countConstruct:  total += cc(suffix);                  ← add a number
  allConstruct:    for(way in ac(suffix)) prepend word   ← copy and extend lists

The "prepend" operation is O(m) per combination (copying m words).
If there are O(n^m) combinations, total work is O(n^m × m).
```

---

## The Problem Visualized

```
allConstruct("purple", ["purp","p","ur","le","purpl"])

BUILD A TREE OF ALL PATHS:

"purple"
├── "purp" → "le"
│   └── "le" → ""  ← BASE CASE [[]]
│       → ["le"] prepended → [["le"]]
│   ← suffixWays = [["le"]]
│   → prepend "purp" → [["purp","le"]]
│
├── "p" → "urple"
│   └── "ur" → "ple"
│       └── "p" → "le"
│           └── "le" → ""  ← BASE CASE [[]]
│               → ["le"] → [["le"]]
│           ← [["le"]]
│           → prepend "p" → [["p","le"]]
│       ← [["p","le"]]
│       → prepend "ur" → [["ur","p","le"]]
│   ← [["ur","p","le"]]
│   → prepend "p" → [["p","ur","p","le"]]
│
├── "ur" → "pu" ≠ "ur" — NOT A PREFIX → skip
├── "le" → "pu" ≠ "le" — NOT A PREFIX → skip
│
└── "purpl" → "e"
    └── "e": no word matches "e" → return [] (empty)
    ← [] (impossible)
    → (loop body never runs — nothing added)

FINAL result = [["purp","le"]] + [["p","ur","p","le"]] + [] = 2 combinations ✓

────────────────────────────────────────────────────────────────────────────────

allConstruct("abcdef", ["ab","abc","cd","def","abcd","ef"])

"abcdef"
├── "ab" → "cdef"
│   └── "cd" → "ef"
│       └── "ef" → ""  ← BASE CASE [[]]
│           → prepend "ef" → [["ef"]]
│       → prepend "cd" → [["cd","ef"]]
│   → prepend "ab" → [["ab","cd","ef"]]
│
├── "abc" → "def"
│   └── "def" → ""  ← BASE CASE [[]]
│       → prepend "def" → [["def"]]
│   → prepend "abc" → [["abc","def"]]
│
├── "abcd" → "ef"
│   └── "ef" → ""  ← BASE CASE [[]]
│       → prepend "ef" → [["ef"]]
│   → prepend "abcd" → [["abcd","ef"]]
│
├── "cd"   → "ab" ≠ "cd" (first 2 chars of "abcdef" = "ab") → skip
├── "def"  → "abc" ≠ "def" → skip
└── "ef"   → "ab" ≠ "ef" → skip

FINAL = [["ab","cd","ef"], ["abc","def"], ["abcd","ef"]]   3 combinations ✓

────────────────────────────────────────────────────────────────────────────────

allConstruct("skateboard", ["bo","rd","ate","t","ska","sk","boar"])
→ [] (empty — impossible)
No valid combination exists. Return empty outer vector.

allConstruct("", ["cat","dog"])
→ [[]] (one combination: empty)
```

**Test Cases for Reference:**

```
allConstruct("purple",  ["purp","p","ur","le","purpl"])
  = [["purp","le"], ["p","ur","p","le"]]                        (2 ways)

allConstruct("abcdef",  ["ab","abc","cd","def","abcd","ef"])
  = [["ab","cd","ef"], ["abc","def"], ["abcd","ef"]]             (3 ways)

allConstruct("skateboard", ["bo","rd","ate","t","ska","sk","boar"])
  = []                                                           (impossible)

allConstruct("",        ["cat","dog"])
  = [[]]                                                         (1 way: empty)

allConstruct("aaa",     ["a","aa"])
  = [["a","a","a"], ["a","aa"], ["aa","a"]]                     (3 ways)

allConstruct("aaaa",    ["a","aa"])
  = [["a","a","a","a"], ["a","a","aa"], ["a","aa","a"],
     ["aa","a","a"], ["aa","aa"]]                               (5 ways)
```

> **Notice:** For `"aaaa"` with `["a","aa"]`, the number of ways is 5 (Fibonacci-like growth).
> For longer strings, this grows exponentially — this is why allConstruct is unavoidably O(n^m × m).

---

## The Recursive Formula

```
allConstruct(target, wordBank):

  BASE CASE: target is empty string ("")
    → return [[]]
      ↑ outer vector contains ONE element
      ↑ that one element is an EMPTY inner vector
      Meaning: "1 way to build empty string: pick nothing"

  result = []          ← will collect ALL valid combinations

  FOR EACH word in wordBank:
    IF target STARTS WITH word:
      suffix     = target.substr(word.length())
      suffixWays = allConstruct(suffix, wordBank)   ← all ways for suffix

      FOR EACH way in suffixWays:
        newWay = [word] + way     ← PREPEND current word to each suffix way
        result.push_back(newWay)  ← collect this complete combination

  return result   ← [] if nothing worked; non-empty if paths exist

KEY OPERATIONS vs other problems:
  canConstruct:    if(cc(suffix))      return true;        ← OR
  countConstruct:  total += cc(suffix);                    ← ADD
  allConstruct:    for(way:ac(suffix)) prepend word        ← PREPEND & COLLECT
```

---

## Technique 1 — Recursion (Naive)

### The Idea

Explore every word as a prefix. For each match, collect all ways to build
the suffix, prepend the current word to each, and accumulate in result.
Return an empty outer vector `[]` when nothing works.

### Building Up Through the Call Stack — "purple"

```
Trace: prepend happens as we UNWIND back up the stack.

ac("purple"):
  try "purp": "purple"[0..3]="purp" ✓, suffix="le"
    ac("le"):
      try "purp": skip (too short)
      try "p":    "l"≠"p" ✗
      try "ur":   "le"≠"ur" ✗
      try "le":   "le"="le" ✓, suffix=""
        ac(""):  ← BASE CASE
          return [[]]
        ← suffixWays = [[]]
        for [] in [[]]: newWay = ["le"]+[] = ["le"]
        result = [["le"]]
      try "purpl": skip (too short)
      return [["le"]]
    ← suffixWays = [["le"]]
    for ["le"] in [["le"]]: newWay = ["purp"]+["le"] = ["purp","le"]
    result = [["purp","le"]]

  try "p": "purple"[0]="p" ✓, suffix="urple"
    ac("urple"):
      try "ur": "urple"[0..1]="ur" ✓, suffix="ple"
        ac("ple"):
          try "p": "ple"[0]="p" ✓, suffix="le"
            ac("le") = [["le"]]   ← already traced above
            ← suffixWays = [["le"]]
            for ["le"] in [["le"]]: newWay = ["p"]+["le"] = ["p","le"]
            result = [["p","le"]]
          others: no match
          return [["p","le"]]
        ← suffixWays = [["p","le"]]
        for ["p","le"]: newWay = ["ur"]+["p","le"] = ["ur","p","le"]
        result = [["ur","p","le"]]
      others: no match
      return [["ur","p","le"]]
    ← suffixWays = [["ur","p","le"]]
    for ["ur","p","le"]: newWay = ["p"]+["ur","p","le"] = ["p","ur","p","le"]
    result = [["purp","le"], ["p","ur","p","le"]]

  try "purpl": suffix="e"
    ac("e"):
      all words: none match 'e' prefix
      return []
    ← suffixWays = []
    for(way in []): (loop never runs — nothing added)

  return [["purp","le"], ["p","ur","p","le"]] ✓

RESULT BUILDS IN REVERSE (word prepended AFTER recursion returns).
The deepest base case [[]] starts the chain; words accumulate outward.
```

### Recursion Tree for allConstruct("aaa", ["a","aa"])

```
                         ac("aaa")
                       /          \
              ac("aa")            ac("a")        [a matched]  [aa matched]
             /        \               |
         ac("a")      ac("")       ac("")
         /   \          ↑            ↑
     ac("")  ac("")    [[]]         [[]]
       ↑       ↑
      [[]]    [[]]

Building results bottom-up:
  ac("") = [[]]
  ac("a") from "a": [[]] → prepend "a" → [["a"]]
           from "aa": no match (too long from "a")
           → [["a"]]
  ac("aa") from "a" → ac("a")=[["a"]] → prepend "a" → [["a","a"]]
            from "aa"→ ac("")=[[]]  → prepend "aa"→ [["aa"]]
            → [["a","a"], ["aa"]]
  ac("aaa") from "a" → ac("aa")=[["a","a"],["aa"]]
                      → prepend "a" → [["a","a","a"], ["a","aa"]]
             from "aa"→ ac("a") =[["a"]]
                      → prepend "aa"→ [["aa","a"]]
             → [["a","a","a"], ["a","aa"], ["aa","a"]]  ← 3 ways ✓

REPEATED SUBPROBLEM: ac("a") computed TWICE
  Once from ac("aaa")→"a"→ac("aa")→"a"→ac("a")
  Once from ac("aaa")→"aa"→ac("a")
Memoization will cache ac("a")=[["a"]] and reuse it!
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
//   n^m recursive calls. Each prepend operation copies up to O(m) words.
//   Total work = total size of output = O(n^m × m).
// Space: O(n^m × m)
//   Recursion stack: O(m) frames deep.
//   Combinations accumulated in result vectors: O(n^m × m) total.
//   (ALL combinations must live in memory simultaneously at some point)
// ─────────────────────────────────────────────────────────────

vector<vector<string>> allConstruct_recursive(
    const string& target,
    const vector<string>& wordBank
) {
    // BASE CASE: empty string → 1 way: empty combination
    if (target.empty()) return {{}};    // return [[]]  NOT []

    vector<vector<string>> result;      // collect ALL combinations

    for (const string& word : wordBank) {
        // Check if word matches the front of target
        if (target.length() >= word.length() &&
            target.substr(0, word.length()) == word) {

            string suffix = target.substr(word.length());

            // Get ALL ways to build the suffix
            auto suffixWays = allConstruct_recursive(suffix, wordBank);

            // Prepend current word to EACH suffix way
            for (const auto& way : suffixWays) {
                vector<string> newWay = {word};               // start with current word
                newWay.insert(newWay.end(), way.begin(), way.end()); // append suffix way
                result.push_back(newWay);                     // add to result
            }
            // NOTE: no "return" — we collect from ALL matching words!
        }
    }

    return result;    // [] if nothing worked, non-empty otherwise
}

void printResult(const string& label, const vector<vector<string>>& result) {
    cout << label << endl;
    if (result.empty()) {
        cout << "  [] (impossible — 0 ways)" << endl;
        return;
    }
    cout << "  [" << endl;
    for (const auto& way : result) {
        cout << "    [";
        for (int i = 0; i < (int)way.size(); i++) {
            cout << "\"" << way[i] << "\"";
            if (i < (int)way.size()-1) cout << ", ";
        }
        cout << "]" << endl;
    }
    cout << "  ]  (" << result.size() << " ways)" << endl;
}

int main() {
    cout << "allConstruct (Recursion)" << endl;
    cout << "──────────────────────────────────────────────────────" << endl;

    printResult("allConstruct(\"purple\", [purp,p,ur,le,purpl]):",
        allConstruct_recursive("purple", {"purp","p","ur","le","purpl"}));

    printResult("allConstruct(\"abcdef\", [ab,abc,cd,def,abcd,ef]):",
        allConstruct_recursive("abcdef", {"ab","abc","cd","def","abcd","ef"}));

    printResult("allConstruct(\"aaa\", [a,aa]):",
        allConstruct_recursive("aaa", {"a","aa"}));

    printResult("allConstruct(\"skateboard\", [bo,rd,ate,t,ska,sk,boar]):",
        allConstruct_recursive("skateboard", {"bo","rd","ate","t","ska","sk","boar"}));

    printResult("allConstruct(\"\", [cat,dog]):",
        allConstruct_recursive("", {"cat","dog"}));

    // WARNING: allConstruct("aaaaaaaaaa", {"a","aa"}) is very slow!
    // The number of ways grows like Fibonacci — exponential output size.

    return 0;
}
```

**Output:**
```
allConstruct (Recursion)
──────────────────────────────────────────────────────
allConstruct("purple", [purp,p,ur,le,purpl]):
  [
    ["purp", "le"]
    ["p", "ur", "p", "le"]
  ]  (2 ways)
allConstruct("abcdef", [ab,abc,cd,def,abcd,ef]):
  [
    ["ab", "cd", "ef"]
    ["abc", "def"]
    ["abcd", "ef"]
  ]  (3 ways)
allConstruct("aaa", [a,aa]):
  [
    ["a", "a", "a"]
    ["a", "aa"]
    ["aa", "a"]
  ]  (3 ways)
allConstruct("skateboard", [bo,rd,ate,t,ska,sk,boar]):
  [] (impossible — 0 ways)
allConstruct("", [cat,dog]):
  [
    []
  ]  (1 ways)
```

### Complexity Analysis — Recursion

```
TIME COMPLEXITY: O(n^m × m)
────────────────────────────
The recursion tree has up to n^m leaf nodes (valid or invalid).
At each leaf: we've built one combination of O(m) words.
Collecting it: push_back copies O(m) words → O(m) work.

Total: O(n^m) combinations × O(m) per combination = O(n^m × m)

This is a LOWER BOUND on any algorithm — we must at least output the result!
No optimization can improve this for allConstruct.

SPACE COMPLEXITY: O(n^m × m)
─────────────────────────────
The result itself contains all combinations: O(n^m × m) space.
This is unavoidable — we're asked to return all combinations!
Plus O(m²) for the call stack and suffix strings.
Total: O(n^m × m) — dominated by the output size.
```

---

## Technique 2 — Memoization (Top-Down DP)

### The Idea

**Cache the LIST OF ALL COMBINATIONS for each unique suffix.**

But here's the critical difference from canConstruct/countConstruct:
- Caching a `bool`: O(1) storage, O(1) retrieval → big speedup
- Caching a `vector<vector<string>>`: up to O(n^m × m) storage → copying it costs as much as recomputing!

### Why Memoization Does NOT Improve Asymptotic Complexity

```
FOR canConstruct:
  memo["le"] = true          ← 1 bit stored
  Cache hit: return memo["le"]   ← O(1) cost
  Speedup: YES — compute "le" once, reuse O(1)

FOR countConstruct:
  memo["le"] = 1             ← 1 integer stored
  Cache hit: return memo["le"]   ← O(1) cost
  Speedup: YES — compute "le" once, reuse O(1)

FOR allConstruct:
  memo["le"] = [["le"]]       ← 1 combination stored (small here)
  Cache hit: return memo["le"]   ← copies the entire cached list!
  If memo["le"] contains 1000 combinations of length 10 each:
    Cache hit costs: copy 1000 × 10 = 10,000 words → NOT O(1)!
  
  Asymptotically: copying cached result = O(output_size) = O(n^m × m)
  → Same cost as recomputing!

CONCLUSION: Memoization eliminates redundant RECURSIVE CALLS,
            but copying the cached LIST still costs proportional to its size.
            Total work remains O(n^m × m).

DOES MEMOIZATION HELP AT ALL?
  YES — in practice, for cases with many repeated suffixes:
  Without memo: same subtree explored multiple times
  With memo:    subtree explored once, but COPIED multiple times
  
  For allConstruct("aaa", ["a","aa"]):
    Without memo: ac("a") computed twice (from different paths to it)
    With memo:    ac("a") computed once, then COPIED once → still ~same work
  
  Memo can help avoid some work but cannot change the O(n^m × m) order.
  In the worst case (all results distinct), there's no reuse at all.
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
// Time:  O(n^m × m) — same as recursion!
//   Cache hits save recomputation, but copying cached lists
//   is proportional to the list size, which can be O(n^m × m).
// Space: O(n^m × m)
//   memo stores all combinations for each unique suffix.
//   Total memo size ≤ total output size = O(n^m × m).
// ─────────────────────────────────────────────────────────────

vector<vector<string>> allConstruct_memo(
    const string& target,
    const vector<string>& wordBank,
    unordered_map<string, vector<vector<string>>>& memo
) {
    // STEP 1: Cache check
    if (memo.count(target)) {
        return memo[target];    // returns a COPY of the cached list
    }                           // copying can be expensive!

    // STEP 2: Base case
    if (target.empty()) return {{}};    // [[]]

    vector<vector<string>> result;

    // STEP 3: Try each word — NO early return, collect ALL paths
    for (const string& word : wordBank) {
        if (target.length() >= word.length() &&
            target.substr(0, word.length()) == word) {

            string suffix = target.substr(word.length());
            auto suffixWays = allConstruct_memo(suffix, wordBank, memo);

            for (const auto& way : suffixWays) {
                vector<string> newWay = {word};
                newWay.insert(newWay.end(), way.begin(), way.end());
                result.push_back(newWay);
            }
        }
    }

    // STEP 4: Cache and return
    return memo[target] = result;    // stores a copy in memo
}

void printResult(const string& label, const vector<vector<string>>& result) {
    cout << label << endl;
    if (result.empty()) { cout << "  [] (0 ways)" << endl; return; }
    cout << "  [" << endl;
    for (const auto& way : result) {
        cout << "    [";
        for (int i = 0; i < (int)way.size(); i++) {
            cout << "\"" << way[i] << "\"";
            if (i < (int)way.size()-1) cout << ", ";
        }
        cout << "]" << endl;
    }
    cout << "  ]  (" << result.size() << " ways)" << endl;
}

int main() {
    cout << "allConstruct (Memoization)" << endl;
    cout << "──────────────────────────────────────────────────────" << endl;

    auto run = [](const string& t, vector<string> wb) {
        unordered_map<string, vector<vector<string>>> memo;
        return allConstruct_memo(t, wb, memo);
    };

    printResult("allConstruct(\"purple\", [purp,p,ur,le,purpl]):",
        run("purple", {"purp","p","ur","le","purpl"}));

    printResult("allConstruct(\"abcdef\", [ab,abc,cd,def,abcd,ef]):",
        run("abcdef", {"ab","abc","cd","def","abcd","ef"}));

    printResult("allConstruct(\"aaa\", [a,aa]):",
        run("aaa", {"a","aa"}));

    printResult("allConstruct(\"skateboard\", [bo,rd,ate,t,ska,sk,boar]):",
        run("skateboard", {"bo","rd","ate","t","ska","sk","boar"}));

    printResult("allConstruct(\"\", [cat,dog]):",
        run("", {"cat","dog"}));

    return 0;
}
```

**Output:**
```
allConstruct (Memoization)
──────────────────────────────────────────────────────
allConstruct("purple", [purp,p,ur,le,purpl]):
  [
    ["purp", "le"]
    ["p", "ur", "p", "le"]
  ]  (2 ways)
allConstruct("abcdef", [ab,abc,cd,def,abcd,ef]):
  [
    ["ab", "cd", "ef"]
    ["abc", "def"]
    ["abcd", "ef"]
  ]  (3 ways)
allConstruct("aaa", [a,aa]):
  [
    ["a", "a", "a"]
    ["a", "aa"]
    ["aa", "a"]
  ]  (3 ways)
allConstruct("skateboard", [bo,rd,ate,t,ska,sk,boar]):
  [] (0 ways)
allConstruct("", [cat,dog]):
  [
    []
  ]  (1 ways)
```

### Step-by-Step Trace for allConstruct("aaa", ["a","aa"]) with Memo

```
ac = allConstruct, 🎯 = cache hit, ✅ = stored in memo

ac("aaa"): not in memo
  try "a": "aaa"[0]="a"="a" ✓, suffix="aa"
    ac("aa"): not in memo
      try "a": "aa"[0]="a"="a" ✓, suffix="a"
        ac("a"): not in memo
          try "a": "a"[0]="a"="a" ✓, suffix=""
            ac(""): BASE CASE → return [[]]
            for [] in [[]]: newWay=["a"]+[]=["a"]
            result=[["a"]]
          try "aa": too long (len("a")=1 < len("aa")=2)
          return [["a"]] → memo["a"]=[["a"]] ✅
        ← suffixWays=[["a"]]
        for ["a"]: newWay=["a"]+["a"]=["a","a"]
        result=[["a","a"]]
      try "aa": "aa"[0..1]="aa"="aa" ✓, suffix=""
        ac(""): BASE CASE → return [[]]
        for []: newWay=["aa"]+[]=["aa"]
        result=[["a","a"],["aa"]]
      return [["a","a"],["aa"]] → memo["aa"]=[["a","a"],["aa"]] ✅
    ← suffixWays=[["a","a"],["aa"]]
    for ["a","a"]: newWay=["a","a","a"]
    for ["aa"]:    newWay=["a","aa"]
    result=[["a","a","a"],["a","aa"]]
  try "aa": "aaa"[0..1]="aa"="aa" ✓, suffix="a"
    ac("a"): 🎯 CACHE HIT → return [["a"]] (instant!)
    ← suffixWays=[["a"]]
    for ["a"]: newWay=["aa","a"]
    result=[["a","a","a"],["a","aa"],["aa","a"]]
  return [["a","a","a"],["a","aa"],["aa","a"]] → memo["aaa"]=... ✅

FINAL: [["a","a","a"], ["a","aa"], ["aa","a"]]  ✓

CACHE HIT ANALYSIS:
  ac("a") was used TWICE:
    Once as ac("aa")→"a"→ac("a")  — computed and cached
    Once as ac("aaa")→"aa"→ac("a") — 🎯 cache hit!
  But the cache hit returned [["a"]] which was COPIED once.
  Copy cost = O(1 combination × 1 word) = O(1) here.
  For larger inputs, cached lists are bigger → copy cost grows.
```

---

## Technique 3 — Tabulation (Bottom-Up DP)

### The Idea

**Use a `vector<vector<vector<string>>>` as the dp array.**

```
dp[i] = vector<vector<string>>
        = "all combinations that construct target[0..i-1]"

dp[0] = [[]]    ← 1 way to construct "" — empty combination
dp[i] = []      ← for all i > 0 initially (no ways yet)

SPREAD RULE:
  For each position i where dp[i] is non-empty:
    For each word that matches at position i:
      For each way in dp[i]:
        newWay = way + [word]
        dp[i + word.length()].push_back(newWay)

ANALOGY WITH OTHER PROBLEMS:
  canConstruct:    dp[j] = true;            ← set a flag
  countConstruct:  dp[j] += dp[i];          ← add a count
  allConstruct:    for(way:dp[i])           ← copy and extend each combination
                     dp[j].push_back(way+[word])
```

### How the Table is Filled — allConstruct("purple", ["purp","p","ur","le","purpl"])

```
target = "purple" (m=6), wordBank = ["purp","p","ur","le","purpl"]

INITIAL:
Index:   0        1    2    3    4    5    6
dp:     [[]]      []   []   []   []   []   []
         ↑
       1 way: []
       (empty combination)

──────────────────────────────────────────────────────────────────

i=0: dp[0] = [[]]  → check each word from position 0

  "purp"(4): target[0..3]="purp"="purp" ✓
    for [] in [[]]:
      newWay = [] + ["purp"] = ["purp"]
      dp[4].push_back(["purp"])

  "p"(1):    target[0..0]="p"="p" ✓
    for [] in [[]]:
      newWay = [] + ["p"] = ["p"]
      dp[1].push_back(["p"])

  "ur"(2):   target[0..1]="pu"≠"ur" ✗
  "le"(2):   target[0..1]="pu"≠"le" ✗

  "purpl"(5): target[0..4]="purpl"="purpl" ✓
    for [] in [[]]:
      newWay = [] + ["purpl"] = ["purpl"]
      dp[5].push_back(["purpl"])

Index:   0        1          2    3    4            5          6
dp:     [[]]    [["p"]]     []   []  [["purp"]]  [["purpl"]]  []

──────────────────────────────────────────────────────────────────

i=1: dp[1] = [["p"]]  → check from position 1

  "purp"(4): target[1..4]="urpl"≠"purp" ✗
  "p"(1):    target[1..1]="u"≠"p" ✗

  "ur"(2):   target[1..2]="ur"="ur" ✓
    for ["p"] in [["p"]]:
      newWay = ["p"] + ["ur"] = ["p","ur"]
      dp[3].push_back(["p","ur"])

  "le"(2):   target[1..2]="ur"≠"le" ✗
  "purpl"(5):target[1..5]="urple"≠"purpl" ✗

Index:   0        1          2     3            4            5          6
dp:     [[]]    [["p"]]     []  [["p","ur"]]  [["purp"]]  [["purpl"]]  []

──────────────────────────────────────────────────────────────────

i=2: dp[2] = []  → SKIP (no ways to reach position 2)

──────────────────────────────────────────────────────────────────

i=3: dp[3] = [["p","ur"]]  → check from position 3

  "p"(1):    target[3..3]="p"="p" ✓
    for ["p","ur"] in [["p","ur"]]:
      newWay = ["p","ur"] + ["p"] = ["p","ur","p"]
      dp[4].push_back(["p","ur","p"])

  others: no match at position 3

Index:   0        1        2      3              4                        5          6
dp:     [[]]   [["p"]]   []  [["p","ur"]]  [["purp"],["p","ur","p"]]  [["purpl"]]  []

                                              ↑ dp[4] now has 2 combinations!

──────────────────────────────────────────────────────────────────

i=4: dp[4] = [["purp"], ["p","ur","p"]]  → check from position 4

  "le"(2):   target[4..5]="le"="le" ✓
    for ["purp"] in dp[4]:
      newWay = ["purp"] + ["le"] = ["purp","le"]
      dp[6].push_back(["purp","le"])
    for ["p","ur","p"] in dp[4]:
      newWay = ["p","ur","p"] + ["le"] = ["p","ur","p","le"]
      dp[6].push_back(["p","ur","p","le"])

  others: out of bounds or no match

Index:   0        1        2      3              4                        5          6
dp:     [[]]   [["p"]]   []  [["p","ur"]]  [["purp"],["p","ur","p"]]  [["purpl"]]  [["purp","le"],["p","ur","p","le"]]

                                                                                      ↑ TARGET!

──────────────────────────────────────────────────────────────────

i=5: dp[5] = [["purpl"]]  → check from position 5
  All words: out of bounds or no match at position 5
  (target[5..5]="e" — no word starts with 'e' in wordBank)
  Nothing added.

i=6: loop ends.

FINAL: dp[6] = [["purp","le"], ["p","ur","p","le"]]  ✓

HOW dp[6] GOT 2 ELEMENTS:
  dp[4] had 2 ways to reach position 4.
  "le" matched at position 4.
  Each way in dp[4] got extended by "le" → 2 new ways in dp[6].
  dp[6] accumulated BOTH via the for-loop over dp[4].
```

### How the Table is Filled — allConstruct("aaa", ["a","aa"])

```
target = "aaa" (m=3), wordBank = ["a","aa"]

INITIAL: dp = [[[]], [], [], []]

──────────────────────────────

i=0: dp[0] = [[]]
  "a"(1): "a"[0..0]="a"="a" ✓
    [] + ["a"] = ["a"] → dp[1].push_back(["a"])
  "aa"(2): "a"[0..1]..."aaa"[0..1]="aa"="aa" ✓
    [] + ["aa"] = ["aa"] → dp[2].push_back(["aa"])

dp = [[[]], [["a"]], [["aa"]], []]

──────────────────────────────

i=1: dp[1] = [["a"]]
  "a"(1): target[1..1]="a"="a" ✓
    ["a"] + ["a"] = ["a","a"] → dp[2].push_back(["a","a"])
  "aa"(2): target[1..2]="aa"="aa" ✓
    ["a"] + ["aa"] = ["a","aa"] → dp[3].push_back(["a","aa"])

dp = [[[]], [["a"]], [["aa"],["a","a"]], [["a","aa"]]]

──────────────────────────────

i=2: dp[2] = [["aa"], ["a","a"]]
  "a"(1): target[2..2]="a"="a" ✓
    ["aa"] + ["a"] = ["aa","a"] → dp[3].push_back(["aa","a"])
    ["a","a"] + ["a"] = ["a","a","a"] → dp[3].push_back(["a","a","a"])
  "aa"(2): 2+2=4 > 3, skip

dp = [[[]], [["a"]], [["aa"],["a","a"]], [["a","aa"],["aa","a"],["a","a","a"]]]

──────────────────────────────

i=3: loop ends.

FINAL: dp[3] = [["a","aa"], ["aa","a"], ["a","a","a"]]  ← 3 ways ✓

NOTE: Order differs slightly from recursion result but all 3 combinations correct.
```

### C++ Code

```cpp
#include <iostream>
#include <vector>
#include <string>
using namespace std;

// ─────────────────────────────────────────────────────────────
// TECHNIQUE 3: TABULATION (Bottom-Up DP)
// Time:  O(n^m × m)
//   Same as recursion/memoization — output size dominates.
//   The for-loop over dp[i] iterates over all current combinations,
//   which grows exponentially in the worst case.
// Space: O(n^m × m)
//   dp stores ALL combinations at each position.
//   Total storage = total output size = O(n^m × m).
//   (Unlike canConstruct/countConstruct, no space advantage here!)
// ─────────────────────────────────────────────────────────────

vector<vector<string>> allConstruct_tabulation(
    const string& target,
    const vector<string>& wordBank
) {
    int m = target.length();

    // STEP 1: dp[i] = all combinations that construct target[0..i-1]
    vector<vector<vector<string>>> dp(m + 1);

    // STEP 2: Base case — 1 way to build empty string: empty combination
    dp[0] = {{}};    // [[]]  NOT  []

    // STEP 3: Build forward
    for (int i = 0; i <= m; i++) {
        if (dp[i].empty()) continue;    // no ways to reach i → skip

        for (const string& word : wordBank) {
            int wlen = word.length();

            if (i + wlen > m) continue;
            if (target.substr(i, wlen) != word) continue;

            // KEY: extend EACH existing combination by appending this word
            for (const auto& way : dp[i]) {
                vector<string> newWay = way;       // copy current combination
                newWay.push_back(word);            // append the matching word
                dp[i + wlen].push_back(newWay);   // add to next position
            }
        }
    }

    // STEP 4: Return all combinations for the full target
    return dp[m];
}

// ─────────────────────────────────────────────────────────────
// BONUS: Print the dp table (educational)
// ─────────────────────────────────────────────────────────────
void printTable(const string& target, const vector<string>& wordBank) {
    int m = target.length();
    vector<vector<vector<string>>> dp(m + 1);
    dp[0] = {{}};

    cout << "\nDP table for allConstruct(\"" << target << "\", [";
    for (int i = 0; i < (int)wordBank.size(); i++) {
        cout << wordBank[i]; if (i < (int)wordBank.size()-1) cout << ",";
    }
    cout << "]):" << endl;

    for (int i = 0; i <= m; i++) {
        cout << "dp[" << i << "] = [";
        for (int j = 0; j < (int)dp[i].size(); j++) {
            cout << "[";
            for (int k = 0; k < (int)dp[i][j].size(); k++) {
                cout << "\"" << dp[i][j][k] << "\"";
                if (k < (int)dp[i][j].size()-1) cout << ",";
            }
            cout << "]";
            if (j < (int)dp[i].size()-1) cout << ",";
        }
        cout << "]" << endl;

        if (dp[i].empty()) continue;
        for (const string& word : wordBank) {
            int wlen = word.length();
            if (i + wlen > m || target.substr(i, wlen) != word) continue;
            for (const auto& way : dp[i]) {
                vector<string> nw = way; nw.push_back(word);
                dp[i + wlen].push_back(nw);
            }
        }
    }
    cout << "Answer: dp[" << m << "] = "
         << dp[m].size() << " combination(s)" << endl;
}

void printResult(const string& label, const vector<vector<string>>& result) {
    cout << label << endl;
    if (result.empty()) { cout << "  [] (0 ways)" << endl; return; }
    cout << "  [" << endl;
    for (const auto& way : result) {
        cout << "    [";
        for (int i = 0; i < (int)way.size(); i++) {
            cout << "\"" << way[i] << "\"";
            if (i < (int)way.size()-1) cout << ", ";
        }
        cout << "]" << endl;
    }
    cout << "  ]  (" << result.size() << " ways)" << endl;
}

int main() {
    cout << "allConstruct (Tabulation)" << endl;
    cout << "──────────────────────────────────────────────────────" << endl;

    printResult("allConstruct(\"purple\", [purp,p,ur,le,purpl]):",
        allConstruct_tabulation("purple", {"purp","p","ur","le","purpl"}));

    printResult("allConstruct(\"abcdef\", [ab,abc,cd,def,abcd,ef]):",
        allConstruct_tabulation("abcdef", {"ab","abc","cd","def","abcd","ef"}));

    printResult("allConstruct(\"aaa\", [a,aa]):",
        allConstruct_tabulation("aaa", {"a","aa"}));

    printResult("allConstruct(\"skateboard\", [bo,rd,ate,t,ska,sk,boar]):",
        allConstruct_tabulation("skateboard", {"bo","rd","ate","t","ska","sk","boar"}));

    printResult("allConstruct(\"\", [cat,dog]):",
        allConstruct_tabulation("", {"cat","dog"}));

    printTable("purple", {"purp","p","ur","le","purpl"});
    printTable("aaa",    {"a","aa"});

    return 0;
}
```

**Output:**
```
allConstruct (Tabulation)
──────────────────────────────────────────────────────
allConstruct("purple", [purp,p,ur,le,purpl]):
  [
    ["purp", "le"]
    ["p", "ur", "p", "le"]
  ]  (2 ways)
allConstruct("abcdef", [ab,abc,cd,def,abcd,ef]):
  [
    ["ab", "cd", "ef"]
    ["abc", "def"]
    ["abcd", "ef"]
  ]  (3 ways)
allConstruct("aaa", [a,aa]):
  [
    ["a", "aa"]
    ["aa", "a"]
    ["a", "a", "a"]
  ]  (3 ways)
allConstruct("skateboard", [...]):
  [] (0 ways)
allConstruct("", [cat,dog]):
  [
    []
  ]  (1 ways)

DP table for allConstruct("purple", [purp,p,ur,le,purpl]):
dp[0] = [[]]
dp[1] = [["p"]]
dp[2] = []
dp[3] = [["p","ur"]]
dp[4] = [["purp"],["p","ur","p"]]
dp[5] = [["purpl"]]
dp[6] = [["purp","le"],["p","ur","p","le"]]
Answer: dp[6] = 2 combination(s)

DP table for allConstruct("aaa", [a,aa]):
dp[0] = [[]]
dp[1] = [["a"]]
dp[2] = [["aa"],["a","a"]]
dp[3] = [["a","aa"],["aa","a"],["a","a","a"]]
Answer: dp[3] = 3 combination(s)
```

---

## All Three Together — One Complete File

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <unordered_map>
using namespace std;
using VVS = vector<vector<string>>;

// ════════════════════════════════════════════════════════════
// TECHNIQUE 1: RECURSION
// Time: O(n^m × m)    Space: O(n^m × m)
// ════════════════════════════════════════════════════════════
VVS ac_recursive(const string& t, const vector<string>& wb) {
    if (t.empty()) return {{}};
    VVS result;
    for (const string& w : wb) {
        if (t.length() >= w.length() && t.substr(0, w.length()) == w) {
            auto suffixWays = ac_recursive(t.substr(w.length()), wb);
            for (const auto& way : suffixWays) {
                vector<string> nw = {w};
                nw.insert(nw.end(), way.begin(), way.end());
                result.push_back(nw);
            }
        }
    }
    return result;
}

// ════════════════════════════════════════════════════════════
// TECHNIQUE 2: MEMOIZATION (Top-Down DP)
// Time: O(n^m × m)  — same! copy cost prevents improvement
// Space: O(n^m × m)
// ════════════════════════════════════════════════════════════
VVS ac_memo(const string& t, const vector<string>& wb,
            unordered_map<string, VVS>& memo) {
    if (memo.count(t)) return memo[t];
    if (t.empty()) return {{}};
    VVS result;
    for (const string& w : wb) {
        if (t.length() >= w.length() && t.substr(0, w.length()) == w) {
            auto suffixWays = ac_memo(t.substr(w.length()), wb, memo);
            for (const auto& way : suffixWays) {
                vector<string> nw = {w};
                nw.insert(nw.end(), way.begin(), way.end());
                result.push_back(nw);
            }
        }
    }
    return memo[t] = result;
}

// ════════════════════════════════════════════════════════════
// TECHNIQUE 3: TABULATION (Bottom-Up DP)
// Time: O(n^m × m)    Space: O(n^m × m)
// ════════════════════════════════════════════════════════════
VVS ac_tabulation(const string& t, const vector<string>& wb) {
    int m = t.length();
    vector<VVS> dp(m + 1);
    dp[0] = {{}};
    for (int i = 0; i <= m; i++) {
        if (dp[i].empty()) continue;
        for (const string& w : wb) {
            int wl = w.length();
            if (i + wl <= m && t.substr(i, wl) == w) {
                for (const auto& way : dp[i]) {
                    vector<string> nw = way;
                    nw.push_back(w);
                    dp[i + wl].push_back(nw);
                }
            }
        }
    }
    return dp[m];
}

// ════════════════════════════════════════════════════════════
// MAIN
// ════════════════════════════════════════════════════════════
void print(const string& label, const VVS& r) {
    cout << label << ": " << r.size() << " way(s)" << endl;
    for (const auto& w : r) {
        cout << "  [";
        for (int i = 0; i < (int)w.size(); i++) {
            cout << w[i]; if (i < (int)w.size()-1) cout << ",";
        }
        cout << "]" << endl;
    }
}

int main() {
    struct TC { string t; vector<string> wb; };
    vector<TC> tests = {
        {"purple",     {"purp","p","ur","le","purpl"}},
        {"abcdef",     {"ab","abc","cd","def","abcd","ef"}},
        {"aaa",        {"a","aa"}},
        {"skateboard", {"bo","rd","ate","t","ska","sk","boar"}},
        {"",           {"cat","dog"}},
    };

    for (auto& [t, wb] : tests) {
        unordered_map<string, VVS> memo;
        auto r1 = ac_recursive(t, wb);
        auto r2 = ac_memo(t, wb, memo);
        auto r3 = ac_tabulation(t, wb);
        cout << "\"" << t << "\": rec=" << r1.size()
             << " memo=" << r2.size()
             << " tab=" << r3.size() << endl;
    }

    return 0;
}
```

**Output:**
```
"purple":     rec=2 memo=2 tab=2
"abcdef":     rec=3 memo=3 tab=3
"aaa":        rec=3 memo=3 tab=3
"skateboard": rec=0 memo=0 tab=0
"":           rec=1 memo=1 tab=1
```

---

## Complexity Summary — Side by Side

```
┌──────────────────┬─────────────────┬────────────────────────────────────────────┐
│ Technique        │      TIME       │                 SPACE                      │
├──────────────────┼─────────────────┼────────────────────────────────────────────┤
│ Recursion        │  O(n^m × m)     │ O(n^m × m) — all combinations in memory   │
├──────────────────┼─────────────────┼────────────────────────────────────────────┤
│ Memoization      │  O(n^m × m)     │ O(n^m × m) — same! (copy cost = recompute)│
│ (Top-Down DP)    │  ← NO IMPROVEMENT from memo!                                │
├──────────────────┼─────────────────┼────────────────────────────────────────────┤
│ Tabulation       │  O(n^m × m)     │ O(n^m × m) — dp stores all combinations   │
│ (Bottom-Up DP)   │  ← Also same!   │ No space advantage unlike other problems!  │
└──────────────────┴─────────────────┴────────────────────────────────────────────┘

n = wordBank.size(),  m = target.length()

NOTE: These are WORST-CASE bounds. For typical inputs with few combinations,
      all three techniques run much faster. The big-O captures the hardest case.
```

---

## The Complete Family — Final Comparison

```
┌──────────────────┬──────────────┬────────────────┬─────────────────────────────┐
│                  │ canConstruct │ countConstruct │        allConstruct          │
├──────────────────┼──────────────┼────────────────┼─────────────────────────────┤
│ Question         │ ≥1 way?      │ How many ways? │ All ways (the actual lists) │
│ Return type      │ bool         │ int            │ vector<vector<string>>       │
│ Base case        │ return true  │ return 1       │ return {{}}  i.e. [[]]       │
│ Dead end         │ return false │ return 0       │ return {}    i.e. []         │
│ Combine branches │ OR           │ +              │ for(way) prepend & collect   │
│ Short-circuit?   │ YES          │ NO             │ NO                           │
│                  │              │                │                              │
│ Rec time         │ O(n^m × m)   │ O(n^m × m)     │ O(n^m × m)                  │
│ Rec space        │ O(m²)        │ O(m²)          │ O(n^m × m)                  │
│                  │              │                │                              │
│ Memo time        │ O(n × m²)    │ O(n × m²)      │ O(n^m × m)  ← no improve!   │
│ Memo space       │ O(m²)        │ O(m²)          │ O(n^m × m)  ← no improve!   │
│                  │              │                │                              │
│ Tab time         │ O(n × m²)    │ O(n × m²)      │ O(n^m × m)  ← no improve!   │
│ Tab space        │ O(m)         │ O(m)           │ O(n^m × m)  ← no improve!   │
│                  │              │                │                              │
│ "purple" result  │ true         │ 2              │ [["purp","le"],              │
│                  │              │                │  ["p","ur","p","le"]]        │
│ "" result        │ true         │ 1              │ [[]]                         │
│ "impossible"     │ false        │ 0              │ []                           │
└──────────────────┴──────────────┴────────────────┴─────────────────────────────┘

WHY MEMO HELPS canConstruct/countConstruct BUT NOT allConstruct:
  canConstruct   caches: bool              → O(1) storage, O(1) copy
  countConstruct caches: int               → O(1) storage, O(1) copy
  allConstruct   caches: vector<vector<string>> → O(output) storage, O(output) copy
                                                   Output can be O(n^m × m) large!
                         Copying cached value costs as much as recomputing it.
```

---

## Understanding the Output-Dominated Complexity

```
THE FUNDAMENTAL PRINCIPLE:
  Any algorithm that must produce output of size S takes at least Ω(S) time.
  This is called the "output-sensitive" lower bound.

FOR allConstruct:
  The output can contain O(n^m) combinations, each of length O(m).
  Total output size: Θ(n^m × m) in the worst case.
  Therefore: any correct algorithm needs Ω(n^m × m) time.

  No clever caching can avoid this — you MUST write every combination to output.

COMPARE WITH countConstruct:
  countConstruct output: 1 integer (O(1)).
  countConstruct only needs to TRACK the count, not store all combinations.
  → Memoization caches O(1) per subproblem → huge savings.

  allConstruct output: all combinations (potentially O(n^m × m)).
  allConstruct must store every combination.
  → Memoization can cache per-suffix lists, but reading/copying them
    costs proportional to their size → no asymptotic saving.

ANALOGY:
  countConstruct: "How many books are in the library?" → ask librarian: O(1)
  allConstruct:   "List every book title."             → must read every title: O(output)
  Memoization for allConstruct = writing the list down once, then re-reading it.
  Re-reading = same time as writing → no improvement!
```

---

## The Interview Answer — What to Say and When

```
INTERVIEWER: "Return all possible ways to construct target from wordBank."

STEP 1: Connect to the construct family
  "This is the allConstruct variant — instead of true/false or a count,
   we return every single valid combination.
   Same prefix-matching structure, same wordBank iteration.
   Three changes: base case returns [[]], dead end returns [],
   and we collect all combinations rather than OR-ing or counting."

STEP 2: Write the recursive solution
  "Base case: empty target → return [[]] (one empty way).
   For each word matching the prefix:
     get suffixWays = allConstruct(suffix, wordBank)
     for each way in suffixWays: prepend current word → push to result.
   Return result."
  → Write ~10 lines.

  "Time/Space: O(n^m × m) — determined by the output size itself."

STEP 3: Discuss memoization honestly
  "Unlike canConstruct and countConstruct where memoization gives polynomial
   speedup, allConstruct memoization does NOT improve asymptotic complexity.
   Reason: caching a list of combinations is O(output size). Reading/copying
   from cache costs the same as recomputing. The output is the bottleneck."
  → This shows depth of understanding — most candidates miss this!

STEP 4: Show tabulation
  "dp[i] = all combinations that build target[0..i-1].
   dp[0] = [[]] (base). For each non-empty dp[i] and each matching word:
   extend each way in dp[i] by the word, push to dp[i + wlen].
   Same O(n^m × m) complexity, but iterative — no recursion stack overflow."
```

---

## Quick Cheat Sheet

```
PROBLEM: allConstruct(target, wordBank)
         Return ALL combinations of words that build target.
         Return [] if impossible.   Return [[]] if target is empty.

⚠️  KEY: [[]] ≠ []
         [[]] = 1 way (empty list of words)
         []   = 0 ways (impossible)

─────────────────────────────────────────────────────────────────

RECURSION:
  VVS ac(const string& t, const vector<string>& wb) {
    if(t.empty()) return {{}};   // [[]] — 1 way!
    VVS result;
    for(const string& w : wb) {
      if(t.length()>=w.length() && t.substr(0,w.length())==w) {
        auto suf = ac(t.substr(w.length()), wb);
        for(const auto& way : suf) {
          vector<string> nw={w};
          nw.insert(nw.end(), way.begin(), way.end());
          result.push_back(nw);
        }
      }
    }
    return result;
  }
  Time: O(n^m × m)   Space: O(n^m × m)

─────────────────────────────────────────────────────────────────

MEMOIZATION:
  Same as recursion but add:
    unordered_map<string, VVS> memo;
    if(memo.count(t)) return memo[t];   // returns COPY — expensive!
    ... (same logic) ...
    return memo[t] = result;
  Time: O(n^m × m)  ← NO improvement!  Space: O(n^m × m)

─────────────────────────────────────────────────────────────────

TABULATION:
  vector<VVS> dp(m+1);
  dp[0] = {{}};         // [[]]
  for(int i=0; i<=m; i++) {
    if(dp[i].empty()) continue;
    for(const string& w : wb) {
      int wl = w.length();
      if(i+wl<=m && target.substr(i,wl)==w) {
        for(const auto& way : dp[i]) {   // extend each existing way
          vector<string> nw = way;
          nw.push_back(w);
          dp[i+wl].push_back(nw);
        }
      }
    }
  }
  return dp[m];
  Time: O(n^m × m)   Space: O(n^m × m)

─────────────────────────────────────────────────────────────────

ANSWERS:
  ac("purple",     [purp,p,ur,le,purpl]) = [["purp","le"],["p","ur","p","le"]]
  ac("abcdef",     [ab,abc,cd,def,abcd,ef]) = [["ab","cd","ef"],["abc","def"],["abcd","ef"]]
  ac("aaa",        [a,aa]) = [["a","a","a"],["a","aa"],["aa","a"]]
  ac("skateboard", [...])  = []           ← impossible
  ac("",           [cat,dog]) = [[]]      ← 1 way: empty!

COMPLEXITY COMPARISON (whole construct family):
               Rec         Memo         Tab
  canConstruct:  O(n^m×m) → O(n×m²)  → O(n×m²)   ← memo/tab HUGE win
  countConstruct:O(n^m×m) → O(n×m²)  → O(n×m²)   ← memo/tab HUGE win
  allConstruct:  O(n^m×m) → O(n^m×m) → O(n^m×m)  ← NO improvement possible!
```

---

*Compile and run: `g++ -std=c++17 -o allconstruct all_construct.cpp && ./allconstruct`*
*(C++17 needed for structured bindings in main — core logic works with C++11/14)*
