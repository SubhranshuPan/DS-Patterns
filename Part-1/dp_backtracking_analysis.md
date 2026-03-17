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

### Why It Works (Intuition)

Think of it as filling a table bottom-up. For every sub-amount from `0` to `amount`, we ask: *"What's the fewest coins I can use?"* By considering each coin and updating all reachable sub-amounts, we guarantee we've explored every valid combination.

### Example

```python
coins = [1, 5, 10, 25]
amount = 30

print(coin_change(coins, amount))  # Output: 2
```

**Trace for `amount = 30`, `coins = [1, 5, 10, 25]`:**

| Stage | What happens |
|-------|-------------|
| After processing coin `1` | `dp[0..30]` = `[0, 1, 2, 3, ..., 30]` — every amount reachable with 1-cent coins |
| After processing coin `5` | `dp[5]` drops from `5` → `1`, `dp[10]` from `10` → `2`, etc. |
| After processing coin `10` | `dp[10]` → `1`, `dp[20]` → `2`, `dp[30]` → `3` |
| After processing coin `25` | `dp[25]` → `1`, `dp[30]` = `min(3, dp[30-25]+1)` = `min(3, dp[5]+1)` = `min(3, 2)` = **2** |

Result: **2 coins** (one 25¢ + one 5¢).

### Time Complexity

| Aspect | Complexity |
|--------|-----------|
| **Time** | **O(C × A)** — where `C` = number of coin denominations, `A` = `amount`. Two nested loops: outer over coins, inner over amounts. |
| **Space** | **O(A)** — single 1D array of size `amount + 1`. |

---

## Function 2: `knapsack(weights, values, capacity)`

### What It Does

Solves the classic **0/1 Knapsack** problem: given `n` items (each with a weight and value), determine the **maximum total value** you can carry in a knapsack of a given `capacity`. Each item can be taken **at most once**.

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
| 2 | `dp = [[0] * (capacity + 1) for _ in range(n+1)]` | Create a 2D table of size `(n+1) × (capacity+1)`, initialized to `0`. `dp[i][w]` = max value achievable using the first `i` items with capacity `w`. Row 0 and column 0 are naturally `0` (no items or no capacity). |
| 3 | `for i in range(1, n + 1):` | Consider items one by one (1-indexed; item `i` corresponds to `weights[i-1]`). |
| 4 | `for w in range(capacity+1):` | For each item, evaluate every possible capacity from `0` to `capacity`. |
| 5 | `if weights[i-1] <= w:` | **Can we fit item `i`?** If the item's weight ≤ current capacity `w`, we have a choice. |
| 5a | `values[i-1] + dp[i-1][w - weights[i-1]]` | **Take the item**: gain its value, plus the best value from the remaining capacity using previous items. |
| 5b | `dp[i-1][w]` | **Skip the item**: carry forward the best value without this item. |
| 6 | `dp[i][w] = dp[i-1][w]` | Item doesn't fit → forced to skip it. |
| 7 | `return dp[n][capacity]` | Answer sits in the bottom-right cell: all `n` items considered, full `capacity`. |

### Why It Works (Intuition)

For every item, at every capacity level, we make the optimal choice: **take it or leave it**. Because we build the table bottom-up (from fewer items and smaller capacities to more items and larger capacities), each cell leverages already-computed optimal sub-solutions.

### Example

```python
weights  = [2, 3, 4, 5]
values   = [3, 4, 5, 6]
capacity = 8

print(knapsack(weights, values, capacity))  # Output: 10
```

**DP Table (rows = items 0..4, columns = capacity 0..8):**

| i \ w | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
|-------|---|---|---|---|---|---|---|---|---|
| **0** (no items) | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| **1** (w=2, v=3) | 0 | 0 | 3 | 3 | 3 | 3 | 3 | 3 | 3 |
| **2** (w=3, v=4) | 0 | 0 | 3 | 4 | 4 | 7 | 7 | 7 | 7 |
| **3** (w=4, v=5) | 0 | 0 | 3 | 4 | 5 | 7 | 8 | 9 | 9 |
| **4** (w=5, v=6) | 0 | 0 | 3 | 4 | 5 | 7 | 8 | 9 | **10** |

Reading `dp[4][8]` = **10** → take items with weights `3` and `5` (values `4 + 6 = 10`).

### Time Complexity

| Aspect | Complexity |
|--------|-----------|
| **Time** | **O(N × W)** — where `N` = number of items, `W` = `capacity`. Two nested loops. |
| **Space** | **O(N × W)** — full 2D table. (Can be optimized to **O(W)** with a 1D rolling array, but this implementation uses the 2D version for clarity.) |

---

## Summary Comparison

| Function | Problem | Time | Space | Key Idea |
|----------|---------|------|-------|----------|
| `coin_change` | Min coins to reach amount | O(C × A) | O(A) | Unbounded — each coin usable multiple times |
| `knapsack` | Max value within weight limit | O(N × W) | O(N × W) | 0/1 — each item used at most once |

> [!TIP]
> Both functions follow the **bottom-up DP** pattern: define a table, set base cases, fill the table iteratively using a recurrence relation, and read the answer from a specific cell.
