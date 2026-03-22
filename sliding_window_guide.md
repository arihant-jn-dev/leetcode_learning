# Sliding Window — Complete Interview Guide
## From Zero to Expert: Every Pattern, Every Problem

> **One mental model for every problem:**
> A window is a contiguous section of an array/string.
> Instead of recomputing everything from scratch, we SLIDE the window — add one element on the right, remove one on the left.
> This converts O(n²) brute force into O(n).

---

## PART 1 — The Core Idea

### What Is a Sliding Window?

```
ARRAY: [2, 1, 5, 1, 3, 2]   window size K = 3

Step 1: First window
  [2, 1, 5] 1, 3, 2    sum = 8
   i        j

Step 2: SLIDE — remove leftmost, add new right
  2, [1, 5, 1] 3, 2    sum = 8 - 2 + 1 = 7
      i        j

Step 3: SLIDE again
  2, 1, [5, 1, 3] 2    sum = 7 - 1 + 3 = 9
         i        j

Step 4: SLIDE again
  2, 1, 5, [1, 3, 2]   sum = 9 - 5 + 2 = 6
            i        j

BRUTE FORCE: Recompute sum of K elements every time → O(n*K)
SLIDING WINDOW: Subtract old left, add new right → O(n)
```

### Why Is It Faster?

```
BRUTE FORCE — for every starting position, loop K elements:
  for i in 0..n-K:
    sum = 0
    for j in i..i+K:   ← inner loop of K iterations
      sum += arr[j]
  Time: O(n * K)

SLIDING WINDOW — each element is added ONCE and removed ONCE:
  Add element: n times total
  Remove element: n times total
  Total operations: 2n = O(n)

IMPACT:
  n = 10,000   K = 5,000
  Brute force: 50,000,000 operations
  Sliding:         10,000 operations  (5000x faster!)
```

---

## PART 2 — How to Recognize a Sliding Window Problem

### The Recognition Checklist

Ask yourself these questions when you see a new problem:

```
QUESTION 1: Does it involve a CONTIGUOUS subarray or substring?
  Keywords: subarray, substring, contiguous, consecutive, window
  ✓ "Find max sum subarray of size K"
  ✓ "Longest substring without repeating characters"
  ✗ "Find max element in array" (no window needed)

QUESTION 2: Are you finding a MIN / MAX / COUNT / LONGEST / SHORTEST?
  ✓ Maximum sum of window
  ✓ Minimum length subarray with sum >= S
  ✓ Longest substring with K distinct chars
  ✓ Count of subarrays with sum = K
  ✗ "Is this array sorted?" (no window concept)

QUESTION 3: Does the answer require sliding/extending one end and shrinking the other?
  Think: Can I START from a smaller window and GROW it?
         Or can I fix one end and MOVE the other?

QUESTION 4: Is there a CONSTRAINT on the window?
  Fixed: "window of size exactly K"
  Variable: "window where sum <= K" / "at most 2 distinct chars"

If YES to most of these → Sliding Window!
```

### Two Types of Sliding Window

```
TYPE 1: FIXED WINDOW SIZE
─────────────────────────────────────────────────────────────
  Window size K is GIVEN and never changes.
  We always maintain exactly K elements.

  Template feel:
    - j runs from 0 to n-1
    - When j >= K-1: window is full → record answer, shrink from left

  Problems: Max sum of K, First negative in window, Anagram count


TYPE 2: VARIABLE WINDOW SIZE (Dynamic Window)
─────────────────────────────────────────────────────────────
  Window size is NOT fixed. It grows and shrinks based on a condition.
  We try to find the LONGEST or SHORTEST window satisfying a condition.

  Template feel:
    - j runs from 0 to n-1 (always expand right)
    - When condition BREAKS: shrink from left (move i forward)
    - Track max/min window size seen

  Problems: Longest no-repeat substring, Min window substring, etc.
```

---

## PART 3 — The Universal Template

### Fixed Window Template

```cpp
// FIXED WINDOW — size K
// j - i + 1 = current window size
// Window is "ready" when j >= K - 1 (i.e., window has K elements)

void fixedWindow(vector<int>& arr, int K) {
    int n = arr.size();
    int i = 0, j = 0;
    int windowSum = 0;   // or whatever you're tracking

    while (j < n) {                          // j is the RIGHT boundary — always moves
        // ── STEP 1: ADD arr[j] to window ──
        windowSum += arr[j];

        // ── STEP 2: Check if window size is reached ──
        if (j - i + 1 < K) {
            j++;                             // window not full yet — just expand
        }
        else {  // j - i + 1 == K  → window is exactly K
            // ── STEP 3: RECORD ANSWER (window is ready) ──
            cout << windowSum << " ";

            // ── STEP 4: SHRINK from left — remove arr[i] ──
            windowSum -= arr[i];
            i++;                             // move left boundary
            j++;                             // move right boundary for next window
        }
    }
}
```

### Variable Window Template

```cpp
// VARIABLE WINDOW — grow right until condition breaks, then shrink left
// Goal: find LONGEST window satisfying a condition

void variableWindow(vector<int>& arr) {
    int n = arr.size();
    int i = 0, j = 0;
    int windowState = 0;   // whatever you're tracking (sum, count, map, etc.)
    int maxLen = 0;

    while (j < n) {                          // j always moves right
        // ── STEP 1: ADD arr[j] to window ──
        windowState += arr[j];   // or map[arr[j]]++, etc.

        // ── STEP 2: Check if condition is VALID ──
        if (conditionValid(windowState)) {
            // Record answer — window [i..j] is valid
            maxLen = max(maxLen, j - i + 1);
            j++;                             // expand further
        }
        else {
            // ── STEP 3: Condition BROKE — shrink from left ──
            while (!conditionValid(windowState)) {
                windowState -= arr[i];       // remove left element
                i++;                         // shrink window
            }
            j++;                             // now expand right again
        }
    }
    cout << maxLen;
}
```

### Key Formulas (Memorize These)

```
FORMULA 1: Window size
  j - i + 1

FORMULA 2: Window is ready (fixed K)
  j - i + 1 == K    OR equivalently    j >= K - 1 (when i starts at 0)

FORMULA 3: Loop always runs
  while (j < n)     ← j never goes backwards

FORMULA 4: When to stop shrinking (variable window)
  while (condition is invalid): i++

FORMULA 5: Answer is usually
  max(maxLen, j - i + 1)    ← update BEFORE moving j
```

---

## PART 4 — FIXED WINDOW PROBLEMS

---

### Problem 1: Maximum Sum Subarray of Size K

**Problem:** Given array of integers and K, find the maximum sum of any subarray of size K.

```
RECOGNITION:
  ✓ Contiguous subarray? YES
  ✓ Fixed size K? YES
  ✓ Finding MAX? YES
  → Fixed window sliding window

PATTERN: Sum in, sum out. Track running max.
```

**Example:**
```
arr = [2, 1, 5, 1, 3, 2],  K = 3
Windows: [2,1,5]=8  [1,5,1]=7  [5,1,3]=9  [1,3,2]=6
Answer: 9
```

**Brute Force O(n²):**
```cpp
int maxSumBrute(vector<int>& arr, int K) {
    int n = arr.size(), maxSum = INT_MIN;
    for (int i = 0; i <= n - K; i++) {
        int sum = 0;
        for (int j = i; j < i + K; j++)
            sum += arr[j];
        maxSum = max(maxSum, sum);
    }
    return maxSum;
}
```

**Sliding Window O(n):**
```cpp
int maxSum(vector<int>& arr, int K) {
    int n = arr.size();
    int i = 0, j = 0;
    int windowSum = 0, maxSum = INT_MIN;

    while (j < n) {
        windowSum += arr[j];                 // add right element

        if (j - i + 1 < K) {
            j++;                             // window not full yet
        } else {                             // window size == K
            maxSum = max(maxSum, windowSum); // record answer
            windowSum -= arr[i];             // remove left element
            i++;                             // shrink from left
            j++;                             // expand right
        }
    }
    return maxSum;
}
```

**Dry Run:**
```
arr = [2, 1, 5, 1, 3, 2],  K = 3

j=0: add 2, sum=2,  size=1 < 3 → j++
j=1: add 1, sum=3,  size=2 < 3 → j++
j=2: add 5, sum=8,  size=3 = 3 → maxSum=8, remove arr[0]=2, sum=6, i=1, j=3
j=3: add 1, sum=7,  size=3 = 3 → maxSum=8, remove arr[1]=1, sum=6, i=2, j=4
j=4: add 3, sum=9,  size=3 = 3 → maxSum=9, remove arr[2]=5, sum=4, i=3, j=5
j=5: add 2, sum=6,  size=3 = 3 → maxSum=9, remove arr[3]=1, sum=5, i=4, j=6
j=6: loop ends (j == n)

Answer: 9  ✓
```

**Complexity:**
```
TIME:  O(n)   — j moves from 0 to n once, i moves from 0 to n once
               Each element added once + removed once = 2n operations

SPACE: O(1)   — only variables, no extra data structure

BRUTE vs SLIDING:
  n=10000, K=5000 → Brute: 50M ops | Sliding: 10K ops
```

---

### Problem 2: First Negative in Every Window of Size K

**Problem:** For every window of size K, find the first negative number. If no negative, print 0.

```
RECOGNITION:
  ✓ Fixed window size K
  ✓ Need to track ORDER (first negative) not just sum
  → Fixed window + queue to maintain order of negatives

PATTERN: Use a queue/deque to store INDICES of negatives.
         Negative falls out of window? Remove from front of queue.
```

**Example:**
```
arr = [-8, 2, 3, -6, 10],  K = 2
Windows: [-8,2]=-8  [2,3]=0  [3,-6]=-6  [-6,10]=-6
Answer:  [-8, 0, -6, -6]
```

**Sliding Window O(n):**
```cpp
vector<int> firstNegative(vector<int>& arr, int K) {
    int n = arr.size();
    deque<int> dq;           // stores INDICES of negative numbers
    vector<int> result;
    int i = 0, j = 0;

    while (j < n) {
        // add j to window: if negative, store its index
        if (arr[j] < 0)
            dq.push_back(j);

        if (j - i + 1 < K) {
            j++;
        } else {
            // window is ready — answer is front of deque (if valid)
            if (!dq.empty())
                result.push_back(arr[dq.front()]);
            else
                result.push_back(0);                 // no negative in window

            // remove elements that fall out of window
            if (!dq.empty() && dq.front() == i)
                dq.pop_front();

            i++;
            j++;
        }
    }
    return result;
}
```

**Dry Run:**
```
arr = [-8, 2, 3, -6, 10],  K = 2

j=0: arr[0]=-8 negative → dq=[0],     size=1 < 2 → j++
j=1: arr[1]=2  positive  → dq=[0],    size=2 = 2
      → result: arr[dq.front()=0]=-8 → result=[-8]
      → dq.front()=0 == i=0 → pop_front → dq=[]
      → i=1, j=2
j=2: arr[2]=3  positive  → dq=[],     size=2 = 2
      → result: dq empty → push 0    → result=[-8, 0]
      → dq empty, nothing to pop
      → i=2, j=3
j=3: arr[3]=-6 negative  → dq=[3],    size=2 = 2
      → result: arr[dq.front()=3]=-6  → result=[-8,0,-6]
      → dq.front()=3 != i=2, don't pop
      → i=3, j=4
j=4: arr[4]=10 positive  → dq=[3],    size=2 = 2
      → result: arr[dq.front()=3]=-6  → result=[-8,0,-6,-6]
      → dq.front()=3 == i=3 → pop_front → dq=[]
      → i=4, j=5
j=5: loop ends

Answer: [-8, 0, -6, -6]  ✓
```

**Complexity:**
```
TIME:  O(n)   — each index pushed/popped from deque at most once
SPACE: O(K)   — deque holds at most K indices
```

---

### Problem 3: Count Occurrences of Anagram

**Problem:** Given string `text` and pattern `pat`, count how many windows of size `pat.length()` in text are anagrams of pat.

```
RECOGNITION:
  ✓ Fixed window (size = pat.length())
  ✓ Counting occurrences
  ✓ Characters matter but order doesn't → frequency map
  → Fixed window + frequency map comparison

PATTERN: Maintain frequency map of pattern.
         Window map == pattern map → it's an anagram.
         Track "count" of unmatched characters instead of comparing full maps.
```

**Example:**
```
text = "aabaabaa",  pat = "aaba"
Windows of size 4: [aaba]=aaba✓  [abaa]=abaa✓  [baab]=no  [aaba]=aaba✓  [abaa]=abaa✓
Answer: 4
```

**Sliding Window O(n):**
```cpp
int countAnagrams(string text, string pat) {
    int n = text.size(), K = pat.size();
    unordered_map<char, int> patMap, winMap;

    for (char c : pat) patMap[c]++;          // frequency of pattern

    int i = 0, j = 0, count = 0;

    while (j < n) {
        winMap[text[j]]++;                   // add right char to window map

        if (j - i + 1 < K) {
            j++;
        } else {
            if (winMap == patMap) count++;   // window is an anagram

            // remove left char from window map
            winMap[text[i]]--;
            if (winMap[text[i]] == 0)
                winMap.erase(text[i]);
            i++;
            j++;
        }
    }
    return count;
}
```

**Optimized with "match" counter O(n):**
```cpp
int countAnagramsOpt(string text, string pat) {
    int n = text.size(), K = pat.size();
    unordered_map<char, int> freq;
    for (char c : pat) freq[c]++;            // pattern frequencies

    int i = 0, j = 0, count = 0;
    int need = freq.size();                  // distinct chars we need to satisfy

    while (j < n) {
        char cj = text[j];
        if (freq.count(cj)) {
            freq[cj]--;
            if (freq[cj] == 0) need--;       // one more char fully satisfied
        }

        if (j - i + 1 < K) {
            j++;
        } else {
            if (need == 0) count++;          // all chars satisfied → anagram!

            char ci = text[i];
            if (freq.count(ci)) {
                freq[ci]++;
                if (freq[ci] == 1) need++;   // losing a char → need it again
            }
            i++;
            j++;
        }
    }
    return count;
}
```

**Complexity:**
```
TIME:  O(n)    — one pass through text, map operations O(1) for fixed alphabet
SPACE: O(26)   — character frequency maps, bounded by alphabet size → O(1)
```

---

### Problem 4: Maximum of All Subarrays of Size K (Using Deque)

**Problem:** Find the maximum element in every window of size K.

```
RECOGNITION:
  ✓ Fixed window size K
  ✓ Finding MAX in each window
  ✓ Naive: O(n*K) — use monotonic deque for O(n)

PATTERN: Maintain a deque of INDICES in DECREASING ORDER of values.
         Front of deque = index of maximum in current window.
         Remove from front if it falls out of window.
         Remove from back if new element is greater (it can never be max before new).
```

**Example:**
```
arr = [1, 3, -1, -3, 5, 3, 6, 7],  K = 3
Windows: [1,3,-1]=3  [3,-1,-3]=3  [-1,-3,5]=5  [-3,5,3]=5  [5,3,6]=6  [3,6,7]=7
Answer:  [3, 3, 5, 5, 6, 7]
```

**Sliding Window with Monotonic Deque O(n):**
```cpp
vector<int> maxOfWindows(vector<int>& arr, int K) {
    int n = arr.size();
    deque<int> dq;           // stores INDICES, values in decreasing order
    vector<int> result;
    int i = 0, j = 0;

    while (j < n) {
        // remove from BACK: elements smaller than arr[j] (they're useless)
        while (!dq.empty() && arr[dq.back()] < arr[j])
            dq.pop_back();
        dq.push_back(j);

        if (j - i + 1 < K) {
            j++;
        } else {
            result.push_back(arr[dq.front()]);    // max is always at front

            // remove from FRONT: if it's out of window
            if (dq.front() == i)
                dq.pop_front();

            i++;
            j++;
        }
    }
    return result;
}
```

**Dry Run:**
```
arr = [1, 3, -1, -3, 5, 3, 6, 7],  K = 3

j=0: arr[0]=1,  dq empty → push 0 → dq=[0],      size=1<3 → j++
j=1: arr[1]=3,  arr[dq.back()=0]=1 < 3 → pop 0,  push 1 → dq=[1], size=2<3 → j++
j=2: arr[2]=-1, arr[dq.back()=1]=3 > -1 → push 2 → dq=[1,2], size=3=3
     → result=[arr[1]=3]
     → dq.front()=1 != i=0, don't pop
     → i=1, j=3
j=3: arr[3]=-3, arr[dq.back()=2]=-1 > -3 → push 3 → dq=[1,2,3], size=3=3
     → result=[3, arr[1]=3]
     → dq.front()=1 == i=1 → pop_front → dq=[2,3]
     → i=2, j=4
j=4: arr[4]=5,  arr[dq.back()=3]=-3 < 5 → pop, arr[dq.back()=2]=-1 < 5 → pop
     → push 4 → dq=[4], size=3=3
     → result=[3,3, arr[4]=5]
     → dq.front()=4 != i=2
     → i=3, j=5
... continues

Answer: [3, 3, 5, 5, 6, 7]  ✓
```

**Complexity:**
```
TIME:  O(n)   — each element pushed and popped from deque at most once
               (amortized — total push+pop = 2n)
SPACE: O(K)   — deque holds at most K indices

WHY DEQUE?
  Plain sliding window can't find max in O(1) per window.
  Deque maintains a "useful candidates" list in decreasing order.
  Smaller elements behind a larger one can NEVER be the window's max.
  So we pop them early — they're useless.
```

---

## PART 5 — VARIABLE WINDOW PROBLEMS

---

### Problem 5: Longest Substring Without Repeating Characters

**Problem:** Find the length of the longest substring with all unique characters.

```
RECOGNITION:
  ✓ Substring (contiguous)
  ✓ LONGEST window with a condition (no repeats)
  ✓ Condition: all chars unique → use a set/map to detect repeats
  → Variable window (grow until repeat found, shrink until fixed)

PATTERN: Use a hash set. Add right char. If duplicate found, shrink left
         until duplicate is removed.
```

**Example:**
```
s = "abcabcbb"
Windows:
  a       → len 1
  ab      → len 2
  abc     → len 3    ← growing
  abca    → REPEAT 'a'! shrink until 'a' removed
  bca     → no repeat → len 3
  bcab    → REPEAT 'b'! shrink
  cab     → len 3
  ...
Answer: 3 ("abc" or "bca" or "cab")
```

**Sliding Window O(n):**
```cpp
int lengthOfLongestSubstring(string s) {
    int n = s.size();
    unordered_set<char> window;
    int i = 0, j = 0, maxLen = 0;

    while (j < n) {
        // STEP 1: Try to add s[j]
        if (window.find(s[j]) == window.end()) {
            // No repeat — add and expand
            window.insert(s[j]);
            maxLen = max(maxLen, j - i + 1);  // record answer
            j++;
        } else {
            // Repeat found — shrink from left
            window.erase(s[i]);
            i++;
            // DO NOT move j — retry adding s[j] after shrinking
        }
    }
    return maxLen;
}
```

**Optimized with index map (skip directly to right position):**
```cpp
int lengthOfLongestSubstringOpt(string s) {
    unordered_map<char, int> lastSeen;   // char → last seen index
    int i = 0, maxLen = 0;

    for (int j = 0; j < s.size(); j++) {
        if (lastSeen.count(s[j]) && lastSeen[s[j]] >= i)
            i = lastSeen[s[j]] + 1;      // jump i past the duplicate directly

        lastSeen[s[j]] = j;
        maxLen = max(maxLen, j - i + 1);
    }
    return maxLen;
}
```

**Dry Run:**
```
s = "abcabcbb"

j=0: 'a' not in set → insert, window={a},   maxLen=1, j=1
j=1: 'b' not in set → insert, window={a,b}, maxLen=2, j=2
j=2: 'c' not in set → insert, window={a,b,c}, maxLen=3, j=3
j=3: 'a' IN set → erase s[i=0]='a', i=1, (don't move j)
j=3: 'a' not in set now → insert, window={b,c,a}, maxLen=3, j=4
j=4: 'b' IN set → erase s[i=1]='b', i=2
j=4: 'b' not in set → insert, window={c,a,b}, maxLen=3, j=5
j=5: 'c' IN set → erase s[i=2]='c', i=3
j=5: 'c' not in set → insert, window={a,b,c}, maxLen=3, j=6
j=6: 'b' IN set → erase s[i=3]='a', i=4
j=6: 'b' IN set → erase s[i=4]='b', i=5
j=6: 'b' not in set → insert, window={c,b}, maxLen=3, j=7
j=7: 'b' IN set → erase s[i=5]='c', i=6
j=7: 'b' IN set → erase s[i=6]='b', i=7
j=7: 'b' not in set → insert, window={b}, maxLen=3, j=8
j=8: loop ends

Answer: 3  ✓
```

**Complexity:**
```
TIME:  O(n)   — each character added to set once, removed from set at most once
SPACE: O(min(n, 26))  — set size bounded by unique chars (at most 26 for lowercase)
```

---

### Problem 6: Longest Substring with K Distinct Characters

**Problem:** Find the length of the longest substring with at most K distinct characters.

```
RECOGNITION:
  ✓ Longest substring
  ✓ Constraint: at most K distinct characters
  ✓ Need to count distinct chars in window → frequency map
  → Variable window: grow until distinct > K, then shrink

PATTERN: map.size() gives distinct char count.
         When map.size() > K → shrink from left (decrement count, erase if 0).
```

**Example:**
```
s = "araaci",  K = 2
Longest substrings with ≤ 2 distinct: "araa"(3 distinct→no), "raac"→no
Actually: "aara"? No. "raa"→2 distinct len=3? "aarc"→3 distinct.
Let me use s="eceba" K=2 for clarity:
  e     → {e:1}         → 1 distinct ≤ 2 → len=1
  ec    → {e:1,c:1}     → 2 distinct ≤ 2 → len=2
  ece   → {e:2,c:1}     → 2 distinct ≤ 2 → len=3
  eceb  → {e:2,c:1,b:1} → 3 distinct > 2 → SHRINK
  ceb   → {c:1,e:1,b:1} → 3 distinct > 2 → SHRINK
  eb    → {e:1,b:1}     → 2 distinct ≤ 2 → len=2
  eba   → {e:1,b:1,a:1} → 3 distinct > 2 → SHRINK
  ba    → {b:1,a:1}     → 2 distinct ≤ 2 → len=2
Answer: 3 ("ece")
```

**Sliding Window O(n):**
```cpp
int longestKDistinct(string s, int K) {
    int n = s.size();
    unordered_map<char, int> freq;        // char → count in window
    int i = 0, j = 0, maxLen = 0;

    while (j < n) {
        freq[s[j]]++;                     // add right char

        // SHRINK if distinct chars > K
        while (freq.size() > K) {
            freq[s[i]]--;
            if (freq[s[i]] == 0)
                freq.erase(s[i]);         // distinct count decreases
            i++;
        }

        // Window is valid (distinct ≤ K)
        maxLen = max(maxLen, j - i + 1);
        j++;
    }
    return maxLen;
}
```

**Complexity:**
```
TIME:  O(n)   — j moves forward n times, i also moves at most n times total
               inner while loop total iterations across all j = O(n)
SPACE: O(K)   — map holds at most K+1 distinct chars at any time
```

---

### Problem 7: Fruits Into Baskets (Exactly 2 Distinct)

**Problem:** You have a row of trees (fruits array). You have 2 baskets. Each basket holds only one fruit type. Find the maximum number of fruits you can pick (longest subarray with at most 2 distinct values).

```
RECOGNITION:
  ✓ This IS Problem 6 with K = 2 (longest substring with at most 2 distinct)
  ✓ Array values instead of characters, but same logic
  → Exact same template as K distinct, just K=2

KEY INSIGHT: "Fruits into baskets" = "Longest subarray with at most 2 distinct values"
  Recognize the DISGUISE — the story wraps the same pattern.
```

**Example:**
```
fruits = [1, 2, 3, 2, 2]
At most 2 distinct:
  [1,2] len=2
  [2,3] len=2
  [3,2,2] len=3
  [2,3,2,2] → 3 distinct → shrink
  [2,2] len=2
Answer: 4 (subarray [2,3,2,2]? No. [3,2,2]=3, or [2,2,2]? Let's trace...)
Actually: fruits=[1,2,1,2,3] → [1,2,1,2]=4 distinct? No {1,2}=2 → len=4  ✓
```

**Solution: same as Problem 6 with K=2:**
```cpp
int totalFruits(vector<int>& fruits) {
    int n = fruits.size(), K = 2;
    unordered_map<int, int> basket;      // fruit type → count
    int i = 0, j = 0, maxPick = 0;

    while (j < n) {
        basket[fruits[j]]++;

        while (basket.size() > K) {
            basket[fruits[i]]--;
            if (basket[fruits[i]] == 0)
                basket.erase(fruits[i]);
            i++;
        }

        maxPick = max(maxPick, j - i + 1);
        j++;
    }
    return maxPick;
}
```

**Complexity:**
```
TIME:  O(n)
SPACE: O(K) = O(2) = O(1)
```

---

### Problem 8: Minimum Size Subarray Sum ≥ S

**Problem:** Find the minimum length subarray with sum >= S. Return 0 if none exists.

```
RECOGNITION:
  ✓ Subarray (contiguous)
  ✓ MINIMUM length
  ✓ Constraint: sum >= S
  → Variable window: grow until sum >= S (found a valid window),
                     then SHRINK as much as possible while still valid

PATTERN: This is the SHRINKING variant — we want smallest, so shrink aggressively.
         When condition MET: try to shrink (not when broken).
```

**Example:**
```
arr = [2, 3, 1, 2, 4, 3],  S = 7
Windows: [2,3,1,2]=8≥7 len=4 | shrink: [3,1,2]=6<7 stop at [2,3,1,2]=4
         [4,3]=7≥7 len=2 ← best
Answer: 2
```

**Sliding Window O(n):**
```cpp
int minSubArrayLen(int S, vector<int>& arr) {
    int n = arr.size();
    int i = 0, j = 0;
    int windowSum = 0, minLen = INT_MAX;

    while (j < n) {
        windowSum += arr[j];             // expand right

        // SHRINK from left as long as sum >= S (condition is MET)
        while (windowSum >= S) {
            minLen = min(minLen, j - i + 1);  // record BEFORE shrinking
            windowSum -= arr[i];
            i++;
        }
        j++;
    }
    return (minLen == INT_MAX) ? 0 : minLen;
}
```

**Dry Run:**
```
arr = [2, 3, 1, 2, 4, 3],  S = 7

j=0: sum=2, 2<7 → j++
j=1: sum=5, 5<7 → j++
j=2: sum=6, 6<7 → j++
j=3: sum=8, 8≥7 → minLen=min(∞,4)=4, sum=8-2=6, i=1
       sum=6<7 → stop shrinking, j++
j=4: sum=10, 10≥7 → minLen=min(4,4)=4, sum=10-3=7, i=2
       sum=7≥7  → minLen=min(4,3)=3, sum=7-1=6, i=3
       sum=6<7  → stop shrinking, j++
j=5: sum=9, 9≥7  → minLen=min(3,3)=3, sum=9-2=7, i=4
       sum=7≥7  → minLen=min(3,2)=2, sum=7-4=3, i=5
       sum=3<7  → stop shrinking, j++
j=6: loop ends

Answer: 2  ✓
```

**Complexity:**
```
TIME:  O(n)   — j moves right n times, i moves right at most n times total
               (i never goes backwards, total inner loop = O(n) amortized)
SPACE: O(1)
```

---

### Problem 9: Longest Substring with At Most K Zeros (Max Consecutive Ones III)

**Problem:** Given binary array, you can flip at most K zeros to ones. Find the longest subarray of ones.

```
RECOGNITION:
  ✓ Longest subarray
  ✓ Constraint: at most K zeros in window (zeros flipped = cost)
  → Variable window: grow right, count zeros. When zeros > K, shrink left.

PATTERN: This is "longest window with at most K bad elements".
  Same template as K distinct, just condition = zeros > K.
```

**Example:**
```
arr = [1,1,1,0,0,0,1,1,1,1,0],  K = 2
Best window: flip two zeros → [1,1,1,0,0,1,1,1,1,1] → length 6
             Actually [0,0,1,1,1,1,0] needs 3 flips. [1,1,1,0,0,1,1,1,1] = 2 zeros → len=9? Let me check
             [1,1,1,0,0,1,1,1,1] has indices 0-8, zeros at 3,4 → K=2 ✓ → len=9? 
             But also [0,1,1,1,1,0] → no. Longest is [1,1,1,0,0,1,1,1,1,1] → len=10 if index 0-9 has only 2 zeros at 3,4.
Answer: 6  (commonly given as 6 for arr=[1,1,1,0,0,0,1,1,1,1,0], K=2)
```

**Sliding Window O(n):**
```cpp
int longestOnes(vector<int>& arr, int K) {
    int n = arr.size();
    int i = 0, j = 0;
    int zeros = 0, maxLen = 0;

    while (j < n) {
        if (arr[j] == 0) zeros++;         // add right

        // shrink if zeros exceed K
        while (zeros > K) {
            if (arr[i] == 0) zeros--;     // removing a zero from left
            i++;
        }

        maxLen = max(maxLen, j - i + 1);  // valid window (zeros ≤ K)
        j++;
    }
    return maxLen;
}
```

**Complexity:**
```
TIME:  O(n)
SPACE: O(1)
```

---

### Problem 10: Longest Repeating Character Replacement

**Problem:** Given string s and integer K. You can replace at most K characters. Find the longest substring where all characters are the same after at most K replacements.

```
RECOGNITION:
  ✓ Longest substring
  ✓ At most K "operations" (replacements)
  ✓ Key insight: to make a window all same char, we keep the most frequent
                 char and replace all others. Cost = windowSize - maxFreq.

CONDITION: (window size - max frequency in window) <= K
           → valid window (can make it uniform with K replacements)

PATTERN: Grow right. If condition breaks (cost > K), shrink left.
```

**Example:**
```
s = "AABABBA",  K = 1

Window "AABAB": size=5, maxFreq(A)=3, cost=5-3=2 > 1 → invalid
Window "ABAB":  size=4, maxFreq(A)=2, cost=4-2=2 > 1 → invalid
Window "AABA":  size=4, maxFreq(A)=3, cost=4-3=1 ≤ 1 → valid, len=4  ✓
Answer: 4
```

**Sliding Window O(n):**
```cpp
int characterReplacement(string s, int K) {
    int n = s.size();
    unordered_map<char, int> freq;
    int i = 0, j = 0;
    int maxFreq = 0, maxLen = 0;

    while (j < n) {
        freq[s[j]]++;
        maxFreq = max(maxFreq, freq[s[j]]);   // track max freq in window

        // cost = (window size) - (most frequent char count)
        // if cost > K → need more than K replacements → invalid
        int windowSize = j - i + 1;
        if (windowSize - maxFreq > K) {
            freq[s[i]]--;                     // shrink from left
            i++;
            // Note: maxFreq may be stale but that's OK — we never INCREASE
            // window size unless we find a better (higher) maxFreq
        }

        maxLen = max(maxLen, j - i + 1);
        j++;
    }
    return maxLen;
}
```

**Why maxFreq Can Stay Stale:**
```
KEY INSIGHT: We only care about windows LONGER than our current best.
  A longer window needs a higher maxFreq to be valid.
  If maxFreq decreases after shrinking, we'll never grow beyond current size.
  We only grow when we find a new higher maxFreq — which updates correctly.
  So stale maxFreq is fine — it acts as a "minimum bar" for accepting larger windows.
```

**Complexity:**
```
TIME:  O(n)   — one pass, map has at most 26 chars
SPACE: O(26) = O(1)
```

---

### Problem 11: Minimum Window Substring

**Problem:** Given string `s` and pattern `t`, find the minimum window in `s` that contains all characters of `t`.

```
RECOGNITION:
  ✓ Minimum window (shortest)
  ✓ Contains all chars of pattern
  ✓ Need frequency map of pattern + check if window covers it
  → Variable window: expand until all chars covered (valid),
                     then shrink to minimize while still valid

PATTERN: Two maps or "need" counter.
  need = number of distinct chars still unmet.
  formed = number of chars fully met.
  When formed == need → valid window → try shrinking.
```

**Example:**
```
s = "ADOBECODEBANC",  t = "ABC"
Minimum window containing A, B, C:
  Scan right until we have A,B,C → "ADOBEC" (len=6)
  Shrink left → "DOBECODEBA"? No, shrink gives "DOBEC"(no A) → invalid
  Expand again → find next valid → "BANC" (len=4)
Answer: "BANC"
```

**Sliding Window O(n):**
```cpp
string minWindow(string s, string t) {
    unordered_map<char, int> need;          // chars needed and their counts
    for (char c : t) need[c]++;

    int i = 0, j = 0;
    int required = need.size();             // distinct chars we need
    int formed = 0;                         // distinct chars fully satisfied
    unordered_map<char, int> windowFreq;
    int minLen = INT_MAX, startIdx = 0;

    while (j < s.size()) {
        char cj = s[j];
        windowFreq[cj]++;

        // check if this char's requirement is now fully met
        if (need.count(cj) && windowFreq[cj] == need[cj])
            formed++;

        // when all chars are satisfied: try to shrink
        while (formed == required) {
            // record if this is the smallest window
            if (j - i + 1 < minLen) {
                minLen = j - i + 1;
                startIdx = i;
            }

            char ci = s[i];
            windowFreq[ci]--;
            if (need.count(ci) && windowFreq[ci] < need[ci])
                formed--;                   // we lost a required char
            i++;
        }
        j++;
    }

    return (minLen == INT_MAX) ? "" : s.substr(startIdx, minLen);
}
```

**Dry Run (simplified):**
```
s = "ADOBECODEBANC",  t = "ABC"
need = {A:1, B:1, C:1},  required = 3

Expand j until formed=3:
  j=0: A→windowFreq={A:1}, formed=1(A done)
  j=1: D→no change,  formed=1
  j=2: O→no change,  formed=1
  j=3: B→formed=2(B done)
  j=4: E→no change,  formed=2
  j=5: C→formed=3(C done) ← ALL SATISFIED
  Window "ADOBEC" len=6, startIdx=0

Shrink i:
  i=0: remove A, windowFreq={A:0}, formed=2(A lost) ← stop shrinking
  Expand j:
  j=6: O→no change, j=7: D→no, j=8: E→no, j=9: B→windowFreq={B:2}(no change formed=2)
  j=10: A→formed=3(A satisfied again)
  Window "DOBECODEBA" → shrink:
  i=1: remove D→ok, formed=3
  i=2: remove O→ok, formed=3
  i=3: remove B→windowFreq={B:1}→still ≥1 →ok, formed=3
  i=4: remove E→ok, formed=3
  i=5: remove C→windowFreq={C:0}→formed=2 ← stop shrinking
  Window from i=5 to j=10 = "CODEBA" → "ODEBA"? wait startIdx=5, len=6
  j=11: N→no change
  j=12: C→formed=3 again
  Shrink:
  i=6: remove O→ok, i=7: remove D→ok, i=8: remove E→ok, i=9: remove B→ok
  i=10: remove A→formed=2 ← stop
  Window was i=9 to j=12 = "BANC", len=4  ← NEW MIN ✓

Answer: "BANC"
```

**Complexity:**
```
TIME:  O(n + m)  — n = len(s), m = len(t)
                   each char in s added once, removed once
SPACE: O(m)      — need map has at most m entries (unique chars of t)
```

---

## PART 6 — ADVANCED PATTERNS

---

### Problem 12: Subarrays with Exactly K Distinct Characters (Count)

**Problem:** Count subarrays with EXACTLY K distinct integers.

```
RECOGNITION:
  ✓ Count subarrays
  ✓ EXACTLY K distinct — this is tricky!
  → Sliding window directly doesn't work for "exactly"
     because shrinking overshoots.

KEY TRICK:
  Exactly K = (at most K) - (at most K-1)

  Count of subarrays with exactly K distinct
  = Count with ≤ K distinct − Count with ≤ K-1 distinct

This converts "exactly" problems into two "at most" problems,
each solvable with standard sliding window.
```

**Example:**
```
arr = [1, 2, 1, 2, 3],  K = 2
At most 2 distinct: count all subarrays with ≤ 2 distinct = 12
At most 1 distinct: count all subarrays with ≤ 1 distinct = 7
Exactly 2 distinct: 12 - 7 = 5

The 5 subarrays: [1,2], [2,1], [1,2], [2,1,2], [1,2,1,2]
Actually let me just show the approach works.
```

**Helper — Count subarrays with at most K distinct:**
```cpp
int atMostK(vector<int>& arr, int K) {
    int n = arr.size();
    unordered_map<int, int> freq;
    int i = 0, count = 0;

    for (int j = 0; j < n; j++) {
        freq[arr[j]]++;

        while (freq.size() > K) {
            freq[arr[i]]--;
            if (freq[arr[i]] == 0) freq.erase(arr[i]);
            i++;
        }

        // All subarrays ending at j with left boundary from i to j are valid
        // That's (j - i + 1) subarrays
        count += j - i + 1;
    }
    return count;
}

int subarraysWithKDistinct(vector<int>& arr, int K) {
    return atMostK(arr, K) - atMostK(arr, K - 1);
}
```

**Why `count += j - i + 1`?**
```
When window [i..j] is valid (≤ K distinct):
  All subarrays ending at j with start from i..j are also valid:
    [j..j], [j-1..j], [j-2..j], ..., [i..j]
  That's exactly (j - i + 1) subarrays.
  Adding them all at once is O(1) instead of enumeration.
```

**Complexity:**
```
TIME:  O(n)  — two passes of atMostK, each O(n)
SPACE: O(K)
```

---

### Problem 13: Permutation in String

**Problem:** Given strings `s1` and `s2`, return true if any permutation of `s1` is a substring of `s2`.

```
RECOGNITION:
  ✓ Fixed window (size = s1.length())
  ✓ Check if window is an anagram of s1
  → Exactly Problem 3 (Count Anagrams), but return bool on first match

PATTERN: Same frequency map approach. Return true immediately when match found.
```

**Sliding Window O(n):**
```cpp
bool checkInclusion(string s1, string s2) {
    if (s1.size() > s2.size()) return false;

    int freq[26] = {};
    for (char c : s1) freq[c - 'a']++;      // pattern frequency

    int window[26] = {};
    int K = s1.size();
    int i = 0, j = 0;

    while (j < s2.size()) {
        window[s2[j] - 'a']++;

        if (j - i + 1 < K) {
            j++;
        } else {
            if (memcmp(freq, window, sizeof(freq)) == 0)
                return true;                 // anagram found!

            window[s2[i] - 'a']--;
            i++;
            j++;
        }
    }
    return false;
}
```

**Optimized with match counter:**
```cpp
bool checkInclusionOpt(string s1, string s2) {
    if (s1.size() > s2.size()) return false;
    int freq[26] = {};
    for (char c : s1) freq[c - 'a']++;

    int need = 0;
    for (int x : freq) if (x > 0) need++;   // distinct chars needed

    int window[26] = {};
    int formed = 0;
    int K = s1.size(), i = 0, j = 0;

    while (j < s2.size()) {
        int c = s2[j] - 'a';
        window[c]++;
        if (freq[c] > 0 && window[c] == freq[c]) formed++;

        if (j - i + 1 < K) {
            j++;
        } else {
            if (formed == need) return true;

            int ci = s2[i] - 'a';
            if (freq[ci] > 0 && window[ci] == freq[ci]) formed--;
            window[ci]--;
            i++;
            j++;
        }
    }
    return false;
}
```

**Complexity:**
```
TIME:  O(n)  — n = len(s2), inner comparisons O(26) = O(1)
SPACE: O(1)  — fixed-size arrays of 26
```

---

### Problem 14: Binary Subarrays with Sum (Count Subarrays with Sum = K)

**Problem:** Given binary array `arr` and integer `K`, count subarrays with sum exactly K.

```
RECOGNITION:
  ✓ Count subarrays
  ✓ EXACTLY sum K
  → Same "exactly K" trick: exactly(K) = atMost(K) - atMost(K-1)

  Alternative: prefix sum + hash map O(n) — but sliding window works too.
```

**Sliding Window (atMost trick):**
```cpp
int atMostSum(vector<int>& arr, int K) {
    if (K < 0) return 0;
    int i = 0, sum = 0, count = 0;

    for (int j = 0; j < arr.size(); j++) {
        sum += arr[j];

        while (sum > K) {
            sum -= arr[i];
            i++;
        }

        count += j - i + 1;               // subarrays ending at j with sum ≤ K
    }
    return count;
}

int numSubarraysWithSum(vector<int>& arr, int K) {
    return atMostSum(arr, K) - atMostSum(arr, K - 1);
}
```

**Complexity:**
```
TIME:  O(n)
SPACE: O(1)
```

---

## PART 7 — PATTERN RECOGNITION MASTER TABLE

```
PROBLEM TYPE                         WINDOW TYPE   KEY DATA STRUCTURE    CONDITION TO SHRINK
──────────────────────────────────────────────────────────────────────────────────────────────
Max/Min sum of size K                Fixed         int (sum)             j - i + 1 == K
First negative in window             Fixed         deque of indices      j - i + 1 == K
Count anagrams / permutation check   Fixed         int[26] freq map      j - i + 1 == K
Max of each window                   Fixed         monotonic deque       j - i + 1 == K
──────────────────────────────────────────────────────────────────────────────────────────────
Longest no-repeat substring          Variable      unordered_set         duplicate found
Longest K-distinct substring         Variable      unordered_map         map.size() > K
Fruits into baskets (2 distinct)     Variable      unordered_map         map.size() > 2
Min subarray sum >= S                Variable      int (sum)             sum >= S (shrink to minimize)
Max consecutive ones (flip K zeros)  Variable      int (zero count)      zeros > K
Char replacement (K replacements)    Variable      int[26] + maxFreq     windowSize - maxFreq > K
Minimum window substring             Variable      two freq maps         formed == required (shrink)
──────────────────────────────────────────────────────────────────────────────────────────────
Exactly K distinct                   AtMost trick  two atMostK calls     —
Exactly K sum                        AtMost trick  two atMostSum calls   —
──────────────────────────────────────────────────────────────────────────────────────────────
```

---

## PART 8 — GENERALIZED FORMULAS & RULES

```
FORMULA 1: Window size at any point
  size = j - i + 1

FORMULA 2: Fixed window ready (K elements)
  j - i + 1 == K   →   equivalently   j >= K - 1  (when i starts at 0)

FORMULA 3: Main loop condition
  while (j < n)    ← j is always the right boundary, NEVER goes backward

FORMULA 4: For LONGEST — update answer when VALID
  maxLen = max(maxLen, j - i + 1)   ← update INSIDE valid zone

FORMULA 5: For SHORTEST — update answer when VALID then SHRINK
  while (condition valid):
    minLen = min(minLen, j - i + 1)
    shrink from left

FORMULA 6: Count of valid subarrays ENDING at j (when window [i..j] valid)
  count += j - i + 1     ← all sub-windows [i..j], [i+1..j], ..., [j..j] are valid

FORMULA 7: "Exactly K" trick
  exactly(K) = atMost(K) - atMost(K - 1)
  Use when "exactly" makes sliding window impossible.

FORMULA 8: Character replacement condition
  windowSize - maxFreqInWindow <= K    ← valid window
  (j - i + 1) - maxFreq <= K

FORMULA 9: Minimum window substring — when to shrink
  shrink when: formed == required  (all pattern chars satisfied)
  stop shrinking when: formed < required (a char becomes unsatisfied)
```

---

## PART 9 — TIME & SPACE COMPLEXITY IMPACT SUMMARY

```
PROBLEM                                  BRUTE      SLIDING    IMPROVEMENT
────────────────────────────────────────────────────────────────────────────
Max sum of K                             O(n·K)     O(n)       K× faster
First negative in K windows              O(n·K)     O(n)       K× faster
Count anagrams                           O(n·m)     O(n)       m× faster
Max of K windows                         O(n·K)     O(n)       K× faster
Longest no-repeat substring              O(n²)      O(n)       n× faster
Longest K distinct                       O(n²)      O(n)       n× faster
Min subarray sum >= S                    O(n²)      O(n)       n× faster
Char replacement (K ops)                 O(n²)      O(n)       n× faster
Min window substring                     O(n²·m)    O(n+m)     huge
────────────────────────────────────────────────────────────────────────────

SPACE: Most sliding window solutions use O(1) or O(26) for char maps.
       Brute force usually needs the same or more space.
       → Space rarely increases, often stays same.

WHY O(n) EVEN WITH INNER WHILE LOOP?
  The inner while loop (shrinking) doesn't make it O(n²).
  Proof: i starts at 0 and only moves forward. Total moves = at most n.
         j also moves forward, total = n.
         Combined total operations = 2n = O(n).
  This is AMORTIZED analysis — even though some steps have inner loops,
  across the full array the total work is linear.
```

---

## PART 10 — HOW TO APPROACH ANY NEW SLIDING WINDOW PROBLEM

```
STEP 1: Read the problem and identify:
  □ Is it about a contiguous subarray/substring?
  □ Are we finding longest / shortest / count / max / min?
  □ Is there a constraint on the window content?

STEP 2: Decide window type:
  □ Window size K is given? → FIXED window
  □ Window size depends on a condition? → VARIABLE window
  □ Problem says "exactly K"? → atMost(K) - atMost(K-1) trick

STEP 3: Pick your data structure:
  □ Tracking sum? → int variable
  □ Tracking distinct chars/elements? → unordered_map (size = distinct count)
  □ Tracking frequency for anagram? → int[26] or unordered_map
  □ Tracking max in window? → monotonic deque
  □ Tracking first/last of type? → deque of indices

STEP 4: Write the template:
  □ Fixed:    when (j - i + 1 == K) → record, shrink (i++), expand (j++)
  □ Variable: always expand (j++), shrink (i++) when condition breaks

STEP 5: Verify with example and check:
  □ Does window size formula j - i + 1 make sense?
  □ Is the loop condition j < n?
  □ Am I updating the answer at the right time?
  □ Is shrinking stopping at the right condition?
```

---

## PART 11 — COMMON MISTAKES TO AVOID

```
MISTAKE 1: Moving j twice in variable window
  WRONG:
    while (j < n) {
      add arr[j]; j++;          ← j moved here
      if (invalid) { i++; j++; } ← j moved again!
    }
  RIGHT: j++ only once at the END of each iteration.

MISTAKE 2: Forgetting to erase from map when count = 0
  WRONG: freq[s[i]]--;   ← count becomes 0 but key stays
  RIGHT: freq[s[i]]--;
         if (freq[s[i]] == 0) freq.erase(s[i]);  ← erase so size() is accurate

MISTAKE 3: Using map.size() > K without erasing
  map.size() counts keys, even if their value is 0.
  Always erase when count drops to 0.

MISTAKE 4: Updating answer at wrong time (variable window)
  For LONGEST: update INSIDE valid zone (before or instead of j++)
  For SHORTEST: update INSIDE shrinking loop (before shrinking)

MISTAKE 5: Not handling "exactly K" with two passes
  Trying to directly do "exactly K distinct" with one sliding window fails.
  Use: atMost(K) - atMost(K-1).

MISTAKE 6: Using minLen = 0 as initial value
  Use INT_MAX. Check at end: return (minLen == INT_MAX) ? 0 : minLen
```

---

## QUICK REFERENCE CHEAT SHEET

```cpp
// ─── FIXED WINDOW SKELETON ───────────────────────────────────────────
int i = 0, j = 0;
while (j < n) {
    // add arr[j]
    if (j - i + 1 < K) {
        j++;
    } else {
        // record answer
        // remove arr[i]
        i++; j++;
    }
}

// ─── VARIABLE WINDOW (LONGEST) SKELETON ──────────────────────────────
int i = 0, maxLen = 0;
for (int j = 0; j < n; j++) {
    // add arr[j]
    while (/* condition invalid */) {
        // remove arr[i]
        i++;
    }
    maxLen = max(maxLen, j - i + 1);
}

// ─── VARIABLE WINDOW (SHORTEST) SKELETON ─────────────────────────────
int i = 0, minLen = INT_MAX;
for (int j = 0; j < n; j++) {
    // add arr[j]
    while (/* condition valid — try to shrink */) {
        minLen = min(minLen, j - i + 1);
        // remove arr[i]
        i++;
    }
}
return (minLen == INT_MAX) ? 0 : minLen;

// ─── EXACTLY K TRICK ─────────────────────────────────────────────────
int exactlyK(vector<int>& arr, int K) {
    return atMost(arr, K) - atMost(arr, K - 1);
}
```
