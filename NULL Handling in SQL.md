# NULL Handling in SQL (IS NULL, COALESCE, NULLIF) – FAANG LeetCode Interview Guide

Mastering NULL handling is one of the easiest ways to separate average SQL solutions from interview-quality solutions. Rather than treating `NULL` as a special case after writing the query, strong candidates design queries around it from the beginning.

---

# Table of Contents

- [Question 1 — Customers Who Never Order (Easy)](#question-1--customers-who-never-order-easy)
- [Question 2 — Employee Bonus (Easy)](#question-2--employee-bonus-easy)
- [Question 3 — Immediate Food Delivery I (Medium)](#question-3--immediate-food-delivery-i-medium)
- [Question 4 — Average Selling Price (Medium)](#question-4--average-selling-price-medium)
- [Question 5 — Product Price at a Given Date (Hard)](#question-5--product-price-at-a-given-date-hard)
- [Question 6 — Confirmation Rate (Medium)](#question-6--confirmation-rate-medium)

---

# Question 1 — Customers Who Never Order (Easy)

**LeetCode:** 183

## Problem

Given:

### Customers

| id | name |
|----|------|
|1|Joe|
|2|Henry|
|3|Sam|
|4|Max|

### Orders

| id | customerId |
|----|------------|
|1|3|
|2|1|

Return every customer who has **never placed an order**.

---

## Pattern

> **LEFT JOIN + IS NULL**

This is one of the most common FAANG interview patterns involving NULL.

---

## Solution 1 — LEFT JOIN + IS NULL (Recommended)

```sql
SELECT c.name AS Customers
FROM Customers c
LEFT JOIN Orders o
ON c.id = o.customerId
WHERE o.customerId IS NULL;
```

---

## Execution Flow

```
Customers
     │
LEFT JOIN
     │
Orders
     │
Missing Match?
     │
NULL values created
     │
Filter using IS NULL
     │
Answer
```

---

## Join Result

|Customer|Order|
|---------|-----|
|Joe|2|
|Henry|NULL|
|Sam|1|
|Max|NULL|

Filtering

```sql
WHERE o.customerId IS NULL
```

returns

|Customer|
|---------|
|Henry|
|Max|

---

## Why IS NULL Instead of = NULL?

Incorrect

```sql
WHERE o.customerId = NULL
```

This always evaluates to UNKNOWN.

Correct

```sql
WHERE o.customerId IS NULL
```

Interviewers deliberately test this mistake.

---

## Solution 2 — NOT EXISTS

```sql
SELECT name AS Customers
FROM Customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM Orders o
    WHERE o.customerId = c.id
);
```

---

## Which Solution is Better?

|Method|Performance|Interview Rating|
|-------|-----------|----------------|
|LEFT JOIN + IS NULL|Excellent|★★★★★|
|NOT EXISTS|Excellent|★★★★★|

Most optimizers generate nearly identical execution plans.

---

## Complexity

|Metric|Value|
|------|-----|
|Time|O(n + m)|
|Space|O(1)|

---

## Companies

- Google
- Amazon
- Meta
- Apple
- Microsoft

---

## LLM-Proof Variations

### Variation 1

Return customers with **exactly one** order.

---

### Variation 2

Return customers whose last order was over one year ago.

---

### Variation 3

Return customers with no orders OR only cancelled orders.

---

## Key Takeaways

- NULLs created by LEFT JOIN indicate missing matches.
- Always use `IS NULL`.
- This is one of the highest-frequency SQL interview patterns.
- `NOT EXISTS` is an equally strong alternative.

---

# Question 2 — Employee Bonus (Easy)

**LeetCode:** 577

## Problem

Given two tables:

### Employee

|empId|name|
|-----|----|
|1|Brad|
|2|John|
|3|Dan|

### Bonus

|empId|bonus|
|-----|-----|
|2|500|
|3|2000|

Return employees whose bonus is

- less than 1000
- OR no bonus exists.

---

## Pattern

> LEFT JOIN + NULL filtering + Conditional Logic

---

## Solution 1

```sql
SELECT
    e.name,
    b.bonus
FROM Employee e
LEFT JOIN Bonus b
ON e.empId = b.empId
WHERE b.bonus < 1000
   OR b.bonus IS NULL;
```

---

## Execution Table

After LEFT JOIN

|Employee|Bonus|
|---------|-----|
|Brad|NULL|
|John|500|
|Dan|2000|

Filtering

```
500 < 1000 ✔

NULL ✔

2000 ✘
```

Output

|name|bonus|
|----|-----|
|Brad|NULL|
|John|500|

---

## Solution 2 — COALESCE

Instead of checking NULL explicitly:

```sql
SELECT
    e.name,
    b.bonus
FROM Employee e
LEFT JOIN Bonus b
ON e.empId = b.empId
WHERE COALESCE(b.bonus,0) < 1000;
```

Here

```
NULL

↓

0

↓

0 < 1000

↓

TRUE
```

---

## Why COALESCE Helps

Instead of

```
NULL
```

the database evaluates

```
COALESCE(NULL,0)

↓

0
```

which simplifies predicates.

---

## Complexity

|Metric|Value|
|------|-----|
|Time|O(n+m)|
|Space|O(1)|

---

## Companies

- Meta
- Amazon
- Google
- Bloomberg

---

## LLM-Proof Variations

### Variation 1

Treat missing bonuses as 500.

---

### Variation 2

Return employees with bonus greater than department average, counting NULL as zero.

---

### Variation 3

Replace NULL bonus with "No Bonus".

---

## Key Takeaways

- `COALESCE()` often removes verbose NULL checks.
- LEFT JOIN commonly produces NULL values.
- Know when `IS NULL` is simpler than `COALESCE()`.

---

# Question 3 — Immediate Food Delivery I (Medium)

**LeetCode:** 1173

## Problem

Find the percentage of customers whose **first order** was delivered on the same day they ordered.

Relevant columns:

|customer_id|
|------------|

|order_date|

|customer_pref_delivery_date|

---

## Pattern

> Conditional aggregation with NULL-safe logic

---

## Solution 1

```sql
SELECT
ROUND(
100 *
AVG(
CASE
WHEN order_date = customer_pref_delivery_date
THEN 1
ELSE 0
END
),
2
) AS immediate_percentage
FROM Delivery
WHERE (customer_id, order_date) IN
(
SELECT
customer_id,
MIN(order_date)
FROM Delivery
GROUP BY customer_id
);
```

---

## Why NULL Matters

Suppose

```
customer_pref_delivery_date = NULL
```

Then

```sql
order_date = customer_pref_delivery_date
```

becomes

```
UNKNOWN
```

not FALSE.

The CASE expression safely converts this into

```
ELSE 0
```

preventing incorrect averages.

---

## Alternative Using COALESCE

Some interviewers ask you to treat missing preferred delivery date as order date.

```sql
SELECT
ROUND(
100 *
AVG(
CASE
WHEN order_date =
COALESCE(customer_pref_delivery_date,
order_date)
THEN 1
ELSE 0
END
),
2
)
AS immediate_percentage
FROM Delivery
WHERE (customer_id,order_date) IN
(
SELECT
customer_id,
MIN(order_date)
FROM Delivery
GROUP BY customer_id
);
```

---

## Execution Flow

```
Find First Order
        │
Evaluate CASE
        │
NULL?
        │
COALESCE (optional)
        │
Convert to 1/0
        │
AVG()
        │
Percentage
```

---

## Complexity

|Metric|Value|
|------|-----|
|Time|O(n log n)|
|Space|O(n)|

---

## Companies

- Google
- Uber
- Amazon
- DoorDash
- Meta

---

## Question Archetype

- Conditional aggregation
- CASE with NULL values
- COALESCE for missing data
- First-event analysis

---

## LLM-Proof Variations

### Variation 1

Treat NULL preferred dates as same-day delivery.

---

### Variation 2

Ignore rows containing NULL dates.

---

### Variation 3

Return percentages by month.

---

## Key Takeaways

- Comparisons involving NULL never evaluate to TRUE.
- `CASE` expressions safely normalize NULL outcomes.
- `COALESCE()` is useful when interviewers define default business rules for missing values.
- Aggregate calculations should explicitly account for NULL behavior rather than relying on implicit SQL semantics.

---

# Question 4 — Average Selling Price (Medium)

**LeetCode:** 1251

## Problem

Two tables are given.

### Prices

| product_id | start_date | end_date | price |
|------------|------------|----------|-------|
| 1 | 2019-02-17 | 2019-02-28 | 5 |
| 1 | 2019-03-01 | 2019-03-22 | 20 |
| 2 | 2019-02-01 | 2019-02-20 | 15 |
| 2 | 2019-02-21 | 2019-03-31 | 30 |

### UnitsSold

| product_id | purchase_date | units |
|------------|---------------|-------|
| 1 | 2019-02-25 | 100 |
| 1 | 2019-03-01 | 15 |
| 2 | 2019-02-10 | 200 |
| 2 | 2019-03-22 | 30 |

Calculate the **average selling price** of each product.

If a product has **no sales**, return **0** as its average price.

---

## Pattern

> Aggregation + LEFT JOIN + COALESCE + NULL-safe calculations

---

## Solution 1 — LEFT JOIN + COALESCE

```sql
SELECT
    p.product_id,
    ROUND(
        COALESCE(
            SUM(u.units * p.price) / SUM(u.units),
            0
        ),
        2
    ) AS average_price
FROM Prices p
LEFT JOIN UnitsSold u
ON p.product_id = u.product_id
AND u.purchase_date
BETWEEN p.start_date AND p.end_date
GROUP BY p.product_id;
```

---

## Execution Flow

```
Prices
      │
LEFT JOIN
      │
UnitsSold
      │
Date Filter
      │
Weighted Sum
      │
Total Units
      │
Divide
      │
NULL?
      │
COALESCE(...,0)
```

---

## Why COALESCE Is Necessary

Without sales:

```
SUM(units * price)

↓

NULL
```

Similarly,

```
SUM(units)

↓

NULL
```

Result:

```
NULL / NULL

↓

NULL
```

Business requirement:

```
Return 0
```

Therefore,

```sql
COALESCE(expression,0)
```

is mandatory.

---

## Solution 2 — CASE

```sql
SELECT
    p.product_id,
    ROUND(
        CASE
            WHEN SUM(u.units) IS NULL THEN 0
            ELSE SUM(u.units * p.price) /
                 SUM(u.units)
        END,
        2
    ) AS average_price
FROM Prices p
LEFT JOIN UnitsSold u
ON p.product_id = u.product_id
AND u.purchase_date BETWEEN p.start_date
                        AND p.end_date
GROUP BY p.product_id;
```

---

## Execution Example

|Product|Revenue|Units|Average|
|-------|-------|-----|--------|
|1|800|115|6.96|
|2|3900|230|16.96|
|3|NULL|NULL|0|

---

## Complexity

|Metric|Value|
|------|-----|
|Time|O(n log n)|
|Space|O(1)|

---

## Companies

- Amazon
- Google
- Apple
- Meta
- Airbnb

---

## Question Archetype

- Aggregation with NULL values
- Weighted averages
- LEFT JOIN producing NULL rows
- Business default values

---

## LLM-Proof Variations

### Variation 1

Return **NULL** instead of 0 for products without sales.

---

### Variation 2

Ignore transactions with NULL unit counts.

---

### Variation 3

Compute monthly weighted average prices.

---

## Key Takeaways

- Aggregate functions often return NULL when no matching rows exist.
- `COALESCE()` is cleaner than wrapping every aggregate with `CASE`.
- Always check interview requirements for default values.

---

# Question 5 — Product Price at a Given Date (Hard)

**LeetCode:** 1164

## Problem

Every product initially costs **10**.

The **Products** table records price changes.

Return the product price on **2019-08-16**.

If no update exists before that date, return **10**.

---

## Pattern

> Missing historical rows + COALESCE + NULL replacement

---

## Solution 1

```sql
SELECT
    p.product_id,
    COALESCE(
        (
            SELECT new_price
            FROM Products p2
            WHERE p2.product_id = p.product_id
            AND change_date <= '2019-08-16'
            ORDER BY change_date DESC
            LIMIT 1
        ),
        10
    ) AS price
FROM Products p
GROUP BY p.product_id;
```

---

## Why COALESCE?

Suppose product 5

```
No updates
```

Subquery returns

```
NULL
```

Business rule

```
Default = 10
```

Therefore

```
COALESCE(NULL,10)

↓

10
```

---

## Alternative Solution

```sql
WITH LatestPrice AS
(
SELECT
product_id,
new_price,
ROW_NUMBER() OVER(
PARTITION BY product_id
ORDER BY change_date DESC
) rn
FROM Products
WHERE change_date <= '2019-08-16'
)

SELECT
p.product_id,
COALESCE(l.new_price,10) AS price
FROM
(
SELECT DISTINCT product_id
FROM Products
)p
LEFT JOIN LatestPrice l
ON p.product_id=l.product_id
AND l.rn=1;
```

---

## Execution Diagram

```
Price History

↓

Latest Record Before Date

↓

Found?

├── Yes → Price
└── No → NULL

↓

COALESCE

↓

10
```

---

## Complexity

|Metric|Value|
|------|-----|
|Time|O(n log n)|
|Space|O(n)|

---

## Companies

- Google
- Meta
- Amazon
- Stripe
- Apple

---

## Question Archetype

- Historical snapshot
- Missing history
- Default values
- Window functions + NULL handling

---

## LLM-Proof Variations

### Variation 1

Default price is 100 instead of 10.

---

### Variation 2

Return the latest price before every transaction date.

---

### Variation 3

Return prices only for active products.

---

## Key Takeaways

- `COALESCE()` frequently appears after scalar subqueries.
- Window functions naturally pair with NULL replacement.
- Historical lookup questions almost always include missing-history edge cases.

---

# Question 6 — Confirmation Rate (Medium)

**LeetCode:** 1934

## Problem

Given:

### Signups

| user_id | time_stamp |
|---------|------------|

### Confirmations

| user_id | action |
|---------|---------|
|3|confirmed|
|3|timeout|
|5|confirmed|

Return each user's **confirmation rate**.

Users with **no confirmations** should have a rate of **0.00**.

---

## Pattern

> LEFT JOIN + Conditional Aggregation + NULLIF + COALESCE

This problem is one of the best demonstrations of combining all three NULL-handling functions:

- `IS NULL`
- `COALESCE`
- `NULLIF`

---

## Solution

```sql
SELECT
    s.user_id,
    ROUND(
        COALESCE(
            AVG(
                CASE
                    WHEN c.action = 'confirmed'
                    THEN 1
                    ELSE 0
                END
            ),
            0
        ),
        2
    ) AS confirmation_rate
FROM Signups s
LEFT JOIN Confirmations c
ON s.user_id = c.user_id
GROUP BY s.user_id;
```

---

## Alternative Using NULLIF

Another interview-friendly solution avoids division by zero explicitly.

```sql
SELECT
    s.user_id,
    ROUND(
        COALESCE(
            SUM(
                CASE
                    WHEN c.action='confirmed'
                    THEN 1
                    ELSE 0
                END
            ) /
            NULLIF(COUNT(c.action),0),
            0
        ),
        2
    ) AS confirmation_rate
FROM Signups s
LEFT JOIN Confirmations c
ON s.user_id=s.user_id
GROUP BY s.user_id;
```

---

## Why NULLIF?

Suppose

```
COUNT(action)

↓

0
```

Without NULLIF

```
confirmed / 0

↓

Division Error
```

Using

```sql
NULLIF(COUNT(action),0)
```

becomes

```
NULL
```

Then

```
COALESCE(NULL,0)

↓

0
```

No runtime error.

---

## Execution Flow

```
LEFT JOIN

↓

Count Confirmed

↓

Count Total

↓

NULLIF(total,0)

↓

Safe Division

↓

COALESCE(...,0)

↓

Final Rate
```

---

## Complexity

|Metric|Value|
|------|-----|
|Time|O(n log n)|
|Space|O(1)|

---

## Companies

- Meta
- Google
- Amazon
- Microsoft
- Uber

---

## Question Archetype

- Conditional aggregation
- Division-by-zero prevention
- LEFT JOIN with missing rows
- NULL-safe arithmetic

---

## LLM-Proof Variations

### Variation 1

Return the confirmation rate by month.

---

### Variation 2

Ignore timeout actions older than 30 days.

---

### Variation 3

Calculate confirmation rates separately for premium and free users.

---

## Key Takeaways

- `NULLIF()` is the standard SQL technique for preventing divide-by-zero errors.
- `COALESCE()` converts NULL arithmetic results into business-friendly defaults.
- `AVG(CASE...)` is usually the shortest solution, while `SUM()/NULLIF(COUNT(),0)` demonstrates stronger interview knowledge.
- Many FAANG interview questions combine `LEFT JOIN`, conditional aggregation, and NULL-safe calculations in a single query.

---

# Final Revision Checklist

| NULL Technique | Where It Appears | Common Interview Use |
|----------------|------------------|----------------------|
| `IS NULL` | LEFT JOIN filtering | Find missing records |
| `IS NOT NULL` | Data validation | Ignore incomplete rows |
| `COALESCE()` | Aggregation, joins, scalar subqueries | Replace missing values with defaults |
| `NULLIF()` | Division, comparisons | Prevent divide-by-zero |
| `CASE ... IS NULL` | Business rules | Conditional NULL handling |
| `LEFT JOIN + IS NULL` | Anti-joins | "Never ordered", "No activity" questions |
| `COALESCE(SUM(...),0)` | Reporting | Default aggregate values |
| `NULLIF(COUNT(...),0)` | Ratios | Safe percentage calculations |

---

## FAANG Interview Summary

NULL handling appears across nearly every SQL interview category:

| Archetype | Primary Function |
|-----------|------------------|
| Missing JOIN records | `IS NULL` |
| Default values | `COALESCE()` |
| Divide-by-zero | `NULLIF()` |
| Historical snapshots | `COALESCE()` |
| Aggregation reports | `COALESCE(SUM())` |
| Ratios & percentages | `NULLIF()` + `COALESCE()` |
| Conditional logic | `CASE` + NULL handling |

A strong interview solution is not just syntactically correct—it explicitly accounts for NULL behavior, preventing incorrect results and runtime errors while matching business requirements.


