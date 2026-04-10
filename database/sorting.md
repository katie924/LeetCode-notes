# Sorting Pattern

## Concept Overview

Sorting is used to arrange rows in a specific order so we can find the highest, lowest, first, last, or nth result.

In SQL, we usually use `ORDER BY`.  
In Pandas, we usually use `sort_values()`.

This is a very common pattern in database problems because many questions depend on the order of the rows, not just the values themselves.

---

## Key Idea / Intuition

- Start from the rows you want to compare
- Sort them by the most important column
- Decide whether duplicate values should count separately
- Keep only the rows you need after sorting

- Common SQL tools:
  | Tool | Purpose |
  | ---  | ---     |
  | `ORDER BY` | Sort rows by one or more columns |
  | `DISTINCT` | Remove duplicates before or after sorting |
  | `LIMIT x` | Specifies the maximum number of rows to return |
  | `OFFSET y` | The number of rows to skip from the beginning |

- Common Pandas tools:
  | Tool | Purpose |
  | ---  | ---     |
  | `sort_values()` | Sort rows by one or more columns |
  | `drop_duplicates()` | Remove duplicate values |
  | `head()` | Keep the first few rows |
  | `iloc[]` | Pick a row by position |

---

## Example Problems

### [177. Nth Highest Salary](https://leetcode.com/problems/nth-highest-salary/description/)

#### Description

**Problem**  
Return the `n`th highest distinct salary. If it does not exist, return `NULL`.

**Pattern**  
`DISTINCT` + `ORDER BY` + `LIMIT / OFFSET`  
Function return with scalar subquery behavior

**Why this problem belongs here**  
The main idea is to sort distinct salaries from high to low, then pick the `n`th value by position.

#### Example

**Input**

`Employee`
| Id | Salary |
|---|---|
| 1 | 100 |
| 2 | 200 |
| 3 | 300 |

`N = 2`

**Output**

| getNthHighestSalary(2) |
|---|
| 200 |

#### Solution

**MySQL**
```sql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
    SET N = N - 1;
    RETURN (
        SELECT DISTINCT salary
        FROM Employee
        ORDER BY salary DESC
        LIMIT 1 OFFSET N
    );
END
```
- About the `CREATE FUNCTION` structure:
  - `CREATE FUNCTION getNthHighestSalary(N INT)` defines a function named `getNthHighestSalary` with one input parameter, `N`.
  - `RETURNS INT` means this function must return one integer value.
  - `BEGIN ... END` marks the function body.
  - `RETURN (...)` returns the result of the expression inside it.
  - In this problem, the expression inside `RETURN` is a query, so MySQL uses that query result as the function output.
- Here, `SELECT ... LIMIT 1 OFFSET N` is used inside `RETURN (...)` as a scalar subquery.
- If the subquery returns no row, MySQL treats the result as `NULL`.
- That is why `getNthHighestSalary(2)` can return `NULL` when the second distinct salary does not exist.
- This is different from a standalone query like Problem 176, where no matching row means the final result is an empty result set instead of one row with `NULL`.

**Pandas**
```python
def nth_highest_salary(employee: pd.DataFrame, N: int) -> pd.DataFrame:
    df = employee["salary"].sort_values(ascending=False).drop_duplicates()
    return pd.DataFrame(
        {
            f"getNthHighestSalary({N})": [
                None if N <= 0 or len(df) < N else df.iloc[N - 1]
            ]
        }
    )
```

---

## Common Pitfalls

- Forgetting `DISTINCT` when duplicates should not count
- Sorting in the wrong direction
- Using `LIMIT` without thinking about whether you need `OFFSET`
- Forgetting that some nth-value problems need a fallback when the row does not exist

---

## Summary

- Use this pattern when the main task is to sort rows and pick a position from the ordered result
- It is especially common for highest, lowest, and nth-value problems
- If the problem needs an intermediate grouped result first, combine sorting with aggregation or a subquery

---

## Related Concepts

- Subqueries
- Ranking
- Aggregation
