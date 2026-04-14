# Ranking Pattern

## Concept Overview

Ranking is a type of window function.
It assigns an order or position to each row after sorting.

In SQL, we usually use `ROW_NUMBER()`, `RANK()`, and `DENSE_RANK()`.  
In Pandas, we usually use `rank()`.

This pattern is common in database problems that ask for top results, row positions, ties, or ranking within each group.

---

## Key Idea / Intuition

- Start from the rows you want to compare
- Sort them by the most important column
- Decide whether ties should share the same number
- Choose the ranking function that matches the result you need

- Common tools:
  | SQL |  Pandas | Meaning |
  | ---  | ---    | ---     |
  | `ROW_NUMBER() OVER (ORDER BY)` | `rank(method='first')` | Give every row a unique position after sorting |
  | `RANK() OVER (ORDER BY)` | `rank(method='min')` | Give the same rank to ties, with gaps after ties |
  | `DENSE_RANK() OVER (ORDER BY)` | `rank(method='dense')` | Give the same rank to ties, without gaps after ties |
  | `... OVER (PARTITION BY ... ORDER BY)` | `groupby().rank()` |Restart the ranking inside each group |

---

## Example Problems

### [178. Rank Scores](https://leetcode.com/problems/rank-scores/description/)

#### Description

**Problem**  
Rank scores from highest to lowest. Equal scores should have the same rank, and the next rank should be consecutive.

**Pattern**  
`DENSE_RANK()`

**Why this problem belongs here**  
This is the standard `DENSE_RANK()` example because tied scores share a rank and the next rank does not skip numbers.

#### Example

**Input**

`Scores`
| score |
|---|
| 4.00 |
| 4.00 |
| 3.85 |
| 3.65 |
| 3.65 |
| 3.50 |

**Output**

| score | rank |
|---|---|
| 4.00 | 1 |
| 4.00 | 1 |
| 3.85 | 2 |
| 3.65 | 3 |
| 3.65 | 3 |
| 3.50 | 4 |

#### Solution

**SQL**
```sql
SELECT score,
       DENSE_RANK() OVER (ORDER BY score DESC) AS rank
FROM Scores;
```

**Pandas**
```python
def order_scores(scores: pd.DataFrame) -> pd.DataFrame:
    scores['rank'] = scores['score'].rank(method='dense', ascending=False)
    return scores.sort_values(by='rank').drop('id', axis=1)
```

---

### [185. Department Top Three Salaries](https://leetcode.com/problems/department-top-three-salaries/description/)

#### Description

**Problem**  
Find employees whose salaries are in the top three unique salaries of each department.

**Pattern**  
Partitioned `DENSE_RANK()`

**Why this problem belongs here**  
We rank salaries inside each department and keep ranks `1` to `3`. `DENSE_RANK()` is important because employees with the same salary should share the same rank.

#### Example

**Input**

See the original problem statement for the full sample tables.

**Output**

See the original problem statement for the full sample result.

#### Solution

**SQL**
```sql
SELECT d.name AS Department, e.name AS Employee, e.salary AS Salary
FROM (
    SELECT *,
           DENSE_RANK() OVER (PARTITION BY departmentId ORDER BY salary DESC) AS rnk
    FROM Employee
) e
LEFT JOIN Department d ON e.departmentId = d.id
WHERE rnk <= 3;
```

- `PARTITION BY departmentId` restarts the ranking inside each department.
- `ORDER BY salary DESC` ranks higher salaries first.
- `rnk <= 3` keeps employees in the top three unique salaries.

```sql
-- Older join style
SELECT Department, Employee, Salary
FROM (
    SELECT d.name AS Department,
           e.name AS Employee,
           e.salary AS Salary,
           DENSE_RANK() OVER (PARTITION BY departmentId ORDER BY e.salary DESC) AS r
    FROM Employee e, Department d
    WHERE e.departmentId = d.id
) m
WHERE r <= 3;
```

- `FROM Employee e, Department d` is an older way to write a join.
- Modern SQL usually prefers explicit `JOIN ... ON ...` because it separates join logic from filter logic.

**Pandas**
```python
def top_three_salaries(
    employee: pd.DataFrame, department: pd.DataFrame
) -> pd.DataFrame:
    employee["rnk"] = employee.groupby("departmentId")["salary"].rank(
        ascending=False, method="dense"
    )
    df = employee.query("rnk <= 3") \
        .merge(department, left_on="departmentId", right_on="id") \
        [['name_y', 'name_x', 'salary']]
    df.columns = ['Department', 'Employee', 'Salary']
    return df
```

---

## Common Pitfalls

- Using `ROW_NUMBER()` when tied values should share the same rank
- Using `RANK()` when the problem needs no skipped numbers
- Using `DENSE_RANK()` when the problem needs exactly one unique row
- Forgetting the `ORDER BY` inside the window function
- Choosing the wrong sort direction, such as ascending instead of descending

---

## Summary

- `ROW_NUMBER()` gives every row a unique number
- `RANK()` gives the same rank to ties and may skip numbers
- `DENSE_RANK()` gives the same rank to ties without skipping numbers
- This pattern is useful for deduplication and ranking inside groups

---

## Related Concepts

- Sorting
- Window Functions
- Aggregation
