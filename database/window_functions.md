# Value Window Function Pattern

## Concept Overview

Value window functions are a type of window function.
They allow you to access values from other rows without changing the number of rows.

In SQL, we usually use `LEAD()`, `LAG()`, `FIRST_VALUE()`, and `LAST_VALUE()`.  
In Pandas, we usually use `shift()`.

This pattern is common in database problems that require comparing a row with its previous or next rows, detecting consecutive patterns, or tracking changes over time.

---

## Key Idea / Intuition

-  Start from the row you want to evaluate
-  Look at neighboring rows (previous or next)
-  Compare values across rows
-  Apply conditions based on those comparisons

-  Common tools:
    | SQL                                    | Pandas               | Meaning                         |
    | -------------------------------------- | -------------------- | ------------------------------- |
    | `LAG(col, k)`                          | `shift(k)`           | Look at the previous k-th row   |
    | `LEAD(col, k)`                         | `shift(-k)`          | Look at the next k-th row       |
    | `FIRST_VALUE()`                        | `transform('first')` | Get the first value in a window |
    | `LAST_VALUE()`                         | `transform('last')`  | Get the last value in a window  |
    | `... OVER (PARTITION BY ... ORDER BY)` | `groupby().shift()`  | Apply within each group         |

---

## Example Problems

### [180. Consecutive Numbers](https://leetcode.com/problems/consecutive-numbers/description/)

#### Description

**Problem**  
Find all numbers that appear at least three times consecutively.

**Pattern**  
`LEAD()` / `LAG()` (adjacent row comparison)

**Why this problem belongs here**  
This problem requires comparing each row with its next two rows to check whether the same value appears consecutively. It is a direct application of value window functions.

#### Example

**Input**

`Logs`

| id | num |
| -- | --- |
| 1  | 1   |
| 2  | 1   |
| 3  | 1   |
| 4  | 2   |
| 5  | 1   |
| 6  | 2   |
| 7  | 2   |

**Output**

| ConsecutiveNums |
| --------------- |
| 1               |

#### Solution

**MySQL**

```sql
-- Use LEAD() to get next rows first, then filter in a subquery
SELECT DISTINCT num AS ConsecutiveNums
FROM (
    SELECT num,
           LEAD(num, 1) OVER (ORDER BY id) AS next_num,
           LEAD(num, 2) OVER (ORDER BY id) AS next_num2
    FROM Logs
) t
WHERE num = next_num AND num = next_num2;
```

```sql
-- Self Join approach
SELECT DISTINCT l1.num AS ConsecutiveNums
FROM Logs l1, Logs l2, Logs l3
WHERE l1.id = l2.id - 1 
  AND l2.id = l3.id - 1
  AND l1.num = l2.num 
  AND l2.num = l3.num;
```
- Join the table with itself to align consecutive rows
- Simulates adjacent row comparison using self-joins
- More general but less intuitive

**Pandas**

```python
# Create shifted columns, then filter rows.
def consecutive_numbers(logs: pd.DataFrame) -> pd.DataFrame:
    logs['num2'] = logs['num'].shift(-1)
    logs['num3'] = logs['num2'].shift(-1)

    return logs.query('num == num2 == num3')[['num']] \
        .drop_duplicates() \
        .rename(columns={'num': 'ConsecutiveNums'})
```

```python
def consecutive_numbers(logs: pd.DataFrame) -> pd.DataFrame:
    result = logs[
        (logs['num'] == logs['num'].shift(-1)) &
        (logs['num'] == logs['num'].shift(-2))
    ][['num']].drop_duplicates()

    return result.rename(columns={'num': 'ConsecutiveNums'})
```

---

## Common Pitfalls

- Forgetting to sort rows before applying window functions
- Using `LEAD()`/`LAG()` without `ORDER BY`, leading to incorrect comparisons
- Not handling boundary rows (first or last rows return NULL)
- Comparing values without considering NULL behavior
- Using aggregation instead of row-wise comparison

---

## Summary

- Value window functions access data from other rows without collapsing rows
- `LAG()` looks backward, `LEAD()` looks forward
- They are useful for detecting patterns across adjacent rows
- This pattern is common in time series, consecutive checks, and change detection

---

## Related Concepts

- Window Functions
- Adjacent Row Comparison
- Gaps and Islands
- Time Series Analysis
