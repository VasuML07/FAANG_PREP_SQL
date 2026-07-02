# Pivoting Data (CASE + GROUP BY) in SQL — LeetCode Prep for FAANG Interviews

> A focused interview guide for mastering **Pivoting Data** using **CASE**, **GROUP BY**, **Conditional Aggregation**, and related SQL patterns through real LeetCode SQL questions.

---

# Table of Contents

- [Problem 1 — Reformat Department Table (Easy)](#problem-1--reformat-department-table-easy)
- [Problem 2 — Monthly Transactions I (Medium)](#problem-2--monthly-transactions-i-medium)
- [Problem 3 — Immediate Food Delivery II (Medium)](#problem-3--immediate-food-delivery-ii-medium)
- [Problem 4 — Restaurant Growth (Medium)](#problem-4--restaurant-growth-medium)
- [Problem 5 — Capital Gain/Loss (Medium)](#problem-5--capital-gainloss-medium)
- [Problem 6 — Product Sales Analysis III (Hard Pattern)](#problem-6--product-sales-analysis-iii-hard-pattern)
- [Techniques & Algorithms](#techniques--algorithms)
- [LLM-Proof Questions](#llm-proof-questions)
- [Interview Cheat Sheet](#interview-cheat-sheet)
- [Common Mistakes](#common-mistakes)
- [Pattern Summary](#pattern-summary)

---

# Problem 1 — Reformat Department Table (Easy)

**LeetCode:** 1179

**Difficulty:** Easy

**Companies**

- Amazon
- Google
- Microsoft
- Apple
- Bloomberg

---

## Problem Statement

You are given a table containing monthly revenues.

| id | revenue | month |
|----|----------|--------|
|1|8000|Jan|
|1|9000|Feb|
|1|10000|Mar|
|2|5000|Jan|
|2|6000|Mar|

Return

|id|Jan_Revenue|Feb_Revenue|Mar_Revenue|...|
|--|-----------|-----------|-----------|---|

Each month becomes a separate column.

This is the classic SQL pivot problem.

---

# Example

## Input

|id|month|revenue|
|--|-----|--------|
|1|Jan|8000|
|1|Feb|9000|
|1|Mar|10000|
|2|Jan|5000|
|2|Mar|6000|

---

## Desired Output

|id|Jan_Revenue|Feb_Revenue|Mar_Revenue|
|--|-----------|-----------|-----------|
|1|8000|9000|10000|
|2|5000|NULL|6000|

---

# Visualization

Before Pivot

```
ID
│
├── Jan → 8000
├── Feb → 9000
└── Mar →10000
```

After Pivot

```
ID | Jan | Feb | Mar
----------------------
1  |8000 |9000 |10000
```

---

# Key Insight

Instead of using SQL's native `PIVOT` operator (not supported everywhere), convert rows into columns using

```
CASE
+
MAX()
+
GROUP BY
```

Each CASE only keeps one month's value.

MAX simply returns that value because only one row satisfies the condition.

---

# Solution Approach

### Step 1

Group all rows belonging to the same employee.

```
GROUP BY id
```

---

### Step 2

Create one CASE expression for every month.

Example

```
CASE
WHEN month='Jan'
THEN revenue
END
```

Rows that aren't January become NULL.

---

### Step 3

Aggregate

```
MAX(...)
```

Since only one January revenue exists,

```
MAX(NULL,NULL,8000,NULL)
```

returns

```
8000
```

---

### Step 4

Repeat for every month.

---

# Query

```sql
SELECT
    id,

    MAX(
        CASE
            WHEN month = 'Jan'
            THEN revenue
        END
    ) AS Jan_Revenue,

    MAX(
        CASE
            WHEN month = 'Feb'
            THEN revenue
        END
    ) AS Feb_Revenue,

    MAX(
        CASE
            WHEN month = 'Mar'
            THEN revenue
        END
    ) AS Mar_Revenue,

    MAX(
        CASE
            WHEN month = 'Apr'
            THEN revenue
        END
    ) AS Apr_Revenue,

    MAX(
        CASE
            WHEN month = 'May'
            THEN revenue
        END
    ) AS May_Revenue,

    MAX(
        CASE
            WHEN month = 'Jun'
            THEN revenue
        END
    ) AS Jun_Revenue,

    MAX(
        CASE
            WHEN month = 'Jul'
            THEN revenue
        END
    ) AS Jul_Revenue,

    MAX(
        CASE
            WHEN month = 'Aug'
            THEN revenue
        END
    ) AS Aug_Revenue,

    MAX(
        CASE
            WHEN month = 'Sep'
            THEN revenue
        END
    ) AS Sep_Revenue,

    MAX(
        CASE
            WHEN month = 'Oct'
            THEN revenue
        END
    ) AS Oct_Revenue,

    MAX(
        CASE
            WHEN month = 'Nov'
            THEN revenue
        END
    ) AS Nov_Revenue,

    MAX(
        CASE
            WHEN month = 'Dec'
            THEN revenue
        END
    ) AS Dec_Revenue

FROM Department

GROUP BY id;
```

---

# Query Execution Flow

```
Department

↓

GROUP BY id

↓

For each month

↓

CASE keeps matching revenue

↓

MAX removes NULLs

↓

One row per employee
```

---

# Why MAX Instead of SUM?

Both work because each employee has at most one revenue per month.

```
MAX(NULL,NULL,7000)=7000

SUM(NULL,NULL,7000)=7000
```

Interviewers generally expect MAX.

---

# Time Complexity

```
O(N)
```

One scan through the table.

---

# Space Complexity

```
O(1)
```

Ignoring output.

---

# Common Pitfalls

### Mistake 1

Using CASE outside aggregate

Incorrect

```sql
CASE WHEN month='Jan'
THEN revenue
END
```

Produces multiple rows.

Need

```sql
MAX(
CASE ...
END)
```

---

### Mistake 2

Grouping by month also

```sql
GROUP BY
id,
month
```

Now every month becomes its own row.

No pivot occurs.

---

### Mistake 3

Using WHERE

Wrong

```sql
WHERE month='Jan'
```

Removes all other months.

---

# Interview Takeaway

Whenever asked

> Convert rows into columns

Immediately think

```
GROUP BY

+

CASE

+

MAX
```

This is probably the most frequently tested SQL pivot pattern.

---

# Problem 2 — Monthly Transactions I (Medium)

**LeetCode:** 1193

**Difficulty:** Medium

**Companies**

- Amazon
- Uber
- Microsoft
- TikTok
- Bloomberg

---

## Problem Statement

Given a transactions table,

Return for every

- month
- country

Calculate

- transaction count
- approved count
- total amount
- approved amount

---

## Example

Input

|id|country|state|amount|trans_date|
|--|-------|-----|------|----------|
|1|US|approved|100|2019-01-01|
|2|US|declined|200|2019-01-15|
|3|India|approved|400|2019-01-20|
|4|India|approved|100|2019-02-01|

---

Desired Output

|month|country|trans_count|approved_count|trans_total_amount|approved_total_amount|
|------|-------|-----------|--------------|------------------|---------------------|
|2019-01|US|2|1|300|100|
|2019-01|India|1|1|400|400|
|2019-02|India|1|1|100|100|

---

# Visualization

Original

```
Month Country State Amount

↓

Need

Month Country

↓

Count
Approved Count
Total Amount
Approved Amount
```

Notice

This is **not** a row-to-column pivot.

Instead,

we pivot **conditions into aggregated metrics**.

Each CASE creates one metric.

---

# Key Insight

Every output column is simply

```
Aggregate

+

CASE
```

Think

```
COUNT Approved

↓

SUM(CASE...)

Total Approved Amount

↓

SUM(CASE...)
```

Instead of filtering rows multiple times.

---

# Solution Approach

### Step 1

Extract month

```sql
DATE_FORMAT(trans_date,'%Y-%m')
```

---

### Step 2

Group

```
Month

+

Country
```

---

### Step 3

Compute transaction count

```
COUNT(*)
```

---

### Step 4

Compute approved count

```
SUM(
CASE
WHEN state='approved'
THEN 1
ELSE 0
END)
```

---

### Step 5

Compute approved amount

```
SUM(
CASE
WHEN approved
THEN amount
ELSE 0
END)
```

---

# Query

```sql
SELECT

DATE_FORMAT(trans_date,'%Y-%m') AS month,

country,

COUNT(*) AS trans_count,

SUM(
CASE
WHEN state='approved'
THEN 1
ELSE 0
END
) AS approved_count,

SUM(amount) AS trans_total_amount,

SUM(
CASE
WHEN state='approved'
THEN amount
ELSE 0
END
) AS approved_total_amount

FROM Transactions

GROUP BY
DATE_FORMAT(trans_date,'%Y-%m'),
country;
```

---

# Execution Flow

```
Transactions

↓

Extract Month

↓

Group By

Month
Country

↓

COUNT(*)

↓

SUM(CASE...)

↓

Final Metrics
```

---

# Why This Is a Pivot Problem

Instead of

```
Approved
Declined
```

being rows,

they become

```
approved_count

approved_total_amount
```

Columns created using CASE.

This is called **conditional aggregation**, one of the most common pivoting techniques in SQL.

---

# Alternative (Less Efficient)

Some candidates write separate subqueries:

```sql
SELECT ...

JOIN

SELECT ...
```

Interviewers generally prefer a **single pass** solution using conditional aggregation because it scans the table only once.

---

# Complexity

Time

```
O(N)
```

Space

```
O(1)
```

---

# Common Pitfalls

### Forgetting ELSE 0

Wrong

```sql
SUM(
CASE
WHEN state='approved'
THEN amount
END)
```

Although SUM ignores NULLs, using `ELSE 0` makes intent explicit and avoids surprises when adapting the pattern to other aggregates.

---

### Grouping by Full Date

Incorrect

```sql
GROUP BY trans_date
```

Produces daily results instead of monthly results.

---

### Using COUNT(CASE...)

Many candidates write

```sql
COUNT(
CASE
WHEN state='approved'
THEN 1
END)
```

This works because `COUNT` ignores NULLs, but `SUM(CASE...)` is generally clearer and scales better for more complex conditional metrics.

---

# Why Interviewers Like This Problem

It evaluates whether you can:

- derive grouping keys from dates
- build multiple metrics in a single query
- replace multiple scans with conditional aggregation
- recognize when CASE belongs inside an aggregate rather than in the SELECT list

---


# Problem 3 — Immediate Food Delivery II (Medium)

**LeetCode:** 1174

**Difficulty:** Medium

**Companies**

- Amazon
- Google
- DoorDash
- Uber
- ByteDance

---

## Problem Statement

A food delivery is considered **immediate** if the customer's preferred delivery date is the same as the order date.

For each customer, consider **only their first order**.

Return the percentage of customers whose first order was immediate.

Although this isn't a traditional row-to-column pivot, it is a classic **conditional aggregation** problem where a condition is converted into a summary metric.

---

# Example

## Input

|delivery_id|customer_id|order_date|customer_pref_delivery_date|
|------------|-----------|----------|---------------------------|
|1|1|2019-08-01|2019-08-02|
|2|2|2019-08-02|2019-08-02|
|3|1|2019-08-11|2019-08-11|
|4|3|2019-08-24|2019-08-24|

---

## First Orders

|customer|first_order|immediate?|
|---------|-----------|----------|
|1|2019-08-01|No|
|2|2019-08-02|Yes|
|3|2019-08-24|Yes|

---

## Output

```
66.67
```

---

# Visualization

Original Table

```
Customer

↓

Many Orders

↓

Need Only

First Order

↓

CASE

↓

Immediate?

↓

Percentage
```

---

# Key Insight

The problem has **two stages**:

1. Find each customer's first order.
2. Convert the immediate/non-immediate condition into numbers using CASE.

This is another example of conditional aggregation.

---

# Solution Approach

### Step 1

Find the earliest order date for every customer.

```
customer_id

↓

MIN(order_date)
```

---

### Step 2

Join back to retrieve the complete first-order record.

---

### Step 3

Evaluate

```
order_date

=

preferred_date
```

---

### Step 4

Convert

```
TRUE → 1

FALSE → 0
```

---

### Step 5

Average them.

Average of 1s and 0s gives the percentage.

---

# Query

```sql
SELECT

ROUND(

AVG(

CASE

WHEN d.order_date =
     d.customer_pref_delivery_date

THEN 1

ELSE 0

END

)*100,2

) AS immediate_percentage

FROM Delivery d

JOIN

(

SELECT

customer_id,

MIN(order_date) AS first_order

FROM Delivery

GROUP BY customer_id

) f

ON d.customer_id=f.customer_id

AND d.order_date=f.first_order;
```

---

# Execution Flow

```
Delivery

↓

GROUP BY customer

↓

MIN(order_date)

↓

Join

↓

CASE

↓

AVG

↓

Percentage
```

---

# Why AVG Works

Suppose

```
1
1
0
1
0
```

Average

```
(1+1+0+1+0)/5

=

0.60
```

Multiply by 100.

```
60%
```

This trick appears frequently in SQL interviews.

---

# Alternative Using Window Functions

```sql
WITH first_orders AS
(
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY customer_id
ORDER BY order_date
) rn
FROM Delivery
)

SELECT
ROUND(
AVG(
CASE
WHEN order_date=
customer_pref_delivery_date
THEN 1
ELSE 0
END
)*100,2
)
FROM first_orders
WHERE rn=1;
```

Interviewers appreciate this solution because it generalizes well when "first" becomes "second", "latest", or "top N".

---

# Complexity

Time

```
O(N log N)
```

(if sorting is required)

Window-function implementations generally require sorting by customer.

---

# Common Pitfalls

### Comparing Every Order

Many candidates forget

```
ONLY FIRST ORDER
```

instead of

```
ALL ORDERS
```

---

### Using COUNT Instead of AVG

This becomes unnecessarily complicated.

Average naturally computes

```
Successful

/

Total
```

---

### Forgetting ROUND()

The problem expects

```
66.67
```

instead of

```
66.6666666
```

---

# Why Interviewers Like This Problem

This question checks whether you can combine:

- grouping
- joins (or window functions)
- conditional aggregation
- percentage calculations

into a compact solution.

---

# Problem 4 — Restaurant Growth (Medium)

**LeetCode:** 1321

**Difficulty:** Medium

**Companies**

- Amazon
- Google
- Microsoft
- Meta
- Apple

---

## Problem Statement

For every day beginning with the **7th day**, report:

- total revenue over the previous seven days
- average daily revenue over those seven days

The result should be ordered by date.

Although this is primarily a **window-function** problem, interviews often extend it by asking candidates to create multiple conditional metrics (weekday vs. weekend revenue, holiday revenue, etc.) using CASE inside window aggregates.

---

# Example

Daily Revenue

|Date|Revenue|
|----|-------|
|1|100|
|2|200|
|3|150|
|4|300|
|5|250|
|6|180|
|7|170|

Output

|Date|Amount|Average|
|----|------|-------|
|7|1350|192.86|

---

# Visualization

```
Day1

↓

Day2

↓

Day3

↓

Day4

↓

Day5

↓

Day6

↓

Day7

↓

Rolling Window

↓

SUM

AVG
```

---

# Key Insight

Instead of grouping,

we create a **moving window**.

Then, interview follow-ups frequently ask for pivoted metrics such as:

```
Weekend Revenue

Weekday Revenue

Holiday Revenue
```

These are built with

```
SUM(

CASE ...

)

OVER(...)
```

The pivot technique remains exactly the same.

---

# Solution Approach

### Step 1

Aggregate revenue by day.

The table may contain multiple customers per date.

---

### Step 2

Apply a 7-day rolling window.

---

### Step 3

Compute

```
SUM(amount)
```

---

### Step 4

Compute

```
AVG(amount)
```

over the same frame.

---

# Query

```sql
WITH daily_sales AS
(
SELECT

visited_on,

SUM(amount) AS amount

FROM Customer

GROUP BY visited_on
)

SELECT

visited_on,

SUM(amount)

OVER(

ORDER BY visited_on

ROWS BETWEEN 6 PRECEDING
AND CURRENT ROW

) AS amount,

ROUND(

AVG(amount)

OVER(

ORDER BY visited_on

ROWS BETWEEN 6 PRECEDING
AND CURRENT ROW

),2

) AS average_amount

FROM daily_sales

LIMIT 1000000
OFFSET 6;
```

---

# Query Execution Flow

```
Customer

↓

GROUP BY Date

↓

Daily Revenue

↓

Window

Previous 6 Days

+

Current Day

↓

SUM

AVG

↓

Output
```

---

# Interview Follow-up (Pivot Extension)

Suppose you're also asked for:

- weekend revenue
- weekday revenue

The rolling window becomes:

```sql
SUM(

CASE

WHEN DAYOFWEEK(visited_on)

IN (1,7)

THEN amount

ELSE 0

END

)

OVER(

ORDER BY visited_on

ROWS BETWEEN 6 PRECEDING
AND CURRENT ROW

)
```

This combines **window functions** with **conditional aggregation**, a common senior-level interview pattern.

---

# Complexity

Time

```
O(N log N)
```

Sorting is required for window evaluation.

---

# Common Pitfalls

### Forgetting Daily Aggregation

Multiple customers can visit on the same day.

Running the window directly on raw rows produces incorrect totals.

---

### Using RANGE Instead of ROWS

`ROWS BETWEEN 6 PRECEDING` counts exactly seven rows.

`RANGE` behaves differently when duplicate ordering values exist.

---

### Off-by-One Window

Many candidates accidentally write

```
5 PRECEDING
```

or

```
7 PRECEDING
```

instead of

```
6 PRECEDING
```

which changes the window size.

---

# Why Interviewers Like This Problem

It tests:

- window functions
- rolling aggregates
- ordering semantics
- follow-up conditional aggregation (pivoted metrics) using CASE

---


# Problem 5 — Capital Gain/Loss (Medium)

**LeetCode:** 1393

**Difficulty:** Medium

**Companies**

- Amazon
- Google
- Bloomberg
- Two Sigma
- Jane Street

---

## Problem Statement

You are given a stock transactions table.

Each transaction is either:

- Buy
- Sell

For every stock, compute the **capital gain or loss**.

A sell contributes positively to profit, while a buy contributes negatively.

---

## Example

### Input

|stock_name|operation|price|
|-----------|---------|-----|
|AAPL|Buy|1000|
|AAPL|Sell|1200|
|AAPL|Buy|500|
|AAPL|Sell|900|
|TSLA|Buy|300|
|TSLA|Sell|450|

---

### Output

|stock_name|capital_gain_loss|
|-----------|-----------------|
|AAPL|600|
|TSLA|150|

---

# Visualization

Original

```
Stock

↓

Buy

Sell

Buy

Sell
```

After Conditional Pivot

```
Buy  → Negative

Sell → Positive

↓

SUM()

↓

Net Profit
```

Unlike row-to-column pivots, this is a **value-to-metric pivot** where categories become separate arithmetic behavior.

---

# Key Insight

Transform transaction types into signed numbers.

```
Buy

↓

-price

Sell

↓

+price
```

Then simply calculate

```
SUM()
```

This avoids writing two separate aggregations.

---

# Solution Approach

### Step 1

Group transactions by stock.

```
GROUP BY stock_name
```

---

### Step 2

Convert operations into values.

```
CASE

Buy

↓

-price

Sell

↓

price
```

---

### Step 3

Sum everything.

The final result automatically becomes

```
Total Sell

-

Total Buy
```

---

# Query

```sql
SELECT

stock_name,

SUM(

CASE

WHEN operation='Buy'

THEN -price

ELSE price

END

) AS capital_gain_loss

FROM Stocks

GROUP BY stock_name;
```

---

# Query Execution Flow

```
Stocks

↓

CASE

Buy

↓

Negative

Sell

↓

Positive

↓

SUM

↓

Capital Gain/Loss
```

---

# Dry Run

### Original Rows

|Stock|Operation|Price|
|------|----------|----|
|AAPL|Buy|1000|
|AAPL|Sell|1200|
|AAPL|Buy|500|
|AAPL|Sell|900|

---

### After CASE

|Value|
|------|
|-1000|
|1200|
|-500|
|900|

---

### SUM

```
-1000

+

1200

+

-500

+

900

=

600
```

---

# Alternative Solution

Some candidates attempt

```sql
SUM(CASE...)

-

SUM(CASE...)
```

Example

```sql
SUM(
CASE
WHEN operation='Sell'
THEN price
ELSE 0
END
)

-

SUM(
CASE
WHEN operation='Buy'
THEN price
ELSE 0
END
)
```

This is correct but scans two CASE expressions instead of one.

The signed-value approach is simpler and easier to maintain.

---

# Complexity

Time

```
O(N)
```

Space

```
O(1)
```

---

# Common Pitfalls

### Forgetting Negative Sign

Wrong

```sql
CASE

WHEN operation='Buy'

THEN price
```

Both operations become positive.

Profit becomes incorrect.

---

### Grouping by Operation

Incorrect

```sql
GROUP BY

stock_name,

operation
```

Now buys and sells appear on different rows.

No final profit is produced.

---

### Assuming Buy Always Comes Before Sell

The solution does **not** rely on transaction order.

Every row is independently transformed into a signed value.

---

# Why Interviewers Like This Problem

This question evaluates whether candidates recognize that:

- CASE can transform values before aggregation.
- Multiple conditional aggregates can often be replaced by a single arithmetic transformation.
- Conditional aggregation isn't limited to counts—it also applies to financial calculations.

---

# Problem 6 — Product Sales Analysis III (Hard Pattern)

**LeetCode:** 1070

**Difficulty:** Easy (commonly used as a Medium/Hard interview follow-up)

**Companies**

- Amazon
- Google
- Meta
- Microsoft
- Apple

> Although the original LeetCode problem is Easy, interviewers frequently extend it into a significantly harder pivoting problem by requesting additional conditional metrics in the same query.

---

## Problem Statement

For every product, return information about its **first year of sale**.

The base problem asks for:

- product_id
- first_year
- quantity
- price

Interview follow-ups commonly ask candidates to additionally compute:

- first-year revenue
- later-year revenue
- total revenue
- first-year sales count
- later-year sales count

using a **single grouped query** with CASE.

---

## Example

### Input

|product_id|year|quantity|price|
|-----------|----|--------|-----|
|1|2018|10|5|
|1|2019|8|7|
|1|2020|6|8|
|2|2019|12|4|
|2|2020|15|5|

---

### Desired Interview Follow-up Output

|product_id|first_year_revenue|later_revenue|
|-----------|-----------------|-------------|
|1|50|104|
|2|48|75|

---

# Visualization

Original

```
Product

↓

2018

↓

2019

↓

2020
```

Pivoted Metrics

```
CASE

↓

First Year Revenue

Later Revenue

Total Revenue
```

Instead of creating new rows, we create new **summary columns**.

---

# Key Insight

First determine each product's earliest year.

Then classify every row into one of two buckets:

```
First Year

Later Years
```

using CASE.

Finally aggregate.

---

# Solution Approach

### Step 1

Find the earliest year.

```sql
MIN(year)
```

per product.

---

### Step 2

Join back.

Each sale now knows whether it belongs to

```
First Year

or

Later Years.
```

---

### Step 3

Compute

```
quantity × price
```

---

### Step 4

Use conditional aggregation.

```
SUM(

CASE

WHEN year=first_year

THEN revenue

ELSE 0

END

)
```

---

# Extended Interview Query

```sql
WITH first_year AS
(
SELECT

product_id,

MIN(year) AS first_year

FROM Sales

GROUP BY product_id
)

SELECT

s.product_id,

SUM(

CASE

WHEN s.year=f.first_year

THEN s.quantity*s.price

ELSE 0

END

) AS first_year_revenue,

SUM(

CASE

WHEN s.year>f.first_year

THEN s.quantity*s.price

ELSE 0

END

) AS later_year_revenue,

SUM(s.quantity*s.price)

AS total_revenue

FROM Sales s

JOIN first_year f

ON s.product_id=f.product_id

GROUP BY s.product_id;
```

---

# Execution Flow

```
Sales

↓

MIN(year)

↓

Join

↓

CASE

↓

Revenue Buckets

↓

SUM

↓

Final Pivoted Metrics
```

---

# Why This Is an Excellent Interview Extension

The original LeetCode problem is straightforward.

The follow-up tests whether you can:

- derive categories dynamically
- classify rows using CASE
- generate multiple metrics in one grouped query
- avoid multiple passes over the same data

This pattern appears frequently in data warehouse and analytics interviews.

---

# Complexity

Time

```
O(N)
```

Space

```
O(1)
```

(excluding grouped output)

---

# Common Pitfalls

### Using WHERE Instead of CASE

Wrong

```sql
WHERE year=first_year
```

This removes later-year rows completely, making it impossible to compute both metrics in one query.

---

### Multiple Table Scans

Many candidates write one query for first-year revenue and another for later-year revenue.

Conditional aggregation computes both simultaneously.

---

### Missing ELSE 0

While `SUM` ignores NULLs, explicit `ELSE 0` improves readability and makes the intent clear, especially when additional arithmetic is added later.

---

# Interview Takeaway

When interviewers ask for:

- first vs. later
- active vs. inactive
- domestic vs. international
- weekday vs. weekend
- approved vs. rejected

the pattern is almost always:

```
GROUP BY

↓

CASE

↓

SUM / COUNT / AVG / MAX

↓

One row with multiple metrics
```

---


# Techniques & Algorithms

This section summarizes **only the techniques used in the six problems above**.

---

# Technique 1 — Conditional Aggregation (The Most Important Pattern)

Appears in

- Problem 1
- Problem 2
- Problem 3
- Problem 5
- Problem 6

Pattern

```sql
SUM(
    CASE
        WHEN condition
        THEN value
        ELSE 0
    END
)
```

or

```sql
COUNT(
    CASE
        WHEN condition
        THEN 1
    END
)
```

or

```sql
MAX(
    CASE
        WHEN condition
        THEN value
    END
)
```

---

## Mental Model

```
Rows

↓

CASE classifies rows

↓

Aggregate combines them

↓

Columns appear
```

Example

Input

|Status|Amount|
|------|------|
|Approved|100|
|Rejected|200|
|Approved|150|

Output

|Approved Amount|Rejected Amount|
|---------------|---------------|
|250|200|

SQL

```sql
SELECT

SUM(
CASE
WHEN status='Approved'
THEN amount
ELSE 0
END
) AS approved,

SUM(
CASE
WHEN status='Rejected'
THEN amount
ELSE 0
END
) AS rejected

FROM Orders;
```

---

# Technique 2 — Row-to-Column Pivot

Appears in

- Problem 1

Pattern

```sql
MAX(
CASE
WHEN category='A'
THEN value
END
)
```

Visual

Before

```
Employee

↓

Jan

↓

Feb

↓

Mar
```

After

```
Employee Jan Feb Mar
```

---

## Why MAX?

Only one row satisfies the CASE.

Example

```
NULL

NULL

500

NULL
```

```
MAX(...)

↓

500
```

---

# Technique 3 — Building Multiple Metrics in One Scan

Appears in

- Problem 2
- Problem 5
- Problem 6

Instead of

```sql
SELECT ...
```

five times,

compute everything together.

Example

```sql
SELECT

SUM(amount),

COUNT(*),

SUM(
CASE
WHEN approved
THEN amount
ELSE 0
END
),

COUNT(
CASE
WHEN approved
THEN 1
END
)

FROM Transactions;
```

One table scan.

Four metrics.

---

# Technique 4 — Dynamic Classification

Appears in

- Problem 3
- Problem 6

First derive a category.

Then aggregate.

Example

```
First Order

↓

Immediate?

↓

CASE

↓

AVG
```

or

```
First Year

↓

Later Years

↓

CASE

↓

SUM
```

General template

```sql
CASE

WHEN derived_condition

THEN ...

ELSE ...

END
```

---

# Technique 5 — CASE Inside Window Functions

Appears in

- Problem 4 (Interview Follow-up)

Pattern

```sql
SUM(

CASE

WHEN condition

THEN amount

ELSE 0

END

)

OVER(

PARTITION BY ...

ORDER BY ...

)
```

Useful for

- rolling weekday revenue
- rolling weekend revenue
- rolling approved sales
- rolling premium customers

---

# Technique 6 — Signed Aggregation

Appears in

- Problem 5

Instead of

```
Sell Revenue

-

Buy Revenue
```

transform values first.

```
Buy

↓

Negative

Sell

↓

Positive

↓

SUM
```

Template

```sql
SUM(

CASE

WHEN operation='Buy'

THEN -amount

ELSE amount

END

)
```

Cleaner than multiple SUM expressions.

---

# Decision Tree

```
Need one output row?

│

├── Need columns?

│      │

│      ├── CASE + MAX

│      └── CASE + SUM

│

├── Need counts?

│      │

│      └── SUM(CASE...)

│

├── Need percentages?

│      │

│      └── AVG(CASE...)

│

├── Need financial totals?

│      │

│      └── Signed CASE

│

└── Need rolling metrics?

       │

       └── CASE + Window Function
```

---

# LLM-Proof Questions

These variations expose common reasoning mistakes made by both candidates and language models.

---

# Question 1 — Multiple Conditions Inside One Pivot

Table

|employee|month|department|salary|
|---------|-----|----------|------|

Return

|employee|IT_Jan|IT_Feb|HR_Jan|HR_Feb|

---

## Correct Pattern

```sql
MAX(

CASE

WHEN department='IT'

AND month='Jan'

THEN salary

END

)
```

Repeat for each output column.

---

## Why LLMs Fail

Many generated answers use

```sql
GROUP BY

employee,

department
```

This produces multiple rows instead of one.

Correct grouping

```sql
GROUP BY employee
```

---

## Spot the Error

If your output has multiple rows per employee,

the GROUP BY is almost certainly incorrect.

---

# Question 2 — NULL Handling During Pivot

Input

|Student|Subject|Marks|
|--------|-------|-----|
|A|Math|90|
|A|Science|NULL|
|B|Math|70|

Return

|Student|Math|Science|

---

Incorrect

```sql
SUM(

CASE

WHEN subject='Science'

THEN marks

END

)
```

If every Science mark is NULL,

SUM returns NULL.

Sometimes interviewers instead expect

```
0
```

Correct

```sql
COALESCE(

SUM(

CASE

WHEN subject='Science'

THEN marks

END

),

0

)
```

---

## Why LLMs Fail

Many solutions forget that

```
NULL

≠

0
```

Always read the expected output carefully.

---

# Question 3 — Overlapping CASE Conditions

Input

|Order|Amount|

Return

- Small (<100)
- Medium (100–500)
- Large (>500)

Incorrect

```sql
CASE

WHEN amount<=100

...

WHEN amount<=500

...
```

The boundary value

```
100
```

belongs to two logical categories depending on the intended specification.

Correct

```sql
CASE

WHEN amount<100

THEN 'Small'

WHEN amount<=500

THEN 'Medium'

ELSE 'Large'

END
```

---

## Why LLMs Fail

Boundary conditions are one of the most common sources of wrong answers.

Always test:

- minimum value
- maximum value
- exact boundary

---

# Visualization Library

---

# Visualization 1 — Classic Pivot

Before

|Employee|Month|Revenue|
|---------|-----|-------|
|1|Jan|100|
|1|Feb|200|
|1|Mar|300|

↓

After

|Employee|Jan|Feb|Mar|
|---------|---|---|---|
|1|100|200|300|

---

# Visualization 2 — Conditional Aggregation

```
Transactions

↓

CASE

Approved?

↓

YES → Amount

NO → 0

↓

SUM

↓

Approved Revenue
```

---

# Visualization 3 — Query Execution Pipeline

```
Input Table

↓

JOIN (optional)

↓

GROUP BY

↓

CASE

↓

Aggregate

↓

Final Output
```

---

# Visualization 4 — CASE Decision Tree

```
                CASE
                  │
      ┌───────────┴───────────┐
      │                       │
 Condition True         Condition False
      │                       │
 Return Value          Return NULL / 0
      │                       │
      └───────────┬───────────┘
                  │
            Aggregate
                  │
            Final Metric
```

---

# Interview Cheat Sheet

|Interview Requirement|Pattern|
|---------------------|-------|
|Rows → Columns|`MAX(CASE...)`|
|Conditional Count|`SUM(CASE ... THEN 1 ELSE 0 END)`|
|Conditional Sum|`SUM(CASE ... THEN amount ELSE 0 END)`|
|Conditional Average|`AVG(CASE ... THEN 1 ELSE 0 END)`|
|Net Profit|Signed `SUM(CASE...)`|
|Rolling Conditional Metric|`SUM(CASE...) OVER(...)`|
|First vs Later|Derived category + `CASE` + aggregate|
|Multiple Metrics|Multiple CASE expressions in one grouped query|

---

# Common Mistakes

|Mistake|Why It Fails|Fix|
|--------|------------|---|
|CASE outside aggregate|Returns multiple rows|Wrap with `SUM`, `MAX`, `COUNT`, or `AVG`|
|Wrong GROUP BY columns|Breaks pivot into multiple rows|Group only by final row identifier|
|Using WHERE instead of CASE|Filters out rows needed for other metrics|Keep all rows and classify with CASE|
|Forgetting ELSE 0|Can introduce NULL results|Use `ELSE 0` where appropriate|
|Using SUM instead of MAX for unique pivot values without understanding why|Works only if uniqueness holds|Use `MAX` for row-to-column pivots unless duplicates are intended|
|Ignoring duplicate source rows|Aggregates unexpected values|Verify uniqueness assumptions before pivoting|
|Incorrect date grouping|Produces daily instead of monthly summaries|Extract the correct time grain before grouping|
|Boundary errors in CASE|Misclassified rows|Test exact edge values|

---

# Pattern Summary

## 1. Row → Column Pivot

```sql
MAX(
CASE
WHEN category='A'
THEN value
END
)
```

---

## 2. Conditional Count

```sql
SUM(

CASE

WHEN condition

THEN 1

ELSE 0

END

)
```

---

## 3. Conditional Sum

```sql
SUM(

CASE

WHEN condition

THEN amount

ELSE 0

END

)
```

---

## 4. Conditional Percentage

```sql
AVG(

CASE

WHEN condition

THEN 1

ELSE 0

END

)*100
```

---

## 5. Signed Aggregation

```sql
SUM(

CASE

WHEN type='Debit'

THEN -amount

ELSE amount

END

)
```

---

## 6. Conditional Window Aggregate

```sql
SUM(

CASE

WHEN condition

THEN amount

ELSE 0

END

)

OVER(

PARTITION BY ...

ORDER BY ...

ROWS BETWEEN ...

)
```

---

# Final Interview Checklist

Before submitting a pivot query, verify:

- Group only by the desired output row identifier.
- Place `CASE` **inside** the aggregate function.
- Choose the correct aggregate (`MAX`, `SUM`, `COUNT`, or `AVG`).
- Check whether missing values should remain `NULL` or become `0`.
- Validate edge cases and boundary conditions.
- Ensure all required metrics are computed in a **single grouped query** whenever practical.
- Confirm the output grain (daily, monthly, yearly, per customer, per product, etc.) matches the problem statement.

---
