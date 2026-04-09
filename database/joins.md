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
  |---|---|---|---|
  | Inner Join | `INNER JOIN` | `how='inner'` | Keep matching rows only |
  | Left Join | `LEFT JOIN` | `how='left'` | Keep all left rows |
  | Right Join | `RIGHT JOIN` | `how='right'` | Keep all right rows |
  | Full Join | `FULL JOIN` | `how='outer'` | Keep all rows |

---

## Example Problems

### [175. Combine Two Tables](https://leetcode.com/problems/combine-two-tables/description/)

#### 📝 Description
**Problem**  
Return each person's first name, last name, city, and state.  
If an address does not exist, `city` and `state` should be `NULL`.

**Pattern**  
`LEFT JOIN`

**Why this problem belongs here**  
We need to keep all rows from `Person` and match rows from `Address` when available.

#### 🧩 Example
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

#### 🧪 Solution
**MySQL**
```sql
SELECT p.firstName, p.lastName, a.city, a.state
FROM Person AS p
LEFT JOIN Address AS a
ON p.personId = a.personId;
```
**Pandas**

```python id="r4k2hd"
def combine_two_tables(person: pd.DataFrame, address: pd.DataFrame) -> pd.DataFrame:
    return person.merge(address, on='personId', how='left')[['firstName', 'lastName', 'city', 'state']]
```

---

## Common Pitfalls

- Using the wrong join type
- Joining on the wrong key
- Forgetting table aliases in self joins
- Producing duplicate rows because the join key is not unique

---

## Summary

- JOIN problems are about combining related tables correctly
- A lot of SQL interview questions depend more on choosing the right join than writing complex logic
- Self joins and repeated joins are very common follow-ups

---

## Related Concepts

- Aggregation (GROUP BY)
- Subqueries
- Window Functions
