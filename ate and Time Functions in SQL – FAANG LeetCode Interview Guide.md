# Date and Time Functions in SQL – FAANG LeetCode Interview Guide

Master SQL Date & Time Functions through real LeetCode problems instead of memorizing syntax.

---

# Table of Contents

- [Problem 1 — Rising Temperature (Easy)](#problem-1--rising-temperature-easy)
- [Problem 2 — User Activity for the Past 30 Days I (Easy)](#problem-2--user-activity-for-the-past-30-days-i-easy)
- [Problem 3 — Game Play Analysis IV (Medium)](#problem-3--game-play-analysis-iv-medium)
- [Problem 4 — Restaurant Growth (Medium)](#problem-4--restaurant-growth-medium)
- [Problem 5 — Human Traffic of Stadium (Hard)](#problem-5--human-traffic-of-stadium-hard)
- [Problem 6 — Monthly Transactions I (Medium)](#problem-6--monthly-transactions-i-medium)
- [Techniques & Algorithms Used](#techniques--algorithms-used)
- [LLM-Proof Date SQL Questions](#llm-proof-date-sql-questions)
- [Interview Cheat Sheet](#interview-cheat-sheet)

---

# Problem 1 — Rising Temperature (Easy)

**LeetCode #197**

**Difficulty:** Easy

**Companies**

- Amazon
- Google
- Meta
- Microsoft
- Bloomberg
- Apple

---

## Problem Statement

Given the Weather table:

| id | recordDate | temperature |
|---|---|---|
|1|2015-01-01|10|
|2|2015-01-02|25|
|3|2015-01-03|20|

Return the IDs where:

- today's temperature is higher than yesterday's
- yesterday means exactly **one calendar day earlier**

Return:

|id|
|---|
|2|

---

## Example Visualization

Weather Table

|Date|Temp|
|---|---|
|Jan 1|10|
|Jan 2|25 ✅|
|Jan 3|20|

Timeline

```
Jan 1 ------ Jan 2 ------ Jan 3

 10°C         25°C         20°C
               ▲
               Higher than yesterday
```

---

# Key Insight

Most candidates compare IDs.

**That is incorrect.**

IDs are **not guaranteed** to be consecutive.

Always compare

```
recordDate
```

instead of

```
id
```

---

# Solution Approach

For every row

Find another row where

```
today.recordDate
=
yesterday.recordDate + 1 day
```

Then compare

```
today.temperature >
yesterday.temperature
```

---

## SQL Solution (MySQL)

```sql
SELECT
    w1.id
FROM Weather w1
JOIN Weather w2
    ON DATEDIFF(w1.recordDate, w2.recordDate) = 1
WHERE w1.temperature > w2.temperature;
```

---

## Explanation

### Step 1

Join the table with itself.

```
Today
↓

Weather w1

Yesterday
↓

Weather w2
```

---

### Step 2

Filter rows exactly one day apart.

```sql
DATEDIFF(w1.recordDate, w2.recordDate)=1
```

Possible comparisons

```
Jan 2
Jan 1
```

Allowed

```
Difference = 1
```

Not allowed

```
Jan 4
Jan 1

Difference = 3
```

---

### Step 3

Compare temperatures.

```sql
WHERE w1.temperature > w2.temperature
```

---

## Alternative Solution

Using DATE_ADD

```sql
SELECT
    w1.id
FROM Weather w1
JOIN Weather w2
ON w1.recordDate = DATE_ADD(w2.recordDate,INTERVAL 1 DAY)
WHERE w1.temperature>w2.temperature;
```

---

## Execution Flow

```
Weather

↓

Self Join

↓

Find Yesterday

↓

Compare Temperature

↓

Return ID
```

---

## Common Mistakes

### Mistake 1

Joining using IDs

```sql
w1.id=w2.id+1
```

Wrong.

Dates may skip.

Example

```
Jan 1

Jan 4
```

IDs can still be consecutive.

---

### Mistake 2

Using

```sql
>=
```

Need

```
Exactly yesterday
```

---

### Mistake 3

Comparing temperatures before joining

Always join first.

---

# Why Interviewers Like This Question

Tests whether candidates understand

- Self Join
- Date arithmetic
- DATEDIFF
- Calendar logic
- Difference between IDs and timestamps

---

# Complexity

Time

```
O(n log n)
```

with index on

```
recordDate
```

Space

```
O(1)
```

---

# Interview Takeaway

Whenever you hear

> yesterday

think

```
DATEDIFF()

or

DATE_ADD()
```

Never compare IDs.

---

# Problem 2 — User Activity for the Past 30 Days I (Easy)

**LeetCode #1141**

**Difficulty:** Easy

**Companies**

- Meta
- Amazon
- Microsoft
- TikTok
- Uber

---

## Problem Statement

Table

Activity

|user_id|session_id|activity_date|activity_type|
|---|---|---|---|

Find

For each day

Count distinct active users

during the **30-day period ending 2019-07-27**

Output

|day|active_users|

---

## Visualization

Reference Date

```
2019-07-27
```

Window

```
<----------------------------->
2019-06-28              2019-07-27
```

Everything outside

```
Ignored
```

---

Example

|Date|Users|
|---|---|
|2019-06-27|❌|
|2019-06-28|✔|
|2019-07-15|✔|
|2019-07-27|✔|

---

# Key Insight

The question is **not**

```
last 30 rows
```

It is

```
last 30 calendar days
```

Always filter using dates first.

Then aggregate.

---

# Solution Approach

Step 1

Keep only dates

```
2019-06-28

through

2019-07-27
```

Step 2

Group by day.

Step 3

Count distinct users.

---

## SQL Solution

```sql
SELECT
    activity_date AS day,
    COUNT(DISTINCT user_id) AS active_users
FROM Activity
WHERE activity_date BETWEEN DATE_SUB('2019-07-27', INTERVAL 29 DAY)
                        AND '2019-07-27'
GROUP BY activity_date;
```

---

## Why 29 Days?

Many candidates write

```
30 DAY
```

But

```
July 27

included

June 28

included
```

Count

```
30 calendar dates
```

Visualization

```
June 28

↓

June 29

↓

...

↓

July 27

Total = 30 days
```

---

## Explanation

### Filter

```sql
WHERE activity_date
BETWEEN
DATE_SUB(...)
AND ...
```

returns

```
Only last 30 calendar days
```

---

### Count Users

```sql
COUNT(DISTINCT user_id)
```

One user

Multiple sessions

Still counts once.

---

### Group

```sql
GROUP BY activity_date
```

Produces

```
One row

per day
```

---

## Execution Flow

```
Activity Table

↓

Date Filter

↓

Group By Date

↓

Count Distinct Users

↓

Output
```

---

## Example

Input

|User|Session|Date|
|---|---|---|
|1|10|July20|
|1|15|July20|
|2|11|July20|
|3|13|July21|

Output

|Day|Users|
|---|---|
|July20|2|
|July21|1|

User 1 has two sessions.

Still

```
COUNT DISTINCT

=

1 user
```

---

## Common Mistakes

### Mistake 1

Using

```sql
COUNT(user_id)
```

Wrong.

Need

```
COUNT DISTINCT
```

---

### Mistake 2

Using

```sql
CURRENT_DATE
```

The question specifies

```
2019-07-27
```

Always use the fixed reference date.

---

### Mistake 3

Subtracting 30 instead of 29.

Remember

```
Inclusive range
```

---

## Why Interviewers Ask This

Tests

- DATE_SUB
- BETWEEN
- Inclusive ranges
- COUNT DISTINCT
- Calendar reasoning

Many candidates lose points because of

```
29

vs

30

day window
```

---

# Complexity

Time

```
O(n)
```

Space

```
O(1)
```

(with indexing on activity_date)

---

# Interview Takeaway

Whenever you see

```
Last X days
```

Ask yourself

> Inclusive?

or

> Exclusive?

Off-by-one errors are among the most common mistakes in SQL date questions.

# Problem 3 — Game Play Analysis IV (Medium)

**LeetCode #550**

**Difficulty:** Medium

**Companies**

- Meta
- Amazon
- Microsoft
- Google
- ByteDance

---

## Problem Statement

Table: **Activity**

| player_id | device_id | event_date | games_played |
|-----------|-----------|------------|--------------|

A player logs in on different dates.

Find the fraction of players who logged in **again exactly one day after their first login**.

Return the answer rounded to **2 decimal places**.

---

## Example

### Input

|player_id|event_date|
|---|---|
|1|2016-03-01|
|1|2016-03-02|
|2|2017-06-25|
|3|2016-03-02|
|3|2018-07-03|

Player 1

```
First Login
↓

Mar 1

Next Day Login
↓

Mar 2

✔ Qualified
```

Player 2

```
Only one login

❌
```

Player 3

```
Mar 2

↓

Jul 3

Difference > 1 day

❌
```

Output

```
0.33
```

---

## Visualization

```
Player 1

Mar1 ───── Mar2
  ↑          ↑
First      +1 Day ✔

---------------------

Player 2

Jun25

Only Login

---------------------

Player 3

Mar2 ---------------- Jul3

Difference >1
```

---

# Key Insight

Do **not** compare every pair of dates.

Instead:

1. Find each player's **first login**
2. Check whether a login exists **exactly one day later**
3. Compute the ratio

---

# Solution Approach

Step 1

Find

```
MIN(event_date)
```

for every player.

---

Step 2

Join with Activity again.

---

Step 3

Look for

```
first_login + 1 day
```

---

Step 4

Divide

```
Qualified Players

/

Total Players
```

---

## SQL Solution

```sql
SELECT
    ROUND(
        COUNT(a.player_id) /
        (SELECT COUNT(DISTINCT player_id) FROM Activity),
        2
    ) AS fraction
FROM
(
    SELECT
        player_id,
        MIN(event_date) AS first_login
    FROM Activity
    GROUP BY player_id
) f
JOIN Activity a
ON a.player_id = f.player_id
AND a.event_date = DATE_ADD(f.first_login, INTERVAL 1 DAY);
```

---

## Explanation

### Step 1

Find the first login.

```sql
MIN(event_date)
```

Result

|player|first_login|
|---|---|
|1|Mar1|
|2|Jun25|
|3|Mar2|

---

### Step 2

Join again.

```
Player

↓

First Login

↓

Next Day Login?
```

---

### Step 3

Filter

```sql
DATE_ADD(first_login,INTERVAL 1 DAY)
```

Only exact next-day logins remain.

---

### Step 4

Compute fraction.

```
Qualified

/

All Players
```

---

## Query Execution Flow

```
Activity

↓

Group By Player

↓

MIN(event_date)

↓

Join Activity Again

↓

DATE_ADD(+1)

↓

COUNT

↓

ROUND()
```

---

## Common Mistakes

### Mistake 1

Using

```sql
DATEDIFF() <= 1
```

Wrong.

Need

```
Exactly one day.
```

---

### Mistake 2

Using

```sql
MAX(event_date)
```

Need

```
MIN()
```

because the problem asks about the **first login**.

---

### Mistake 3

Dividing by

```
COUNT(*)
```

Need

```
COUNT(DISTINCT player_id)
```

---

## Why Interviewers Like This

Tests

- MIN()
- DATE_ADD()
- Aggregate + Join
- Ratio calculations
- Calendar arithmetic

Very common Meta interview pattern.

---

# Complexity

Time

```
O(n log n)
```

Space

```
O(n)
```

---

# Interview Takeaway

Whenever you read

> first login

Immediately think

```
MIN(date)
```

Whenever you read

> next day

Think

```
DATE_ADD(first_login,INTERVAL 1 DAY)
```

---

# Problem 4 — Restaurant Growth (Medium)

**LeetCode #1321**

**Difficulty:** Medium

**Companies**

- Amazon
- Google
- Meta
- Apple
- Bloomberg

---

## Problem Statement

Table

Customer

|customer_id|visited_on|amount|
|---|---|---|

For every day beginning from the **7th recorded day**, calculate

- total income of the previous 7 days
- average daily income during those 7 days

Return

|visited_on|amount|average_amount|

---

## Visualization

Daily Revenue

|Day|Revenue|
|---|---:|
|Day1|100|
|Day2|120|
|Day3|140|
|Day4|90|
|Day5|160|
|Day6|130|
|Day7|110|

Window

```
Day1 Day2 Day3 Day4 Day5 Day6 Day7

<--------7 Days-------->

Output appears here
```

Move window

```
Day2 Day3 Day4 Day5 Day6 Day7 Day8

<--------7 Days-------->
```

---

# Key Insight

This is **not** about grouping.

This is a **rolling 7-day window**.

Window Functions make this much easier.

---

# Solution Approach

Step 1

Combine all customer purchases for each day.

---

Step 2

Create a rolling window.

---

Step 3

Calculate

```
SUM()

AVG()
```

over

```
current day

+

previous 6 days
```

---

Step 4

Ignore first six days.

---

## SQL Solution

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
    SUM(amount) OVER(
        ORDER BY visited_on
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS amount,
    ROUND(
        AVG(amount) OVER(
            ORDER BY visited_on
            ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
        ),
        2
    ) AS average_amount
FROM daily_sales
LIMIT 18446744073709551615 OFFSET 6;
```

---

## Explanation

### Step 1

Merge same-day purchases.

```
Many rows

↓

One row/day
```

---

### Step 2

Create

```
Rolling Window
```

```
Current Day

+

Previous 6 Days
```

---

### Step 3

Calculate

```sql
SUM()
```

and

```sql
AVG()
```

inside the window.

---

### Step 4

Skip first six rows.

Reason

There aren't enough previous days.

---

## Execution Flow

```
Customer

↓

Daily Revenue

↓

Window Function

↓

7-Day Sum

↓

7-Day Average

↓

Skip First 6 Days
```

---

## Rolling Window Diagram

```
Day1 Day2 Day3 Day4 Day5 Day6 Day7

Window

↓

Output

----------------------------

Day2 Day3 Day4 Day5 Day6 Day7 Day8

Window

↓

Output
```

---

## Common Mistakes

### Mistake 1

Using

```sql
GROUP BY
```

This problem requires

```
Window Functions
```

---

### Mistake 2

Using

```sql
RANGE
```

instead of

```sql
ROWS
```

ROWS counts physical rows.

Exactly what we need.

---

### Mistake 3

Not aggregating daily revenue first.

Multiple customers

≠

Multiple days.

---

## Why Interviewers Ask This

Tests

- Window Functions
- Rolling Aggregates
- Date ordering
- ROWS BETWEEN
- Time-series analysis

A frequent FAANG analytics interview pattern.

---

# Complexity

Time

```
O(n log n)
```

Space

```
O(n)
```

---

# Interview Takeaway

When you see

> previous 7 days

or

> rolling average

Immediately think

```sql
ROWS BETWEEN
6 PRECEDING
AND CURRENT ROW
```


# Problem 5 — Human Traffic of Stadium (Hard)

**LeetCode #601**

**Difficulty:** Hard

**Companies**

- Amazon
- Meta
- Google
- Microsoft
- Bloomberg

---

## Problem Statement

Table: **Stadium**

| id | visit_date | people |
|----|------------|--------|
| 1 | 2017-01-01 | 10 |
| 2 | 2017-01-02 | 109 |
| 3 | 2017-01-03 | 150 |
| 4 | 2017-01-04 | 99 |
| 5 | 2017-01-05 | 145 |
| 6 | 2017-01-06 | 145 |
| 7 | 2017-01-07 | 145 |

Return all rows that belong to **three or more consecutive days** where

```
people >= 100
```

Sort by

```
visit_date
```

---

## Example Visualization

```
Date        People

Jan1          10 ❌

Jan2         109 ✔
Jan3         150 ✔

Jan4          99 ❌

Jan5         145 ✔
Jan6         145 ✔
Jan7         145 ✔
```

Only

```
Jan5
Jan6
Jan7
```

qualify because they form a consecutive streak of at least three days.

---

# Key Insight

The difficult part is **identifying consecutive records**.

A common interview trick is to create a grouping key using

```
id - ROW_NUMBER()
```

Rows belonging to one consecutive sequence produce the same value.

---

# Solution Approach

### Step 1

Keep only rows with

```sql
people >= 100
```

---

### Step 2

Assign

```sql
ROW_NUMBER()
```

ordered by ID.

---

### Step 3

Compute

```sql
id - ROW_NUMBER()
```

Rows in one consecutive block share the same result.

Example

|id|row_number|id-rn|
|---|---:|---:|
|5|1|4|
|6|2|4|
|7|3|4|

---

### Step 4

Group by that value.

Keep only groups containing at least

```
3 rows
```

---

### Step 5

Return original rows.

---

## SQL Solution

```sql
WITH filtered AS
(
    SELECT
        *,
        id - ROW_NUMBER() OVER(ORDER BY id) AS grp
    FROM Stadium
    WHERE people >= 100
)

SELECT
    id,
    visit_date,
    people
FROM filtered
WHERE grp IN
(
    SELECT grp
    FROM filtered
    GROUP BY grp
    HAVING COUNT(*) >= 3
)
ORDER BY visit_date;
```

---

## Explanation

### Original Data

|id|people|
|---|---:|
|1|10|
|2|109|
|3|150|
|4|99|
|5|145|
|6|145|
|7|145|

---

### After Filtering

|id|people|
|---|---:|
|2|109|
|3|150|
|5|145|
|6|145|
|7|145|

---

### Add Row Number

|id|rn|
|---|---:|
|2|1|
|3|2|
|5|3|
|6|4|
|7|5|

---

### Create Group

```
id - rn
```

|id|rn|grp|
|---|---:|---:|
|2|1|1|
|3|2|1|
|5|3|2|
|6|4|2|
|7|5|2|

Group

```
2
```

contains

```
3 rows
```

Return all of them.

---

## Query Execution Flow

```
Stadium

↓

people >=100

↓

ROW_NUMBER()

↓

id-rn

↓

GROUP BY

↓

HAVING COUNT>=3

↓

Return Rows
```

---

## Common Mistakes

### Mistake 1

Checking only three adjacent rows.

The streak may contain

```
4

5

6

...

days
```

---

### Mistake 2

Using

```sql
LAG()
```

only.

LAG identifies neighbors but cannot easily return an entire streak.

---

### Mistake 3

Grouping by date.

The problem asks for

```
consecutive records
```

not monthly or daily aggregation.

---

## Why Interviewers Ask This

This problem combines several advanced ideas:

- Window Functions
- Consecutive sequence detection
- CTEs
- Grouping tricks
- Filtering after window calculations

It is a classic SQL interview question because it tests pattern recognition rather than memorized syntax.

---

# Complexity

Time

```
O(n log n)
```

Space

```
O(n)
```

---

# Interview Takeaway

Whenever you see

> consecutive rows

or

> consecutive days

consider the pattern

```sql
id - ROW_NUMBER()
```

It appears frequently in hard SQL interview questions.

---

# Problem 6 — Monthly Transactions I (Medium)

**LeetCode #1193**

**Difficulty:** Medium

**Companies**

- Amazon
- Google
- Uber
- PayPal
- Meta

---

## Problem Statement

Table: **Transactions**

| id | country | state | amount | trans_date |
|----|---------|-------|--------|------------|

For each **country** and **month**, report

- total transactions
- approved transactions
- total amount
- approved amount

Output columns

| month | country | trans_count | approved_count | trans_total_amount | approved_total_amount |

---

## Example Visualization

Input

|Date|Country|State|Amount|
|---|---|---|---:|
|2019-01-02|US|approved|1000|
|2019-01-12|US|declined|200|
|2019-02-08|US|approved|300|

Grouping

```
January

↓

US

↓

2 Transactions

↓

1 Approved
```

---

# Key Insight

The challenge is extracting the month while performing multiple conditional aggregations.

Instead of writing multiple queries, use

- date formatting
- conditional aggregation

in one pass.

---

# Solution Approach

### Step 1

Extract

```
YYYY-MM
```

from the transaction date.

---

### Step 2

Group by

```
Month

+

Country
```

---

### Step 3

Use

```sql
SUM(CASE...)
```

to calculate approved counts and approved amounts.

---

## SQL Solution

```sql
SELECT
    DATE_FORMAT(trans_date,'%Y-%m') AS month,
    country,
    COUNT(*) AS trans_count,
    SUM(state='approved') AS approved_count,
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

## Explanation

### Step 1

Convert

```
2019-02-15
```

into

```
2019-02
```

using

```sql
DATE_FORMAT()
```

---

### Step 2

Group by

```
Month

+

Country
```

---

### Step 3

Count all rows.

```sql
COUNT(*)
```

---

### Step 4

Count approved rows.

```sql
SUM(state='approved')
```

In MySQL,

```
TRUE = 1

FALSE = 0
```

---

### Step 5

Compute approved amount.

```sql
SUM(
CASE
WHEN approved
THEN amount
END
)
```

---

## Query Execution Flow

```
Transactions

↓

DATE_FORMAT()

↓

Month

↓

GROUP BY

↓

Conditional Aggregation

↓

Final Report
```

---

## Example Output

|Month|Country|Transactions|Approved|Amount|
|---|---|---:|---:|---:|
|2019-01|US|2|1|1200|
|2019-02|US|1|1|300|

---

## Common Mistakes

### Mistake 1

Grouping directly by

```sql
trans_date
```

That creates one group per day instead of per month.

---

### Mistake 2

Using

```sql
MONTH(trans_date)
```

This merges

```
Jan 2018

and

Jan 2019
```

Always preserve the year.

---

### Mistake 3

Writing separate queries for approved transactions.

Conditional aggregation is cleaner and more efficient.

---

## Why Interviewers Ask This

Tests several frequently used reporting skills:

- Date formatting
- Time-based grouping
- Conditional aggregation
- Business reporting queries

These patterns are common in analytics and data engineering interviews.

---

# Complexity

Time

```
O(n)
```

Space

```
O(1)
```

---

# Interview Takeaway

Whenever you need monthly reports, think:

```sql
DATE_FORMAT(date,'%Y-%m')
```

followed by

```sql
GROUP BY
```

and

```sql
SUM(CASE WHEN ...)
```


# Techniques & Algorithms Used

This section extracts only the techniques that appeared in the six selected LeetCode problems. Think of it as a revision sheet rather than a SQL reference.

---

# Table of Techniques

| Technique | Problems | Purpose |
|-----------|----------|---------|
| `DATEDIFF()` | #197 | Compare dates exactly one day apart |
| `DATE_ADD()` | #197, #550 | Move forward by one day |
| `DATE_SUB()` | #1141 | Create rolling date filters |
| `DATE_FORMAT()` | #1193 | Monthly reporting |
| `MIN(date)` | #550 | Find first event |
| `COUNT(DISTINCT)` | #1141 | Unique user counting |
| Window Functions | #1321, #601 | Rolling windows and sequence detection |
| `ROW_NUMBER()` | #601 | Consecutive row detection |
| Conditional Aggregation | #1193 | Multiple metrics in one query |
| Self Join | #197 | Compare today's row with yesterday |

---

# Pattern 1 — Date Arithmetic

Appears in

- Rising Temperature
- Game Play Analysis IV
- User Activity for the Past 30 Days I

Typical interview wording

> yesterday

> next day

> previous X days

> within N days

Common functions

```sql
DATE_ADD(date, INTERVAL 1 DAY)

DATE_SUB(date, INTERVAL 30 DAY)

DATEDIFF(date1,date2)
```

Visualization

```
Today

↓

2024-05-10

↓

DATE_SUB(...,7 DAY)

↓

2024-05-03
```

---

## When to Use Each

| Situation | Function |
|-----------|----------|
| Yesterday | `DATEDIFF()` or `DATE_ADD()` |
| Last N days | `DATE_SUB()` |
| Exact calendar difference | `DATEDIFF()` |
| Future date | `DATE_ADD()` |

---

# Pattern 2 — Fixed Date Window

Appears in

- User Activity for the Past 30 Days I

Typical filter

```sql
WHERE activity_date
BETWEEN start_date
AND end_date
```

Visualization

```
June28 ---------------- July27

Included

June27

Ignored

July28

Ignored
```

Interview tip

Always verify whether the endpoints are

- inclusive
- exclusive

Many SQL bugs come from off-by-one errors.

---

# Pattern 3 — First Event Detection

Appears in

- Game Play Analysis IV

Typical query

```sql
SELECT
player_id,
MIN(event_date)
FROM Activity
GROUP BY player_id;
```

Visualization

```
Player

↓

Many Logins

↓

MIN()

↓

First Login
```

Whenever the question says

- first order
- first purchase
- first visit
- first login

your first instinct should be

```sql
MIN(date)
```

---

# Pattern 4 — Rolling Window

Appears in

- Restaurant Growth

Typical syntax

```sql
ROWS BETWEEN
6 PRECEDING
AND CURRENT ROW
```

Visualization

```
Day1 Day2 Day3 Day4 Day5 Day6 Day7

<------Window------>

↓

Output
```

Move one row

```
Day2 Day3 Day4 Day5 Day6 Day7 Day8

<------Window------>
```

Rolling windows appear in

- dashboards
- finance
- stock prices
- analytics
- user engagement

---

# Pattern 5 — Consecutive Sequence Detection

Appears in

- Human Traffic of Stadium

Classic interview trick

```sql
id - ROW_NUMBER()
```

Visualization

|id|ROW_NUMBER|Difference|
|---|---:|---:|
|10|1|9|
|11|2|9|
|12|3|9|

Same difference

↓

Same consecutive block

---

Why it works

Suppose IDs increase by exactly one.

```
10

11

12
```

ROW_NUMBER also increases by one.

```
1

2

3
```

Subtracting them produces a constant value.

Break the sequence

```
10

11

15
```

Now

```
15 - 3

≠

10 - 1
```

A new group begins.

---

# Pattern 6 — Monthly Reporting

Appears in

- Monthly Transactions I

Typical query

```sql
DATE_FORMAT(trans_date,'%Y-%m')
```

Visualization

```
2023-01-12

↓

2023-01

----------------

2023-01-27

↓

2023-01

Same Group
```

Avoid

```sql
MONTH(date)
```

because

```
January 2023

January 2024
```

would be merged incorrectly.

---

# Pattern 7 — Conditional Aggregation

Appears in

- Monthly Transactions I

Instead of writing several queries

Use

```sql
SUM(
CASE
WHEN ...
THEN ...
END
)
```

Example

```sql
SUM(
CASE
WHEN state='approved'
THEN amount
ELSE 0
END
)
```

Visualization

```
Approved?

↓

Yes

↓

Add Amount

No

↓

Add 0
```

---

# Pattern 8 — Self Join

Appears in

- Rising Temperature

Visualization

```
Weather

↓

Today

JOIN

Yesterday
```

Typical syntax

```sql
FROM Weather w1

JOIN Weather w2
```

Useful for

- previous day
- next day
- previous salary
- adjacent records

---

# Choosing the Right Date Function

| Interview Phrase | Function |
|------------------|----------|
| Yesterday | `DATEDIFF()` |
| Tomorrow | `DATE_ADD()` |
| Last 30 Days | `DATE_SUB()` |
| First Login | `MIN(date)` |
| Monthly Report | `DATE_FORMAT()` |
| Consecutive Days | `ROW_NUMBER()` |
| Rolling Average | Window Function |
| Daily Active Users | `COUNT(DISTINCT)` |

---

# Decision Tree

```
Need yesterday?

↓

DATEDIFF()

-----------------------

Need future date?

↓

DATE_ADD()

-----------------------

Need previous X days?

↓

DATE_SUB()

-----------------------

Need first occurrence?

↓

MIN()

-----------------------

Need month grouping?

↓

DATE_FORMAT()

-----------------------

Need rolling calculations?

↓

Window Function

-----------------------

Need consecutive rows?

↓

ROW_NUMBER()

-----------------------

Need multiple metrics?

↓

CASE + SUM()
```

---

# FAANG Interview Checklist

Before submitting a date-related SQL solution, verify:

- Is the comparison based on **dates**, not IDs?
- Is the date range inclusive or exclusive?
- Did you preserve the **year** when grouping by month?
- Are duplicate users counted only once where required?
- Should missing dates be handled explicitly?
- Is a rolling window required instead of `GROUP BY`?
- Does the question require **calendar days** or simply consecutive rows?
- Could a window function simplify the query?
- Are you using indexes on date columns where appropriate?

---

# Key Takeaways

| Problem | Main Trick |
|---------|------------|
| Rising Temperature | Self Join + `DATEDIFF()` |
| User Activity Past 30 Days | `DATE_SUB()` + `COUNT(DISTINCT)` |
| Game Play Analysis IV | `MIN(date)` + `DATE_ADD()` |
| Restaurant Growth | Rolling Window Functions |
| Human Traffic of Stadium | `ROW_NUMBER()` grouping trick |
| Monthly Transactions I | `DATE_FORMAT()` + Conditional Aggregation |
