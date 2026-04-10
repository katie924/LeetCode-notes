# Subquery Pattern

## Concept Overview

A subquery is a query inside another query.

It helps break a problem into smaller steps, especially when the outer query depends on an intermediate result.

---

## Key Idea / Intuition

- Solve a smaller question first
- Use that result in the outer query
- Filter, compare, or join based on the inner query result

Subqueries are common in:
- existence checks
- filtering with grouped results
- comparing rows against an intermediate result

---

## Example Problems

### [176. Second Highest Salary](https://leetcode.com/problems/second-highest-salary/description/)

#### Description
**Problem**  
Find the second highest distinct salary.

**Pattern**  
Subquery + aggregation

**Why this problem belongs here**  
The inner query finds the highest salary first. The outer query removes that value, and `MAX()` returns the largest salary among the remaining rows.

#### Example
**Input**

`Employee`
| Id | Salary |
|---|---|
| 1 | 100 |
| 2 | 200 |
| 3 | 300 |

**Output**

| SecondHighestSalary |
|---|
| 200 |

#### Solution

**MySQL**
```sql
SELECT MAX(salary) AS SecondHighestSalary
FROM Employee
WHERE salary <> (
    SELECT MAX(salary) FROM Employee
);
-- `<` also works here because we want salaries below the maximum
```

**Pandas**
```python
def second_highest_salary(employee: pd.DataFrame) -> pd.DataFrame:
    max_salary = employee["salary"].max()
    filtered = employee.loc[employee["salary"] != max_salary, "salary"]
    return pd.DataFrame({"SecondHighestSalary": [filtered.max()]})
```

```python
# Sorting approach
def second_highest_salary(employee: pd.DataFrame) -> pd.DataFrame:
    ser = employee["salary"].sort_values(ascending=False).drop_duplicates()
    return pd.DataFrame(
        {"SecondHighestSalary": [None if ser.size < 2 else ser.iloc[1]]}
    )
    # `len(ser)` is the same as `ser.size` here
```

**Wrong Answer in MySQL**

```sql
SELECT DISTINCT salary AS SecondHighestSalary
FROM Employee
ORDER BY salary DESC
LIMIT 1 OFFSET 1;
```
- This query works only when a second highest distinct salary exists.
- If the table has fewer than two distinct salaries, `LIMIT 1 OFFSET 1` returns no row.
- The expected output is one row with `NULL` instead of an empty result.

---

## Common Pitfalls

- Returning more than one row when the outer query expects one value
- Forgetting to use `DISTINCT` when duplicates matter
- Writing a subquery when a window function would be simpler

---

## Summary

- Subqueries are useful when the answer depends on an intermediate result
- They work well with grouped maxima, filtering, and existence-style checks
- Many problems can be solved either with subqueries or with window functions

---

## Related Concepts

- Aggregation (GROUP BY)
- Sorting
- Joins
- Filtering
