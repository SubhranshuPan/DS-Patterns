# DP-Backtracking.ipynb — Function Analysis

---

## Function 1: `coin_change(coins, amount)`

### What It Does

Finds the **minimum number of coins** needed to make up a given `amount`. If it's impossible, returns `-1`.

### Code Walkthrough

```python
def coin_change(coins, amount):
    """Minimum coins to make amount"""
    dp = [float('inf')] * (amount + 1)   # Step 1
    dp[0] = 0                             # Step 2

    for coin in coins:                    # Step 3
        for i in range(coin, amount + 1): # Step 4
            dp[i] = min(dp[i], dp[i-coin] + 1)  # Step 5

    return dp[amount] if dp[amount] != float('inf') else -1  # Step 6
```

| Step | Line | Explanation |
|------|------|-------------|
| 1 | `dp = [float('inf')] * (amount + 1)` | Create a DP array of size `amount + 1`, initialized to **infinity** (`inf`). `dp[i]` will hold the minimum coins needed to make amount `i`. Infinity means "not yet reachable." |
| 2 | `dp[0] = 0` | **Base case**: 0 coins are needed to make amount `0`. |
| 3 | `for coin in coins:` | Iterate over each coin denomination. |
| 4 | `for i in range(coin, amount + 1):` | For the current coin, try every amount from `coin` up to `amount`. We start at `coin` because you can't use a coin larger than the sub-amount. |
| 5 | `dp[i] = min(dp[i], dp[i-coin] + 1)` | **Core transition**: "Can I improve `dp[i]` by using one more of this `coin`?" If `dp[i - coin] + 1` (use this coin + best way to make the remainder) is smaller than the current `dp[i]`, update it. |
| 6 | `return dp[amount] if dp[amount] != float('inf') else -1` | If `dp[amount]` is still infinity, no combination of coins can make the amount → return `-1`. Otherwise return the answer. |

### Example

```python
coins = [1, 5, 10, 25]
amount = 30
print(coin_change(coins, amount))  # Output: 2  (one 25¢ + one 5¢)
```

### Time Complexity

| Aspect | Complexity |
|--------|-----------|
| **Time** | **O(C × A)** — `C` = coin denominations, `A` = amount |
| **Space** | **O(A)** — single 1D array |

---

## Function 2: `knapsack(weights, values, capacity)`

### What It Does

Solves the **0/1 Knapsack** problem: maximize total value of items fitting in a knapsack of given `capacity`. Each item used **at most once**.

### Code Walkthrough

```python
def knapsack(weights, values, capacity):
    """0/1 Knapsack problem"""
    n = len(weights)                                        # Step 1
    dp = [[0] * (capacity + 1) for _ in range(n+1)]        # Step 2

    for i in range(1, n + 1):                               # Step 3
        for w in range(capacity+1):                         # Step 4
            if weights[i-1] <= w:                           # Step 5
                dp[i][w] = max(
                    values[i-1] + dp[i-1][w - weights[i-1]],  # Take item
                    dp[i-1][w]                                  # Skip item
                )
            else:
                dp[i][w] = dp[i-1][w]                       # Step 6

    return dp[n][capacity]                                  # Step 7
```

| Step | Line | Explanation |
|------|------|-------------|
| 1 | `n = len(weights)` | Number of items. |
| 2 | `dp = [[0]*(capacity+1) for _ in range(n+1)]` | 2D table `(n+1) × (capacity+1)`, init to `0`. `dp[i][w]` = max value with first `i` items and capacity `w`. |
| 3–4 | Nested loops | Consider each item at every capacity level. |
| 5 | `if weights[i-1] <= w` | If item fits: pick the better of **take** or **skip**. |
| 6 | `else: dp[i][w] = dp[i-1][w]` | Item doesn't fit → skip. |
| 7 | `return dp[n][capacity]` | Answer in the bottom-right cell. |

### Example

```python
weights  = [2, 3, 4, 5]
values   = [3, 4, 5, 6]
capacity = 8
print(knapsack(weights, values, capacity))  # Output: 10
```

### Time Complexity

| Aspect | Complexity |
|--------|-----------|
| **Time** | **O(N × W)** — `N` = items, `W` = capacity |
| **Space** | **O(N × W)** — full 2D table |

---

## Function 3: `subsets(nums)` — Pattern 13: Backtracking

### What It Does

Generates **all possible subsets** (the power set) of a list of numbers. For `n` elements, there are `2^n` subsets (including the empty set and the full set).

### Code Walkthrough

```python
def subsets(nums):
    result = []                          # Step 1

    def backtrack(start, path):          # Step 2
        result.append(path[:])           # Step 3
        for i in range(start, len(nums)):  # Step 4
            path.append(nums[i])         # Step 5  (Choose)
            backtrack(i + 1, path)       # Step 6  (Explore)
            path.pop()                   # Step 7  (Un-choose)

    backtrack(0, [])                     # Step 8
    return result                        # Step 9
```

| Step | Line | Explanation |
|------|------|-------------|
| 1 | `result = []` | Initialize an empty list to collect all subsets. |
| 2 | `def backtrack(start, path):` | Define the recursive helper. `start` = index to begin picking from (avoids duplicates/revisiting). `path` = the subset we're currently building. |
| 3 | `result.append(path[:])` | **Record the current subset.** `path[:]` creates a **copy** — critical because `path` is mutated in-place. This runs at *every* call, meaning every partial path (including `[]`) is a valid subset. |
| 4 | `for i in range(start, len(nums)):` | Try adding each remaining element (from index `start` onwards) to the current path. Starting at `start` ensures we never go backwards, which **prevents duplicate subsets**. |
| 5 | `path.append(nums[i])` | **Choose**: add element `nums[i]` to the current subset. |
| 6 | `backtrack(i + 1, path)` | **Explore**: recurse with `start = i + 1`, so only elements *after* the one we just picked are considered next. |
| 7 | `path.pop()` | **Un-choose (Backtrack)**: remove the last element to restore the state, so the next iteration of the `for` loop can try a *different* element at this position. |
| 8 | `backtrack(0, [])` | Kick off the recursion from index `0` with an empty path. |
| 9 | `return result` | Return all collected subsets. |

### Why It Works (Intuition — The Backtracking Pattern)

The function uses the classic **Choose → Explore → Un-choose** backtracking template:

```
                        backtrack(0, [])
                      /        |         \
               [1]            [2]          [3]
              /    \            |
          [1,2]   [1,3]      [2,3]
           |
        [1,2,3]
```

At each node in this recursion tree:
1. The **current path is recorded** as a valid subset (step 3)
2. We **try extending** the path with each unused element after `start` (step 4–5)
3. After exploring that branch, we **undo** the choice and try the next element (step 7)

This guarantees every possible combination is visited exactly once.

### Example

```python
nums = [1, 2, 3]
print(subsets(nums))
# Output: [[], [1], [1,2], [1,2,3], [1,3], [2], [2,3], [3]]
```

**Step-by-step trace:**

| Call | `start` | `path` | Action |
|------|---------|--------|--------|
| 1 | 0 | `[]` | Append `[]` to result |
| 2 | — | `[1]` | Choose `1`, recurse |
| 3 | 1 | `[1]` | Append `[1]` to result |
| 4 | — | `[1,2]` | Choose `2`, recurse |
| 5 | 2 | `[1,2]` | Append `[1,2]` to result |
| 6 | — | `[1,2,3]` | Choose `3`, recurse |
| 7 | 3 | `[1,2,3]` | Append `[1,2,3]` to result. Loop ends (start=3 = len). Return. |
| 8 | — | `[1,2]` | Pop `3`. Loop ends. Return. |
| 9 | — | `[1]` | Pop `2`. Next iteration: choose `3` |
| 10 | — | `[1,3]` | Choose `3`, recurse |
| 11 | 3 | `[1,3]` | Append `[1,3]`. Loop ends. Return. |
| 12 | — | `[1]` | Pop `3`. Loop ends. Return. |
| 13 | — | `[]` | Pop `1`. Next iteration: choose `2` |
| 14 | — | `[2]` | Choose `2`, recurse |
| 15 | 2 | `[2]` | Append `[2]`. Choose `3`, recurse |
| 16 | 3 | `[2,3]` | Append `[2,3]`. Return. |
| 17 | — | `[2]` | Pop `3`. Return. |
| 18 | — | `[]` | Pop `2`. Next iteration: choose `3` |
| 19 | — | `[3]` | Choose `3`, recurse |
| 20 | 3 | `[3]` | Append `[3]`. Return. |
| 21 | — | `[]` | Pop `3`. Loop ends. Done! |

**Final result**: `[[], [1], [1,2], [1,2,3], [1,3], [2], [2,3], [3]]` — all **8 = 2³** subsets.

### Time Complexity

| Aspect | Complexity | Reasoning |
|--------|-----------|-----------|
| **Time** | **O(N × 2ᴺ)** | There are `2^N` subsets. For each subset, copying the path (`path[:]`) takes up to `O(N)` time. |
| **Space** | **O(N × 2ᴺ)** | Storing all `2^N` subsets, each of average length `N/2`. The recursion stack adds `O(N)` depth, dominated by the output size. |

> [!TIP]
> **Why O(N × 2ᴺ) and not just O(2ᴺ)?** Because each of the `2^N` subsets must be *copied* into the result list. Copying a subset of length `k` costs `O(k)`, and the sum of all subset sizes is `N × 2^(N-1)`, which is `O(N × 2^N)`.

---

## Summary Comparison

| Function | Pattern | Problem | Time | Space |
|----------|---------|---------|------|-------|
| `coin_change` | DP (Bottom-up) | Min coins to reach amount | O(C × A) | O(A) |
| `knapsack` | DP (Bottom-up) | Max value within weight limit | O(N × W) | O(N × W) |
| `subsets` | Backtracking | Generate all subsets | O(N × 2ᴺ) | O(N × 2ᴺ) |
