# Basic SELECT & WHERE — FAANG Interview Guide (LeetCode SQL)

> **Topic:** Basic SELECT and Filtering (WHERE)
>
> **Difficulty Coverage:** Easy → Medium → Hard (Filtering Perspective)
>
> **Interview Focus:** Learn every filtering pattern that repeatedly appears in FAANG SQL interviews.

---

# Table of Contents

- [Interview Patterns Covered](#interview-patterns-covered)
- [Question 1 — Recyclable and Low Fat Products (Easy)](#question-1--recyclable-and-low-fat-products-easy)
- [Question 2 — Find Customer Referee (Easy)](#question-2--find-customer-referee-easy)
- [Question 3 — Patients With a Condition (Easy)](#question-3--patients-with-a-condition-easy)
- [Question 4 — Big Countries (Easy)](#question-4--big-countries-easy)
- [Question 5 — Customers Who Never Order (Easy)](#question-5--customers-who-never-order-easy)
- [Question 6 — Immediate Food Delivery I (Medium Filtering Pattern)](#question-6--immediate-food-delivery-i-medium-filtering-pattern)
- [Common FAANG Follow-Up Questions](#common-faang-follow-up-questions)
- [Pattern Cheat Sheet](#pattern-cheat-sheet)

---

# Interview Patterns Covered

| Pattern | Used In |
|----------|----------|
| Basic WHERE | Q1 Q4 |
| Multiple Conditions | Q1 Q4 |
| AND / OR | Q1 Q4 |
| NOT | Q2 |
| NULL Handling | Q2 Q5 |
| LIKE | Q3 |
| Wildcards (%) | Q3 |
| IN / NOT IN | Q2 |
| EXISTS | Q5 |
| Correlated Subquery | Q5 |
| Filtering Before Aggregation | Q6 |
| Date Comparison | Q6 |

---

# Question 1 — Recyclable and Low Fat Products (Easy)

**LeetCode #1757**

Difficulty: Easy

Companies:
- Amazon
- Google
- Microsoft

---

## Problem

Table

| product_id | low_fats | recyclable |
|------------|----------|-------------|
| 0 | Y | N |
| 1 | Y | Y |
| 2 | N | Y |

Find products that are:

- Low fat
- Recyclable

Return product_id.

---

## Pattern

Multiple WHERE Conditions

```
WHERE A
AND B
```

---

## SQL Solution

```sql
SELECT product_id
FROM Products
WHERE low_fats = 'Y'
AND recyclable = 'Y';
```

---

## Why This Works

Both conditions must be true.

Think of WHERE as filtering rows one by one.

```
Product

        │
        ▼
low_fats='Y' ?
        │
   yes──┘
        ▼
recyclable='Y' ?
        │
   yes──┘
        ▼
Output
```

---

## Alternative

```sql
SELECT product_id
FROM Products
WHERE recyclable='Y'
AND low_fats='Y';
```

SQL optimizer treats these equally.

---

## Edge Cases

- lowercase y?
- NULL values
- additional flags

---

## Interview Follow-ups

- Add price >100
- Only products created this month
- Exclude discontinued products

---

## FAANG Usage

Filtering product catalogs

Example:

```
Amazon

WHERE
in_stock='Y'
AND prime='Y'
AND price<100
```

---

# Question 2 — Find Customer Referee (Easy)

**LeetCode #584**

Companies

- Meta
- Amazon

---

## Problem

| id | name | referee_id |
|----|------|------------|
|1|Will|NULL|
|2|Jane|NULL|
|3|Alex|2|
|4|Bill|NULL|

Return everyone except customers referred by customer 2.

---

## Important SQL Trick

Many candidates forget:

```
NULL != 2

returns UNKNOWN
```

Therefore

```
WHERE referee_id != 2
```

is WRONG.

---

## Correct Solution

```sql
SELECT name
FROM Customer
WHERE referee_id != 2
OR referee_id IS NULL;
```

---

## Better Visualization

```
referee_id

NULL
2
5

Condition

!=2

NULL → Unknown

Ignored

Need

IS NULL
```

---

## Alternative

```sql
SELECT name
FROM Customer
WHERE COALESCE(referee_id,0)<>2;
```

---

## Why Interviewers Ask This

To test NULL knowledge.

NULL behaves differently from numbers.

---

## Edge Cases

- Multiple NULLs
- referee_id = 0
- Empty table

---

## FAANG Usage

Customer referral systems

Marketing campaigns

Affiliate filtering

---

# Question 3 — Patients With a Condition (Easy)

**LeetCode #1527**

Companies

- Google
- Apple

---

## Problem

Patients table

Return patients whose condition starts with

```
DIAB1
```

Example

```
DIAB100
```

```
DIAB123
```

---

## SQL Solution

```sql
SELECT patient_id,
       patient_name,
       conditions
FROM Patients
WHERE conditions LIKE 'DIAB1%';
```

---

## Pattern

LIKE

Wildcard

```
%

Matches

0 or more characters
```

---

## Visualization

```
DIAB1%

Matches

DIAB100

DIAB12

DIAB123ABC

Does NOT Match

DIAB2

ABCDIAB1
```

---

## Alternative

If DIAB1 may appear anywhere

```sql
WHERE conditions LIKE '%DIAB1%'
```

---

## Edge Cases

Multiple diseases

```
FLU DIAB100 ASTHMA
```

Need

```sql
LIKE '% DIAB1%'
```

or

```sql
REGEXP
```

depending on SQL dialect.

---

## Interview Follow-ups

- Ends with pattern
- Contains pattern
- Starts with any of several prefixes

---

## FAANG Usage

Healthcare

Medical coding

Fraud detection

Customer tagging

---

# Question 4 — Big Countries (Easy)

**LeetCode #595**

Companies

- Microsoft
- Amazon

---

## Problem

Return countries where

```
Area >=3000000

OR

Population>=25000000
```

---

## SQL Solution

```sql
SELECT name,
       population,
       area
FROM World
WHERE area >= 3000000
OR population >= 25000000;
```

---

## Pattern

```
AND

OR

Precedence
```

Always use parentheses when conditions become larger.

---

## Visualization

```
Country

Area?

Population?

Either True

↓

Return
```

---

## Alternative

```sql
WHERE (area>=3000000)
OR (population>=25000000)
```

Cleaner.

---

## Edge Cases

Exactly

```
3000000

25000000
```

Need

>=

not >

---

## Interview Follow-up

Find countries satisfying

Population

AND

GDP

AND

Area

---

## FAANG Usage

Analytics dashboards

Business intelligence

Market segmentation

---

# Question 5 — Customers Who Never Order (Easy)

**LeetCode #183**

Companies

- Amazon
- Meta
- Google

---

## Problem

Customers

| id | name |
|----|------|
|1|Joe|
|2|Henry|

Orders

| id | customerId |
|----|------------|
|1|2|

Return

Joe

---

## Solution 1 (Recommended)

```sql
SELECT name AS Customers
FROM Customers c
WHERE NOT EXISTS (
SELECT 1
FROM Orders o
WHERE c.id=o.customerId
);
```

---

## Why EXISTS?

```
Customer

↓

Search Orders

↓

Found?

↓

No

↓

Return
```

---

## Alternative

```sql
SELECT name
FROM Customers
WHERE id NOT IN
(
SELECT customerId
FROM Orders
);
```

---

## Dangerous Edge Case

Suppose

Orders

```
NULL
```

Then

```
NOT IN

fails
```

This is why

```
NOT EXISTS

is preferred
```

---

## Third Solution

```sql
SELECT c.name
FROM Customers c
LEFT JOIN Orders o
ON c.id=o.customerId
WHERE o.customerId IS NULL;
```

---

## Which is Best?

| Method | Interview Rating |
|----------|-----------------|
| NOT EXISTS | ⭐⭐⭐⭐⭐ |
| LEFT JOIN | ⭐⭐⭐⭐ |
| NOT IN | ⭐⭐ |

---

## FAANG Usage

Users without

Purchases

Payments

Subscriptions

Messages

Activity

---

# Question 6 — Immediate Food Delivery I (Medium Filtering Pattern)

**LeetCode #1173**

Companies

- Uber
- DoorDash
- Amazon

---

## Problem

Immediate order means

```
order_date

=

customer_pref_delivery_date
```

Find percentage of immediate orders.

---

## Solution

```sql
SELECT
ROUND(
100.0 *
SUM(order_date = customer_pref_delivery_date)
/COUNT(*),
2
) AS immediate_percentage
FROM Delivery;
```

---

## Filtering Idea

Instead of WHERE,

use Boolean expression.

```
TRUE

↓

1

FALSE

↓

0
```

Sum counts matches.

---

## Alternative

```sql
SELECT
ROUND(
100.0*
COUNT(
CASE
WHEN order_date=customer_pref_delivery_date
THEN 1
END
)
/COUNT(*),2)
FROM Delivery;
```

---

## Interview Discussion

Filtering can happen

- WHERE
- CASE
- HAVING
- EXISTS
- JOIN condition

Interviewers often convert one into another.

---

## Edge Cases

No rows

Duplicate rows

NULL dates

Timezone conversions

---

## FAANG Usage

Delivery metrics

Same-day delivery

SLA dashboards

Customer satisfaction

---

# Common FAANG Follow-Up Questions

| Question | Skill Tested |
|------------|--------------|
| Rewrite without WHERE | CASE |
| Rewrite using EXISTS | EXISTS |
| Rewrite using JOIN | JOIN |
| Handle NULL safely | NULL |
| Make query index-friendly | Performance |
| Convert NOT IN to NOT EXISTS | Optimization |
| Replace LIKE with REGEXP | Pattern Matching |
| Optimize for millions of rows | Execution Plan |

---

# Pattern Cheat Sheet

| Requirement | SQL Pattern |
|-------------|-------------|
| Equal | `=` |
| Not Equal | `!=` or `<>` |
| Greater | `>` |
| Greater Equal | `>=` |
| Less | `<` |
| Less Equal | `<=` |
| Multiple Conditions | `AND` |
| Either Condition | `OR` |
| Exclude Values | `NOT` |
| Value List | `IN (...)` |
| Exclude List | `NOT IN (...)` |
| Null Check | `IS NULL` |
| Not Null | `IS NOT NULL` |
| Starts With | `LIKE 'abc%'` |
| Ends With | `LIKE '%abc'` |
| Contains | `LIKE '%abc%'` |
| One Character | `LIKE '_'` |
| Exists | `EXISTS` |
| Doesn't Exist | `NOT EXISTS` |
| Safe Missing Rows | `LEFT JOIN ... IS NULL` |

---

# Interview Takeaways

After mastering these six problems, you should be comfortable with:

- Writing efficient `SELECT` statements.
- Combining multiple filtering conditions using `AND`, `OR`, and `NOT`.
- Correctly handling `NULL` values with `IS NULL` and `NOT EXISTS`.
- Using `LIKE` with wildcards for pattern matching.
- Choosing between `NOT IN`, `NOT EXISTS`, and `LEFT JOIN`.
- Applying filtering logic in `WHERE`, `CASE`, and subqueries.
- Recognizing common filtering patterns used in production systems at Google, Amazon, Meta, Microsoft, Apple, Netflix, and similar companies.
