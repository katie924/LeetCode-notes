# JOIN / MERGE Pattern

## Concept Overview

JOIN is used to combine data from multiple tables based on a related column.

In SQL → `JOIN`, 
In Pandas → `merge()`

This is one of the **most important concepts** in data processing.

---

## Key Idea / Intuition

* You have **two tables**
* You match them using a **key column**
* You decide what to keep:
  * Only matches
  * All left
  * All right
  * Everything
* Type of JOIN:
  | Type       | SQL        | Pandas      | Description         |
  | ---------- | ---------- | ----------- | ------------------- |
  | Inner Join | INNER JOIN | how='inner' | Only matching rows  |
  | Left Join  | LEFT JOIN  | how='left'  | Keep all left rows  |
  | Right Join | RIGHT JOIN | how='right' | Keep all right rows |
  | Full Join  | FULL JOIN  | how='outer' | Keep everything     |

---

## Example Problems

### [175. Combine Two Tables](https://leetcode.com/problems/combine-two-tables/description/)

#### 📝 Description
**Problem:**
Return each person's first name, last name, city, and state.  
If an address does not exist, `city` and `state` should be `NULL`.

**Pattern:**
`LEFT JOIN`

**Why this problem belongs here:**
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

* Duplicate keys → unexpected row duplication
* Wrong join type → missing or extra rows

---

## Summary

* JOIN = combine tables using a key
* SQL ↔ Pandas mapping is direct
* Most problems involve:

  * choosing correct join type
  * handling duplicates

---

## Related Concepts

* Aggregation (GROUP BY)
* Subqueries
* Window Functions
