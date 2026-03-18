# Assignment-2
```python
from typing import List, Tuple

# -----------------------------
# DATA STRUCTURE
# -----------------------------
class District:
    def __init__(self, id, cost, value):
        self.id = id
        self.cost = cost
        self.value = value

# -----------------------------
# CHECK CONFLICTS
# -----------------------------
def is_valid(selected_ids, conflicts):
    for a, b in conflicts:
        if a in selected_ids and b in selected_ids:
            return False
    return True

# -----------------------------
# 1. DYNAMIC PROGRAMMING
# -----------------------------
def knapsack_dp(districts: List[District], B: int, conflicts):
    n = len(districts)

    # dp[i][b] = max value using first i items and budget b
    dp = [[0] * (B + 1) for _ in range(n + 1)]

    # keep track of selections
    keep = [[set() for _ in range(B + 1)] for _ in range(n + 1)]

    for i in range(1, n + 1):
        d = districts[i - 1]
        for b in range(B + 1):
            # not take
            dp[i][b] = dp[i - 1][b]
            keep[i][b] = keep[i - 1][b].copy()

            # take if possible
            if d.cost <= b:
                prev_set = keep[i - 1][b - d.cost].copy()
                prev_set.add(d.id)

                if is_valid(prev_set, conflicts):
                    val = dp[i - 1][b - d.cost] + d.value
                    if val > dp[i][b]:
                        dp[i][b] = val
                        keep[i][b] = prev_set

    return dp[n][B], keep[n][B]


# -----------------------------
# 2. GREEDY STRATEGIES
# -----------------------------

# Greedy by highest value
def greedy_value(districts, B, conflicts):
    sorted_d = sorted(districts, key=lambda x: x.value, reverse=True)

    total_cost = 0
    total_value = 0
    selected = set()

    for d in sorted_d:
        if total_cost + d.cost <= B:
            new_set = selected | {d.id}
            if is_valid(new_set, conflicts):
                selected.add(d.id)
                total_cost += d.cost
                total_value += d.value

    return total_value, selected, total_cost


# Greedy by value/cost ratio
def greedy_ratio(districts, B, conflicts):
    sorted_d = sorted(districts, key=lambda x: x.value / x.cost, reverse=True)

    total_cost = 0
    total_value = 0
    selected = set()

    for d in sorted_d:
        if total_cost + d.cost <= B:
            new_set = selected | {d.id}
            if is_valid(new_set, conflicts):
                selected.add(d.id)
                total_cost += d.cost
                total_value += d.value

    return total_value, selected, total_cost


# -----------------------------
# 3. COMPARISON
# -----------------------------
def compare(districts, B, conflicts):
    print("\n--- TEST ---")

    dp_value, dp_set = knapsack_dp(districts, B, conflicts)
    print("DP value:", dp_value, "Selected:", dp_set)

    g1_val, g1_set, _ = greedy_value(districts, B, conflicts)
    print("Greedy (value):", g1_val, g1_set)

    g2_val, g2_set, _ = greedy_ratio(districts, B, conflicts)
    print("Greedy (ratio):", g2_val, g2_set)

    def gap(dp, gr):
        if dp == 0:
            return 0
        return (dp - gr) / dp * 100

    print("Gap value:", round(gap(dp_value, g1_val), 2), "%")
    print("Gap ratio:", round(gap(dp_value, g2_val), 2), "%")


# -----------------------------
# 4. TEST CASES
# -----------------------------
def run_tests():
    # conflicts example: (1,2) means cannot pick both
    conflicts = [(1, 2)]

    # Test 1
    d1 = [
        District(1, 3, 25),
        District(2, 2, 20),
        District(3, 1, 15),
        District(4, 4, 40)
    ]
    compare(d1, 5, conflicts)

    # Test 2
    d2 = [
        District(1, 5, 50),
        District(2, 3, 30),
        District(3, 4, 40),
        District(4, 2, 20)
    ]
    compare(d2, 6, conflicts)

    # Test 3
    d3 = [
        District(1, 2, 10),
        District(2, 2, 9),
        District(3, 3, 14),
        District(4, 4, 16)
    ]
    compare(d3, 5, conflicts)

    # Test 4
    d4 = [
        District(1, 6, 60),
        District(2, 5, 50),
        District(3, 4, 40),
        District(4, 3, 30)
    ]
    compare(d4, 8, conflicts)

    # Test 5 (greedy FAIL example)
    d5 = [
        District(1, 10, 60),
        District(2, 20, 100),
        District(3, 30, 120)
    ]
    # Budget 50
    # Greedy ratio picks 1 + 2 = 160
    # DP picks 2 + 3 = 220 (better)
    compare(d5, 50, [])

run_tests()
