# Merge Sort - Complete Documentation

## Overview

**Merge Sort** is a **divide-and-conquer** algorithm that breaks down a list into smaller sublists, sorts them recursively, and then merges the sorted sublists to produce a final sorted output.

### Key Characteristics
- **Algorithm Type**: Divide and Conquer
- **Stability**: Stable (maintains relative order of equal elements)
- **Time Complexity**: O(n log n) in all cases (best, average, worst)
- **Space Complexity**: O(n) - requires additional space for temporary arrays

---

## Problem Statement

**LeetCode Reference**: [Merge Sorted Array](https://leetcode.com/problems/merge-sorted-array/description/)

---

## Implementation

```cpp
#include <bits/stdc++.h>
using namespace std;
#include <iostream>
#include <vector>

// Function to merge two sorted subarrays into one sorted array
void merge(vector<int>& arr, int left, int mid, int right) {
    
    // Calculate sizes of the two subarrays to be merged
    int n1 = mid - left + 1;  // Left subarray size
    int n2 = right - mid;     // Right subarray size
    
    // Create temporary arrays to hold the subarrays
    vector<int> leftArr(n1);
    vector<int> rightArr(n2);
    
    // Copy data to temporary arrays
    int k = left;
    for (int i = 0; i < n1; i++) {
        leftArr[i] = arr[k];
        k++;
    }   
    for (int i = 0; i < n2; i++) {  // Fixed: 'I' should be 'i'
        rightArr[i] = arr[k];
        k++;
    }
    
    // Merge the temporary arrays back into arr[left..right]
    int i = 0;      // Initial index of left subarray
    int j = 0;      // Initial index of right subarray
    k = left;       // Initial index of merged array
    
    // Compare elements from both arrays and put the smaller one in the result
    while (i < n1 && j < n2) {
        if (leftArr[i] <= rightArr[j]) {
            arr[k] = leftArr[i];
            i++;
        } else {
            arr[k] = rightArr[j];
            j++;
        }
        k++;
    }
    
    // Copy remaining elements of leftArr[] if any
    while (i < n1) {
        arr[k] = leftArr[i];
        i++;
        k++;
    }
    
    // Copy remaining elements of rightArr[] if any
    while (j < n2) {
        arr[k] = rightArr[j];
        j++;
        k++;
    }
}

// Main merge sort function that recursively sorts the array
void mergeSort(vector<int>& arr, int l, int r) {
    // Base case: if left >= right, array has 0 or 1 element (already sorted)
    if (l >= r)                      
        return;
    
    // Calculate middle index (safe from overflow)
    int mid = l + (r - l) / 2;
    
    mergeSort(arr, l, mid);        // Sort left half
    mergeSort(arr, mid + 1, r);    // Sort right half
    merge(arr, l, mid, r);         // Merge the sorted halves
}

int main() {
    vector<int> arr = {38, 27, 43, 3, 9, 82, 10};
    
    // Divide the array into halves recursively
    mergeSort(arr, 0, arr.size() - 1);
    
    cout << "Sorted array:\n";
    for (int num : arr)
        cout << num << " ";
    cout << endl;
    
    return 0;
}
```

---

## Important Interview Questions

### Q1: Why use `l + (r - l) / 2` instead of `(l + r) / 2`?

**Answer**: To avoid integer overflow when `l` and `r` are large values.

**Explanation with Example**:

```
Given:
l = 1,500,000,000
r = 2,000,000,000

❌ Using (l + r) / 2:
l + r = 3,500,000,000  → INTEGER OVERFLOW!

✅ Using l + (r - l) / 2:
r - l = 500,000,000
(r - l) / 2 = 250,000,000
mid = l + 250,000,000 = 1,750,000,000  → SAFE!
```

**Key Points**:
- `(left + right) / 2` works in most cases
- For very large integers, `left + right` could exceed the maximum integer value
- `left + (right - left) / 2` is **safer** and **recommended** in production code

---

## How Merge Sort Works

### Simple Example: `[5, 2, 4, 1, 3]`

```
DIVIDE PHASE:
[5, 2, 4, 1, 3]
       ↓
[5, 2, 4]   [1, 3]
    ↓          ↓
[5, 2] [4]   [1] [3]
  ↓
[5] [2]

MERGE PHASE (bottom-up):
[5] [2]     → [2, 5]
[2, 5] [4]  → [2, 4, 5]
[1] [3]     → [1, 3]
[2, 4, 5] [1, 3] → [1, 2, 3, 4, 5]
```

---

## Detailed Example: `[38, 27, 43, 3, 9, 82, 10]`

### Phase 1: DIVIDE

```
[38, 27, 43, 3, 9, 82, 10]
          ↓
[38, 27, 43, 3]        [9, 82, 10]
       ↓                      ↓
[38, 27] [43, 3]      [9] [82, 10]
   ↓        ↓                 ↓
[38] [27] [43] [3]        [82] [10]
```

### Phase 2: MERGE

```
Step 1: Merge pairs
[38] [27]     → [27, 38]
[43] [3]      → [3, 43]
[82] [10]     → [10, 82]

Step 2: Merge groups
[27, 38] [3, 43]  → [3, 27, 38, 43]
[9] [10, 82]      → [9, 10, 82]

Step 3: Final merge
[3, 27, 38, 43] [9, 10, 82] → [3, 9, 10, 27, 38, 43, 82]
```

---

## Recursion Tree Visualization

### Tree Structure

```
                    [38, 27, 43, 3, 9, 82, 10]
                    /                         \
        [38, 27, 43, 3]                  [9, 82, 10]
           /         \                      /        \
     [38, 27]      [43, 3]            [9]      [82, 10]
      /    \        /    \                      /     \
   [38]  [27]   [43]  [3]                   [82]   [10]
```

**DIVIDE PHASE** (Top to Bottom): Split until single elements

**MERGE PHASE** (Bottom to Top): Combine sorted subarrays

```
                    [3, 9, 10, 27, 38, 43, 82]  ← Final Result
                    /                         \
          [3, 27, 38, 43]                [9, 10, 82]
             /         \                    /        \
       [27, 38]      [3, 43]           [9]      [10, 82]
        /    \        /    \                      /     \
     [38]  [27]   [43]  [3]                   [82]   [10]
```

---

## Execution Order with Call Stack

### Full Execution Sequence

```
1.  mergeSort([38, 27, 43, 3, 9, 82, 10])  ← Initial call
2.  mergeSort([38, 27, 43, 3])             ← Left half
3.  mergeSort([38, 27])
4.  mergeSort([38])                         ← Base case, return
5.  mergeSort([27])                         ← Base case, return
6.  merge([38], [27]) → [27, 38]           ← First merge
7.  mergeSort([43, 3])
8.  mergeSort([43])                         ← Base case, return
9.  mergeSort([3])                          ← Base case, return
10. merge([43], [3]) → [3, 43]
11. merge([27, 38], [3, 43]) → [3, 27, 38, 43]

12. mergeSort([9, 82, 10])                 ← Right half
13. mergeSort([9])                          ← Base case, return
14. mergeSort([82, 10])
15. mergeSort([82])                         ← Base case, return
16. mergeSort([10])                         ← Base case, return
17. merge([82], [10]) → [10, 82]
18. merge([9], [10, 82]) → [9, 10, 82]

19. merge([3, 27, 38, 43], [9, 10, 82]) → [3, 9, 10, 27, 38, 43, 82]  ← Final merge
```

---

## Detailed Step-by-Step Execution

### Visual Representation with Indices

```
Initial array: [38, 27, 43, 3, 9, 82, 10]
Indices:        0   1   2   3  4  5   6

                            mergeSort(0, 6)
                            /              \
                  mergeSort(0, 3)      mergeSort(4, 6)
                    /        \              /        \
          mergeSort(0, 1)  mergeSort(2, 3)  mergeSort(4, 5)  mergeSort(6, 6)
            /      \         /      \          /      \             |
    mergeSort(0,0) mergeSort(1,1) mergeSort(2,2) mergeSort(3,3) mergeSort(4,4) mergeSort(5,5) [10]
         |            |            |            |            |            |
       [38]         [27]         [43]          [3]          [9]         [82]
```

### Execution Steps with Call Stack

```
Step 1: Call mergeSort(0, 6) for full array
        - mid = 0 + (6 - 0) / 2 = 3
        - Execute: mergeSort(0, 3)

Step 2: Call mergeSort(0, 3) for [38, 27, 43, 3]
        - mid = 0 + (3 - 0) / 2 = 1
        - Execute: mergeSort(0, 1)

Step 3: Call mergeSort(0, 1) for [38, 27]
        - mid = 0 + (1 - 0) / 2 = 0
        - Execute: mergeSort(0, 0)

Step 4: Call mergeSort(0, 0) for [38]
        - Base case (l >= r), return immediately

Step 5: Back to mergeSort(0, 1)
        - Step A complete
        - Execute Step B: mergeSort(1, 1)

Step 6: Call mergeSort(1, 1) for [27]
        - Base case (l >= r), return immediately

Step 7: Back to mergeSort(0, 1)
        - Both halves sorted
        - Execute Step C: merge(0, 0, 1)
        - Result: [27, 38]

Step 8: Back to mergeSort(0, 3)
        - Left half sorted: [27, 38]
        - Execute Step B: mergeSort(2, 3)

Step 9: Call mergeSort(2, 3) for [43, 3]
        - Similar process as above
        - Result: [3, 43]

Step 10: Back to mergeSort(0, 3)
         - Execute merge(0, 1, 3)
         - Merge [27, 38] and [3, 43]
         - Result: [3, 27, 38, 43]

Step 11: Back to mergeSort(0, 6)
         - Left half complete
         - Execute Step B: mergeSort(4, 6)

Step 12: Call mergeSort(4, 6) for [9, 82, 10]
         - Similar recursive process
         - Result: [9, 10, 82]

Step 13: Back to mergeSort(0, 6)
         - Execute final merge(0, 3, 6)
         - Merge [3, 27, 38, 43] and [9, 10, 82]
         - Result: [3, 9, 10, 27, 38, 43, 82] ✓
```

---

## Understanding the Three Steps in mergeSort

```cpp
mergeSort(arr, l, mid);     // Step A: Sort left half
mergeSort(arr, mid + 1, r); // Step B: Sort right half
merge(arr, l, mid, r);      // Step C: Merge sorted halves
```

**Key Insight**: Step C only executes **after** Steps A and B complete (due to recursion).

---

## Time and Space Complexity Analysis

### Summary

| Aspect | Value |
|--------|-------|
| **Time Complexity** | O(n log n) |
| **Space Complexity** | O(n) |
| **Best Case** | O(n log n) |
| **Average Case** | O(n log n) |
| **Worst Case** | O(n log n) |

### Recurrence Relation

```
T(n) = 2 * T(n/2) + O(n)

Where:
- 2 * T(n/2) = Time to sort two halves
- O(n) = Time to merge the sorted halves
```

---

## Understanding Time Complexity: O(n log n)

### Recursion Tree Approach

Merge Sort creates a **recursion tree** where each level represents a stage of division or merging.

#### Tree Height

**Maximum depth** = Number of times you can divide n by 2 until size becomes 1

```
height = log₂(n)
```

**Example**: For n = 8
- Level 0: 8 elements → 1 array
- Level 1: 4 elements each → 2 arrays  
- Level 2: 2 elements each → 4 arrays
- Level 3: 1 element each → 8 arrays

Total levels = log₂(8) = 3

#### Work Per Level

At each level, we process **all n elements** during the merge operation:

```
Level 0: n elements          → O(n) work
Level 1: n/2 + n/2 = n       → O(n) work
Level 2: n/4 + n/4 + n/4 + n/4 = n → O(n) work
...
Level k: n                   → O(n) work
```

#### Total Time Complexity

```
Total Time = (Work per level) × (Number of levels)
           = O(n) × O(log n)
           = O(n log n)
```

---

## Detailed Level Analysis

### Level-by-Level Breakdown

```
🔹 Level 0 (Root - n elements)
   - Number of subarrays: 1
   - Size of each: n
   - Merge work: O(n)

🔹 Level 1 (Split into 2 parts of n/2)
   - Number of subarrays: 2
   - Size of each: n/2
   - Merge work: 2 × O(n/2) = O(n)

🔹 Level 2 (Split into 4 parts of n/4)
   - Number of subarrays: 4
   - Size of each: n/4
   - Merge work: 4 × O(n/4) = O(n)

🔹 Level k (Generic level)
   - Number of subarrays: 2^k
   - Size of each: n / 2^k
   - Merge work: 2^k × (n / 2^k) = n
```

### Mathematical Proof

At level k:
- Number of subarrays = 2^k
- Size of each subarray = n / 2^k
- Work at this level = 2^k × (n / 2^k) = **n**

Since there are **log₂(n)** levels, total work = **n × log₂(n)**

Therefore, **T(n) = O(n log n)**

---

## Space Complexity: O(n)

### Why O(n)?

1. **Temporary Arrays**: The `merge` function creates temporary arrays to hold left and right subarrays
   - Maximum combined size = n

2. **Recursion Stack**: Call stack depth = O(log n)

3. **Total Space** = O(n) + O(log n) = **O(n)**

---

## Visual Example: Time Complexity for n = 8

```
                     [8 elements]                    ← Level 0: 1 × 8 = 8 comparisons
                    /            \
           [4 elements]    [4 elements]              ← Level 1: 2 × 4 = 8 comparisons
              /    \          /    \
          [2] [2]        [2] [2]                     ← Level 2: 4 × 2 = 8 comparisons
          / \  / \       / \  / \
        [1][1][1][1]   [1][1][1][1]                  ← Level 3: 8 × 1 = 8 comparisons

Total levels = log₂(8) = 3
Work per level = 8
Total work = 8 × 3 = 24 = 8 log₂(8) = n log n
```

---

## Key Takeaways

### ✅ Advantages
1. **Guaranteed O(n log n)** performance - no worst case degradation
2. **Stable sort** - maintains relative order of equal elements
3. **Predictable** - always takes the same time regardless of input
4. **Parallelizable** - can be optimized for parallel processing

### ❌ Disadvantages
1. **O(n) space** - requires additional memory
2. **Not in-place** - needs temporary arrays
3. **Overhead** - slower than quicksort in practice for small arrays

### 🎯 When to Use
- When **stability** is required
- When **guaranteed O(n log n)** is needed (no worst case)
- When **auxiliary space** is available
- For **external sorting** (sorting large files)

---

## Common Interview Questions

### Q2: Is Merge Sort stable?
**Yes**, Merge Sort is stable because when merging, equal elements from the left subarray are placed before those from the right subarray, maintaining their original relative order.

### Q3: Can Merge Sort be done in-place?
Technically yes, but it's complex and loses its O(n log n) guarantee. The standard implementation uses O(n) extra space for simplicity and efficiency.

### Q4: Why use Merge Sort over Quick Sort?
- **Merge Sort**: Guaranteed O(n log n), stable, but needs O(n) space
- **Quick Sort**: Average O(n log n) but worst case O(n²), in-place, not stable

Choose Merge Sort when you need **stability** and **guaranteed performance**.

### Q5: What is the recurrence relation?
```
T(n) = 2T(n/2) + O(n)
```
- `2T(n/2)`: Time to sort two halves
- `O(n)`: Time to merge

---

## Comparison with Other Sorting Algorithms

| Algorithm | Time (Best) | Time (Average) | Time (Worst) | Space | Stable |
|-----------|-------------|----------------|--------------|-------|--------|
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | ✅ Yes |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) | ❌ No |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | ❌ No |
| Bubble Sort | O(n) | O(n²) | O(n²) | O(1) | ✅ Yes |
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) | ✅ Yes |

---

## Additional Notes

### Optimization Tips
1. **Hybrid Approach**: Switch to insertion sort for small subarrays (< 10 elements)
2. **In-place Merging**: For memory-constrained environments (complex to implement)
3. **Natural Merge Sort**: Takes advantage of already-sorted sequences

### Real-World Applications
- **External sorting** (files too large for memory)
- **Linked list sorting** (efficient for linked structures)
- **Inversion count** problems
- **Database sorting** operations
- **Parallel sorting** algorithms (like parallel merge sort)

---

*Document created: January 25, 2026*  
*Algorithm: Merge Sort (Divide and Conquer)*  
*Time Complexity: O(n log n) | Space Complexity: O(n)*
