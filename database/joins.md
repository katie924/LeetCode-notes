# JOIN / MERGE Pattern

## Concept Overview

JOIN is used to combine data from multiple tables based on a related column.

In SQL, we use `JOIN`.  
In Pandas, we use `merge()`.

This is one of the **most important concepts** in data processing.

---

## Key Idea / Intuition

- Start from one table
- Match rows from another table using a key
- Choose the correct join type
  * Only matches
  * All left
  * All right
  * Everything

- Common join types:
  | Type | SQL | Pandas | Meaning |
  | ---  | --- | ---    | ---     |
  | Inner Join | `INNER JOIN` | `how='inner'` | Keep matching rows only |
  | Left Join  | `LEFT JOIN`  | `how='left'`  | Keep all left rows      |
  | Right Join | `RIGHT JOIN` | `how='right'` | Keep all right rows     |
  | Full Join  | `FULL JOIN`  | `how='outer'` | Keep all rows           |

---

## Example Problems

### [175. Combine Two Tables](https://leetcode.com/problems/combine-two-tables/description/)

#### Description
**Problem**  
Return each person's first name, last name, city, and state.  
If an address does not exist, `city` and `state` should be `NULL`.

**Pattern**  
`LEFT JOIN`

**Why this problem belongs here**  
We need to keep all rows from `Person` and match rows from `Address` when available.

#### Example
**Input**

`Person`
| personId | lastName | firstName |
|----------|----------|-----------|
| 1        | Wang     | Allen     |
| 2        | Alice    | Bob       |

`Address`
| addressId | personId | city          | state      |
|----------|----------|---------------|------------|
| 1        | 2        | New York City | New York   |
| 2        | 3        | Leetcode      | California |

**Output**
| firstName | lastName | city          | state    |
|-----------|----------|---------------|----------|
| Allen     | Wang     | NULL          | NULL     |
| Bob       | Alice    | New York City | New York |

#### Solution
**MySQL**
```sql
SELECT p.firstName, p.lastName, a.city, a.state
FROM Person AS p
LEFT JOIN Address AS a
ON p.personId = a.personId;
```

**Pandas**

```python
def combine_two_tables(person: pd.DataFrame, address: pd.DataFrame) -> pd.DataFrame:
    return person.merge(address, on='personId', how='left')[['firstName', 'lastName', 'city', 'state']]
```

---

### [181. Employees Earning More Than Their Managers](https://leetcode.com/problems/employees-earning-more-than-their-managers/description/)

#### Description
**Problem**  
Find employees who earn more than their managers.

**Pattern**  
`JOIN`

**Why this problem belongs here**  
The `Employee` table needs to be joined with itself: one copy represents the employee, and the other copy represents that employee's manager.

#### Example
**Input**

`Employee`
| id | name  | salary | managerId |
|---|---|---|---|
| 1 | Joe   | 70000 | 3 |
| 2 | Henry | 80000 | 4 |
| 3 | Sam   | 60000 | NULL |
| 4 | Max   | 90000 | NULL |

**Output**

| Employee |
|---|
| Joe |

#### Solution
**MySQL**
```sql
SELECT e.name AS Employee
FROM Employee e
JOIN Employee e2 ON e.managerId = e2.id
WHERE e.salary > e2.salary;
```

```sql
-- This is an older style of writing an inner join.
SELECT e1.name AS Employee
FROM Employee e1, Employee e2
WHERE e1.managerId = e2.id
  AND e1.salary > e2.salary;
```

**Pandas**
```python
def find_employees(employee: pd.DataFrame) -> pd.DataFrame:
    return employee.merge(employee, left_on="managerId", right_on="id") \
        .query("salary_x > salary_y")[["name_x"]] \
        .rename(columns={"name_x": "Employee"})
```

---

### [183. Customers Who Never Order](https://leetcode.com/problems/customers-who-never-order/description/)

#### Description
**Problem**  
Find customers who never placed an order.

**Pattern**  
`LEFT JOIN` + `IS NULL`

**Why this problem belongs here**  
We keep all rows from `Customers`, join matching rows from `Orders`, and then keep only customers without a matching order.

#### Example
**Input**

`Customers`
| id | name |
|---|---|
| 1 | Joe |
| 2 | Henry |
| 3 | Sam |
| 4 | Max |

`Orders`
| id | customerId |
|---|---|
| 1 | 3 |
| 2 | 1 |

**Output**

| Customers |
|---|
| Henry |
| Max |

#### Solution
**MySQL**
```sql
SELECT c.name AS Customers
FROM Customers c
LEFT JOIN Orders o ON c.id = o.customerId
WHERE o.customerId IS NULL;
```

```sql
-- subquery approach
SELECT name AS Customers
FROM Customers
WHERE id NOT IN (
    SELECT customerId
    FROM Orders
);
```

**Pandas**
```python
# merge approach
def find_customers(customers: pd.DataFrame, orders: pd.DataFrame) -> pd.DataFrame:
    df = customers.merge(orders, left_on="id", right_on="customerId", how="left") \
        .query("customerId.isna()")
    return df[["name"]].rename(columns={"name": "Customers"})
```

```python
# isin approach
def find_customers(customers: pd.DataFrame, orders: pd.DataFrame) -> pd.DataFrame:
    df = customers[~customers["id"].isin(orders["customerId"])]
    return df[["name"]].rename(columns={"name": "Customers"})
```

---

## Common Pitfalls

- Using the wrong join type
- Joining on the wrong key
- Forgetting table aliases in self joins
- Producing duplicate rows because the join key is not unique

---

## Summary

- Join problems are about combining related tables correctly
- Many SQL interview questions depend more on choosing the right join than on complex logic
- Self joins and repeated joins are common follow-up patterns

---

## Related Concepts

- Aggregation (GROUP BY)
- Subqueries
- Window Functions
