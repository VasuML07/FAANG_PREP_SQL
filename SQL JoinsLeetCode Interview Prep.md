# SQL Joins (INNER, LEFT, RIGHT, FULL OUTER, CROSS) — LeetCode Interview Prep

> Focus: Interview-oriented JOIN problem solving through real LeetCode questions.
>
> Difficulty Mix: Easy • Medium • Hard
>
> Questions: **6 (All Non-Premium)**

---

# Table of Contents

- [1. 175. Combine Two Tables (Easy)](#1-175-combine-two-tables-easy)
- [2. 181. Employees Earning More Than Their Managers (Easy)](#2-181-employees-earning-more-than-their-managers-easy)
- [3. 197. Rising Temperature (Easy)](#3-197-rising-temperature-easy)
- [4. 570. Managers with at Least 5 Direct Reports (Medium)](#4-570-managers-with-at-least-5-direct-reports-medium)
- [5. 1068. Product Sales Analysis I (Easy)](#5-1068-product-sales-analysis-i-easy)
- [6. 1581. Customer Who Visited but Did Not Make Any Transactions (Easy)](#6-1581-customer-who-visited-but-did-not-make-any-transactions-easy)

---

# 1. 175. Combine Two Tables (Easy)

## Problem Statement

Return each person's first name, last name and city/state if available.

Tables:

- Person
- Address

Some people may not have an address.

---

## Key Techniques & Algorithms

- LEFT JOIN
- Nullable joins
- Foreign key matching

Visualization

```
Person

1 Alice
2 Bob
3 Charlie

Address

1 NY
3 LA

LEFT JOIN

Alice    NY
Bob      NULL
Charlie  LA
```

---

## Solution Approach

The interview wants to verify whether you understand that every person must appear in the output.

Since Address may be missing,

- Person becomes the driving table.
- LEFT JOIN preserves every row.
- Missing matches become NULL.

---

## SQL

```sql
SELECT
    p.firstName,
    p.lastName,
    a.city,
    a.state
FROM Person p
LEFT JOIN Address a
ON p.personId = a.personId;
```

---

## Tricks & Edge Cases

✓ INNER JOIN would incorrectly remove people without addresses.

✓ Don't filter city/state in WHERE after LEFT JOIN.

Bad:

```sql
WHERE city IS NOT NULL
```

This converts it into an INNER JOIN.

---

## LLM-Proof Discussion

Many candidates memorize LEFT JOIN.

Interviewers instead ask:

> Which table should be on the LEFT?

Choosing the wrong driving table immediately produces incorrect results.

Understanding preservation is more important than remembering syntax.

---

## Companies Asking This

- Amazon
- Google
- Meta
- Microsoft
- Bloomberg

---

# 2. 181. Employees Earning More Than Their Managers (Easy)

## Problem Statement

Return employees whose salary is greater than their manager's salary.

Single table:

Employee

---

## Key Techniques & Algorithms

- Self Join
- Table aliasing
- INNER JOIN

Visualization

```
Employee

ID Name Salary Manager

1 A 5000 NULL
2 B 7000 1
3 C 4000 1

Self Join

Employee
      |
      |
Manager
```

---

## Solution Approach

Manager and employee exist inside the same table.

Create two copies.

- One alias = employee
- One alias = manager

Join employee.managerId = manager.id.

Compare salaries.

---

## SQL

```sql
SELECT
    e.name AS Employee
FROM Employee e
INNER JOIN Employee m
ON e.managerId = m.id
WHERE e.salary > m.salary;
```

---

## Tricks & Edge Cases

Remember:

```
managerId -> id
```

NOT

```
id -> managerId
```

Root managers have NULL managerId.

INNER JOIN naturally removes them.

---

## LLM-Proof Discussion

This problem evaluates whether you recognize hierarchical data.

Without understanding self joins,

many candidates incorrectly attempt subqueries that become unnecessarily complicated.

---

## Companies Asking This

- Amazon
- Meta
- Apple
- Google

---

# 3. 197. Rising Temperature (Easy)

## Problem Statement

Find dates where today's temperature is higher than yesterday's.

---

## Key Techniques & Algorithms

- Self Join
- Date arithmetic
- INNER JOIN

Visualization

```
Yesterday ---- Today

20°C  ----> 25°C ✓

25°C  ----> 18°C ✗
```

---

## Solution Approach

Treat Weather as two copies.

Join today's row with yesterday's row.

Compare temperatures.

---

## SQL

```sql
SELECT
    w1.id
FROM Weather w1
JOIN Weather w2
ON DATEDIFF(w1.recordDate, w2.recordDate) = 1
WHERE w1.temperature > w2.temperature;
```

---

## Tricks & Edge Cases

Do NOT compare IDs.

Dates may not be consecutive.

Always compare dates.

---

## LLM-Proof Discussion

Many LLM-generated solutions incorrectly assume:

```
today.id = yesterday.id + 1
```

This breaks immediately when IDs are non-sequential.

Interviewers intentionally include this trap.

---

## Companies Asking This

- Meta
- Amazon
- Google

---

# 4. 570. Managers with at Least 5 Direct Reports (Medium)

## Problem Statement

Return managers having at least five direct reports.

---

## Key Techniques & Algorithms

- Self Join
- GROUP BY
- COUNT
- Aggregation after join

Visualization

```
Manager

      A
    / | | | \
   B C D E F

Count = 5
```

---

## Solution Approach

Join employees to their managers.

Each employee contributes one report.

Group by manager.

Filter using HAVING.

---

## SQL

```sql
SELECT
    m.name
FROM Employee e
JOIN Employee m
ON e.managerId = m.id
GROUP BY
    m.id,
    m.name
HAVING COUNT(*) >= 5;
```

---

## Tricks & Edge Cases

Don't group by employee.

Count employees reporting to manager.

HAVING filters aggregated values.

WHERE cannot.

---

## LLM-Proof Discussion

This problem checks understanding of execution order.

```
JOIN

↓

GROUP BY

↓

HAVING
```

Candidates confusing WHERE and HAVING usually fail.

---

## Companies Asking This

- Amazon
- Meta
- Microsoft
- Uber

---

# 5. 1068. Product Sales Analysis I (Easy)

## Problem Statement

Return product name, year and price for every sale.

Tables

- Sales
- Product

---

## Key Techniques & Algorithms

- INNER JOIN
- Foreign key lookup

Visualization

```
Sales

101
102

↓

JOIN

↓

Products

Laptop
Mouse
```

---

## Solution Approach

Every sale references a valid product.

Join using productId.

Return requested columns.

---

## SQL

```sql
SELECT
    p.product_name,
    s.year,
    s.price
FROM Sales s
INNER JOIN Product p
ON s.product_id = p.product_id;
```

---

## Tricks & Edge Cases

INNER JOIN is sufficient because every sale references an existing product.

No aggregation required.

---

## LLM-Proof Discussion

The interview checks whether candidates avoid unnecessary complexity.

Many beginners introduce GROUP BY or subqueries that are completely unnecessary.

---

## Companies Asking This

- Amazon
- Google
- Oracle

---

# 6. 1581. Customer Who Visited but Did Not Make Any Transactions (Easy)

## Problem Statement

Find customers who visited but made no transactions.

---

## Key Techniques & Algorithms

- LEFT JOIN
- NULL filtering
- GROUP BY

Visualization

```
Visits

1
2
3
4

Transactions

2
4

LEFT JOIN

1 NULL
2 OK
3 NULL
4 OK

Filter NULL
```

---

## Solution Approach

Preserve every visit.

Join transactions.

Missing transaction becomes NULL.

Count visits grouped by customer.

---

## SQL

```sql
SELECT
    v.customer_id,
    COUNT(*) AS count_no_trans
FROM Visits v
LEFT JOIN Transactions t
ON v.visit_id = t.visit_id
WHERE t.transaction_id IS NULL
GROUP BY v.customer_id;
```

---

## Tricks & Edge Cases

The NULL check belongs after the LEFT JOIN.

Do not use INNER JOIN.

Otherwise all missing visits disappear.

---

## LLM-Proof Discussion

This question distinguishes candidates who understand NULL-producing joins from those who merely memorize syntax.

Recognizing that unmatched rows are represented by NULL is the key insight.

---

## Companies Asking This

- Amazon
- Meta
- Google
- Microsoft

---

# JOIN Pattern Cheat Sheet

| Pattern | Typical Interview Signal | Common Problems |
|----------|--------------------------|-----------------|
| INNER JOIN | Matching rows only | 1068 |
| LEFT JOIN | Preserve left table | 175, 1581 |
| Self JOIN | Hierarchical or previous-day comparison | 181, 197, 570 |
| LEFT JOIN + NULL | Find missing records | 1581 |
| JOIN + GROUP BY | Count related rows | 570 |

---

# JOIN Interview Takeaways

| Pattern | Recognition Hint |
|----------|------------------|
| LEFT JOIN | Keep every row from one table |
| INNER JOIN | Only matching rows matter |
| Self JOIN | Same table compared to itself |
| GROUP BY after JOIN | Count relationships |
| NULL filtering | Detect missing matches |

---

# Interview Notes

Although SQL supports **RIGHT JOIN**, **FULL OUTER JOIN**, and **CROSS JOIN**, these appear very rarely in LeetCode compared to **INNER**, **LEFT**, and **SELF JOIN** patterns. In FAANG interviews, most join-related questions ultimately reduce to:

- Choosing the correct driving table
- Preserving or discarding unmatched rows intentionally
- Self-joining hierarchical or time-series data
- Combining joins with aggregation (`GROUP BY` + `HAVING`)
- Correctly handling `NULL` values after joins

Mastering the six problems above covers the overwhelming majority of join-based SQL interview patterns seen on LeetCode and in FAANG technical screens.
