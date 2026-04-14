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

### [184. Department Highest Salary](https://leetcode.com/problems/department-highest-salary/description/)

#### Description
**Problem**  
Find employees who have the highest salary in each department.

**Pattern**  
Subquery + aggregation

**Why this problem belongs here**  
The subquery finds the maximum salary for each department. The outer query keeps employees whose `(departmentId, salary)` matches one of those maximum salary pairs.

#### Example
**Input**

`Employee`
| id | name | salary | departmentId |
|---|---|---|---|
| 1 | Joe | 70000 | 1 |
| 2 | Jim | 90000 | 1 |
| 3 | Henry | 80000 | 2 |
| 4 | Sam | 60000 | 2 |
| 5 | Max | 90000 | 1 |

`Department`
| id | name |
|---|---|
| 1 | IT |
| 2 | Sales |

**Output**

| Department | Employee | Salary |
|---|---|---|
| IT | Jim | 90000 |
| Sales | Henry | 80000 |
| IT | Max | 90000 |

#### Solution

**MySQL**
```sql
-- Derived table + join approach
SELECT d.name AS Department, e.name AS Employee, e.salary AS Salary
FROM (
    SELECT departmentId, MAX(salary) AS max_salary
    FROM Employee
    GROUP BY departmentId
) m
JOIN Department d ON m.departmentId = d.id
JOIN Employee e ON m.departmentId = e.departmentId
               AND m.max_salary = e.salary;
```
- The derived table `m` finds the maximum salary for each department.
- Then we join back to `Employee` to find the employee names with that salary.
- This handles ties, so multiple employees can be returned for the same department.

```sql
-- Tuple IN approach
SELECT d.name AS Department, e.name AS Employee, e.salary AS Salary
FROM Employee e
LEFT JOIN Department d ON e.departmentId = d.id
WHERE (e.departmentId, e.salary) IN (
    SELECT departmentId, MAX(salary)
    FROM Employee
    GROUP BY departmentId
);
```
- The subquery returns each department's maximum salary.
- The outer query keeps employees whose department and salary match that result.

```sql
-- Alternative with DENSE_RANK()
SELECT d.name AS Department, e.name AS Employee, e.salary AS Salary
FROM (
    SELECT *,
           DENSE_RANK() OVER (PARTITION BY departmentId ORDER BY salary DESC) AS rnk
    FROM Employee
) e
LEFT JOIN Department d ON e.departmentId = d.id
WHERE rnk = 1;
```
- `DENSE_RANK()` ranks salaries inside each department.
- `rnk = 1` keeps all employees tied for the highest salary.

**Pandas**
```python
# groupby max + merge approach
def department_highest_salary(
    employee: pd.DataFrame, department: pd.DataFrame
) -> pd.DataFrame:
    df = employee.groupby("departmentId")["salary"].max().reset_index() \
        .merge(department, left_on="departmentId", right_on="id") \
        .merge(employee, on=["departmentId", "salary"]) \
            [["name_x", "name_y", "salary"]]
    df.columns = ["Department", "Employee", "Salary"]
    return df
```

```python
# merge + groupby filter approach
def department_highest_salary(
    employee: pd.DataFrame, department: pd.DataFrame
) -> pd.DataFrame:
    df = employee.merge(department, left_on="departmentId", right_on="id")
    df = df.groupby("departmentId").apply(lambda x: x[x["salary"] == max(x["salary"])])\
        .reset_index(drop=True)[["name_y", "name_x", "salary"]]
    df.columns = ["Department", "Employee", "Salary"]
    return df
```

```python
# ranking approach
def department_highest_salary(
    employee: pd.DataFrame, department: pd.DataFrame
) -> pd.DataFrame:
    employee["rnk"] = employee.groupby("departmentId")["salary"].rank(
        method="dense", ascending=False
    )
    df = employee.merge(department, left_on="departmentId", right_on="id") \
        .query("rnk == 1")[["name_y", "name_x", "salary"]]
    df.columns = ["Department", "Employee", "Salary"]
    return df
```

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
