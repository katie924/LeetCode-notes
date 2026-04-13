# Aggregation Pattern

## Concept Overview

Aggregation is a type of window function. It is used to combine multiple rows into one summarized result.
It helps answer questions like "how many", "how much", "average", "maximum", or "minimum".

In SQL, we usually use `COUNT()`, `SUM()`, `AVG()`, `MIN()`, and `MAX()` with `GROUP BY`.  
In Pandas, we usually use `groupby()`, `count()`, `sum()`, `mean()`, `min()`, and `max()`.

This pattern is common in database problems that ask for counts, totals, averages, maximum values, minimum values, or duplicate groups.

---

## Key Idea / Intuition

- Start from the rows you want to summarize
- Decide which column should define each group
- Use an aggregate function to calculate one result per group
- Use `HAVING` when you need to filter after grouping

- Common tools:
  | SQL | Pandas | Meaning |
  | --- | --- | --- |
  | `GROUP BY` | `groupby()` | Group rows by one or more columns |
  | `COUNT()` | `count()` / `size()` | Count rows or non-null values in each group |
  | `SUM()` | `sum()` | Add values in each group |
  | `AVG()` | `mean()` | Find the average value in each group |
  | `MIN()` / `MAX()` | `min()` / `max()` | Find the smallest or largest value in each group |
  | `HAVING` | filter after `groupby()` | Keep only groups that match a condition |

---

## Example Problems

### [182. Duplicate Emails](https://leetcode.com/problems/duplicate-emails/description/)

#### Description

**Problem**  
Find all emails that appear more than once.

**Pattern**  
`GROUP BY` + `HAVING COUNT(*)`

**Why this problem belongs here**  
We group rows by `email`, count how many rows each email has, and keep only groups where the count is greater than `1`.

#### Example

**Input**

`Person`
| id | email |
|---|---|
| 1 | a@b.com |
| 2 | c@d.com |
| 3 | a@b.com |

**Output**

| Email |
|---|
| a@b.com |

#### Solution

**MySQL**
```sql
SELECT email AS Email
FROM Person
GROUP BY email
HAVING COUNT(*) > 1;
```
- `COUNT(*)` is usually preferred here because it counts rows directly.
- Since `email` is not `NULL` in this problem, `COUNT(email)` also works.

**Pandas**
```python
# groupby count DataFrame approach
def duplicate_emails(person: pd.DataFrame) -> pd.DataFrame:
    return person.groupby("email")[["id"]].count() \
        .query("id > 1") \
        .reset_index().rename(columns={"email": "Email"}).drop("id", axis=1)
```
- `count()` counts non-null values in the `id` column for each email.
- `query("id > 1")` keeps only duplicate emails.

```python
# groupby count Series approach
def duplicate_emails(person: pd.DataFrame) -> pd.DataFrame:
    return person.groupby("email")["id"].count() \
        .loc[lambda x: x > 1] \
        .index.to_frame(name="Email").reset_index(drop=True)
```
- Since `id` is not `NULL`, this gives the same result as `size()` here.
- `.loc[lambda x: x > 1]` keeps only duplicate emails.
- `.index.to_frame(name="Email")` converts the email index back into a DataFrame.

```python
# groupby size Series approach
def duplicate_emails(person: pd.DataFrame) -> pd.DataFrame:
    return person.groupby("email").size() \
        .loc[lambda x: x > 1] \
        .index.to_frame(name="Email").reset_index(drop=True)
```
- `size()` counts how many rows each email has.

```python
# groupby size Series alternative
def duplicate_emails(person: pd.DataFrame) -> pd.DataFrame:
    return person.groupby("email").size() \
        .loc[lambda x: x > 1] \
        .reset_index()[["email"]].rename(columns={"email": "Email"})
```
- This version also works, but `index.to_frame(name="Email")` is more direct because the duplicate emails are already in the index.

```python
# groupby filter approach
def duplicate_emails(person: pd.DataFrame) -> pd.DataFrame:
    return person.groupby("email") \
        .filter(lambda x: len(x) > 1)[["email"]] \
        .drop_duplicates().rename(columns={"email": "Email"})
```
- `groupby("email")` creates one group for each email.
- `filter(lambda x: len(x) > 1)` keeps only groups that have more than one row.
- This is close to the idea of `GROUP BY email HAVING COUNT(*) > 1`.

```python
# duplicated approach
def duplicate_emails(person: pd.DataFrame) -> pd.DataFrame:
    bool_ser = person["email"].duplicated()
    return person.loc[bool_ser, ["email"]] \
        .drop_duplicates().rename(columns={"email": "Email"})
```
- `duplicated()` uses `keep="first"` by default, which marks duplicates as `True` except for the first occurrence.
- `drop_duplicates()` makes sure each duplicate email appears only once in the final answer.

---

## Common Pitfalls

- Forgetting that `WHERE` runs before grouping, while `HAVING` runs after grouping
- Grouping by the wrong column, which changes the meaning of each group
- Selecting non-aggregated columns that are not included in `GROUP BY`
- Confusing `COUNT(*)` with `COUNT(column)` when the column may contain `NULL`
- Forgetting to reset or rename columns after `groupby()` in pandas

---

## Summary

- Use aggregation when the problem asks for grouped results
- Use `GROUP BY` to define each group
- Use `HAVING` to filter groups after aggregation

---

## Related Concepts

- Joins
- Subqueries
- Window Functions
