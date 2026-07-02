# CASE WHEN Statements in SQL — FAANG Interview Preparation (LeetCode)

> A practical interview guide focused entirely on solving LeetCode SQL questions using **CASE WHEN**. Every concept is learned through real interview problems rather than isolated theory.

---

# Table of Contents

- [Question 1 — Not Boring Movies (Easy)](#question-1--not-boring-movies-easy)
- [Question 2 — Immediate Food Delivery II (Medium)](#question-2--immediate-food-delivery-ii-medium)
- [Question 3 — Confirmation Rate (Medium)](#question-3--confirmation-rate-medium)
- [Question 4 — Human Traffic of Stadium (Hard)](#question-4--human-traffic-of-stadium-hard)
- [Question 5 — Count Salary Categories (Medium)](#question-5--count-salary-categories-medium)
- [Question 6 — Investments in 2016 (Hard)](#question-6--investments-in-2016-hard)
- [Key Techniques & Algorithms](#key-techniques--algorithms)
- [LLM-Proof Interview Questions](#llm-proof-interview-questions)
- [Company-wise Question Index](#company-wise-question-index)
- [Common Interview Mistakes](#common-interview-mistakes)

---

# Question 1 — Not Boring Movies (Easy)

**LeetCode:** 620

**Difficulty:** Easy

**Primary Topic**

- CASE WHEN
- Filtering
- ORDER BY

**Frequently Asked By**

| Company | Frequency |
|----------|-----------|
| Amazon | ⭐⭐⭐⭐ |
| Google | ⭐⭐⭐ |
| Meta | ⭐⭐⭐ |
| Apple | ⭐⭐ |
| Microsoft | ⭐⭐⭐ |

---

## Problem Statement

Table:

```text
Cinema
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| movie       | varchar |
| description | varchar |
| rating      | float   |
+-------------+---------+
```

Write a query to:

- Ignore movies whose description is **"boring"**
- Keep only movies with **odd IDs**
- Sort by rating in descending order

---

## Constraints

- id is unique.
- rating can contain decimal values.
- description is lowercase.

---

## Example

### Input

| id | movie | description | rating |
|---:|--------|-------------|------:|
|1|War|great|8.9|
|2|Science|boring|6.0|
|3|Irish|fantasy|6.5|
|4|Ice Song|boring|8.6|
|5|House Card|interesting|9.1|

### Output

| id | movie | description | rating |
|---:|--------|-------------|------:|
|5|House Card|interesting|9.1|
|1|War|great|8.9|
|3|Irish|fantasy|6.5|

---

# Optimal Solution

```sql
SELECT *
FROM Cinema
WHERE description <> 'boring'
  AND id % 2 = 1
ORDER BY rating DESC;
```

---

# Why CASE WHEN Matters Here

Although the official solution doesn't require CASE directly, interviewers often extend the question.

Example:

Instead of removing boring movies, classify them.

```sql
SELECT
    movie,
    rating,
    CASE
        WHEN description = 'boring'
            THEN 'Ignore'
        ELSE 'Recommended'
    END AS movie_status
FROM Cinema;
```

Expected output

| movie | rating | movie_status |
|-------|-------:|--------------|
|War|8.9|Recommended|
|Science|6.0|Ignore|

This tests whether candidates understand that **CASE WHEN creates conditional values without filtering rows.**

---

# Interview Insight

Many candidates mistakenly write

```sql
CASE description = 'boring'
```

Correct syntax:

```sql
CASE
    WHEN description='boring'
    THEN ...
    ELSE ...
END
```

---

# Alternative Solution

```sql
SELECT
    *,
    CASE
        WHEN id % 2 = 1
             AND description <> 'boring'
        THEN 1
        ELSE 0
    END AS keep_movie
FROM Cinema;
```

Then filter outside using a subquery.

```sql
SELECT *
FROM
(
    SELECT *,
           CASE
                WHEN id % 2 = 1
                 AND description <> 'boring'
                THEN 1
                ELSE 0
           END AS keep_movie
    FROM Cinema
)t
WHERE keep_movie = 1
ORDER BY rating DESC;
```

---

# Why Interviewers Like This Question

It checks whether you know the difference between

- filtering rows (**WHERE**)
- assigning labels (**CASE WHEN**)

This distinction appears repeatedly in harder SQL interview questions.

---

# Complexity

| Operation | Complexity |
|-----------|------------|
| Scan | O(n) |
| Sort | O(n log n) |

---

---

# Question 2 — Immediate Food Delivery II (Medium)

**LeetCode:** 1174

**Difficulty:** Medium

**Primary Topics**

- CASE WHEN
- Aggregate Functions
- AVG
- Conditional Counting

**Frequently Asked By**

| Company | Frequency |
|----------|-----------|
| Meta | ⭐⭐⭐⭐ |
| Amazon | ⭐⭐⭐⭐ |
| Google | ⭐⭐⭐ |
| Apple | ⭐⭐ |
| Uber | ⭐⭐⭐ |

---

## Problem Statement

Table:

```text
Delivery

+---------------------+------+
| customer_id         | int  |
| order_date          | date |
| customer_pref_date  | date |
+---------------------+------+
```

A customer's **first order** is considered

- Immediate
- Scheduled

Immediate means

```text
order_date = customer_pref_date
```

Return the percentage of customers whose **first order** was immediate.

---

## Constraints

- One customer can have multiple orders.
- Only the earliest order per customer matters.
- Output rounded to 2 decimal places.

---

## Example

Input

|customer|order|preferred|
|--------|------|----------|
|1|2020-01-01|2020-01-01|
|2|2020-01-02|2020-01-05|
|1|2020-02-01|2020-02-01|
|3|2020-03-01|2020-03-01|

First orders

|Customer|Immediate?|
|---------|----------|
|1|Yes|
|2|No|
|3|Yes|

Immediate Percentage

```text
66.67%
```

---

# Optimal Solution

```sql
SELECT
ROUND(
AVG(
CASE
    WHEN order_date = customer_pref_date
    THEN 1
    ELSE 0
END
) * 100,
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

# Solution Breakdown

### Step 1

Find every customer's first order.

```sql
SELECT
customer_id,
MIN(order_date)
FROM Delivery
GROUP BY customer_id;
```

Result

|Customer|First Order|
|---------|-----------|
|1|2020-01-01|
|2|2020-01-02|
|3|2020-03-01|

---

### Step 2

Keep only these rows.

---

### Step 3

Convert condition into numbers.

```sql
CASE
WHEN order_date = customer_pref_date
THEN 1
ELSE 0
END
```

Produces

|Customer|Value|
|---------|----:|
|1|1|
|2|0|
|3|1|

---

### Step 4

Average

```
(1+0+1)/3
```

```
0.6667
```

Multiply by 100.

```
66.67%
```

---

# Why CASE WHEN Is Perfect Here

Interviewers love this pattern:

```sql
AVG(
CASE WHEN condition
THEN 1
ELSE 0
END
)
```

Because

TRUE becomes

```
1
```

FALSE becomes

```
0
```

Average automatically becomes

```
successful_rows / total_rows
```

This is one of the most common SQL interview tricks.

---

# Alternative Solution Using SUM

```sql
SELECT
ROUND(
SUM(
CASE
WHEN order_date = customer_pref_date
THEN 1
ELSE 0
END
)
*100.0
/
COUNT(*),
2
)
FROM Delivery
WHERE (customer_id, order_date)
IN
(
SELECT
customer_id,
MIN(order_date)
FROM Delivery
GROUP BY customer_id
);
```

---

# Which Is Better?

| Method | Preferred? |
|---------|------------|
|AVG(CASE...)|✅ Best|
|SUM()/COUNT()|Also Good|

Most interviewers slightly prefer

```sql
AVG(CASE...)
```

because it is cleaner and scales well to many conditional-percentage problems.

---

# Common Mistakes

### Wrong

```sql
AVG(order_date = customer_pref_date)
```

Not portable across SQL dialects.

---

### Wrong

Using every order instead of first order.

---

### Wrong

```sql
COUNT(
CASE ...
)
```

Remember:

```sql
COUNT()
```

counts non-null values.

If using COUNT with CASE, explicitly return NULL for rows that should not be counted, or prefer SUM/AVG patterns for clarity.

---

# Complexity

| Operation | Complexity |
|-----------|------------|
| Grouping | O(n) |
| Join/Subquery | O(n) |
| Total | O(n) |

---

**Next:** Part 2 covers:

- **Question 3 — Confirmation Rate (Medium)**
- **Question 4 — Human Traffic of Stadium (Hard)**

These introduce more advanced `CASE WHEN` usage with conditional aggregation, joins, and window functions.


---

# Question 3 — Confirmation Rate (Medium)

**LeetCode:** 1934

**Difficulty:** Medium

**Primary Topics**

- CASE WHEN
- LEFT JOIN
- Conditional Aggregation
- AVG()
- NULL Handling

**Frequently Asked By**

| Company | Frequency |
|----------|-----------|
| Meta | ⭐⭐⭐⭐ |
| Amazon | ⭐⭐⭐⭐ |
| Google | ⭐⭐⭐ |
| Apple | ⭐⭐ |
| Microsoft | ⭐⭐⭐ |

---

## Problem Statement

Tables

### Signups

```text
+-------------+------+
| user_id     | int  |
| time_stamp  | date |
+-------------+------+
```

### Confirmations

```text
+---------------+---------+
| user_id       | int     |
| time_stamp    | datetime|
| action        | varchar |
+---------------+---------+
```

`action` can be

- confirmed
- timeout

For every user calculate

```
confirmation rate =
confirmed actions
-----------------
total confirmation requests
```

If a user has **no confirmation requests**, return

```
0
```

Round to **2 decimal places**.

---

## Constraints

- Every signup user must appear in the output.
- Some users never receive a confirmation request.
- Multiple confirmation attempts may exist.

---

## Example

### Signups

|user_id|
|------:|
|3|
|7|
|2|
|6|

### Confirmations

|user_id|action|
|------:|--------|
|3|confirmed|
|3|timeout|
|7|confirmed|
|7|confirmed|
|7|timeout|

---

Output

|user_id|confirmation_rate|
|------:|---------------:|
|2|0.00|
|3|0.50|
|6|0.00|
|7|0.67|

---

# Optimal Solution

```sql
SELECT
    s.user_id,
    ROUND(
        IFNULL(
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

# Step-by-Step Explanation

### Step 1

Keep every signup.

```sql
FROM Signups
LEFT JOIN Confirmations
```

A normal INNER JOIN would lose users without confirmations.

---

### Step 2

Convert confirmation status into numbers.

```sql
CASE
WHEN action='confirmed'
THEN 1
ELSE 0
END
```

Example

|Action|Value|
|------|----:|
|confirmed|1|
|timeout|0|

---

### Step 3

Take average.

Example

|User|Values|
|----|------|
|3|1,0|
|7|1,1,0|

Average

```
User 3

(1+0)/2 = 0.50
```

```
User 7

(1+1+0)/3 = 0.67
```

---

### Step 4

Users without confirmations

For user

```
2
```

AVG becomes

```
NULL
```

Convert it into

```
0
```

using

```sql
IFNULL()
```

---

# Why CASE WHEN Is Perfect Here

Instead of writing

```
confirmed_count
---------------
total_count
```

we simply transform

```
confirmed → 1

timeout → 0
```

Then

```sql
AVG(...)
```

does all the work.

This is one of the most common FAANG SQL interview patterns.

---

# Alternative Solution

Using SUM.

```sql
SELECT
    s.user_id,
    ROUND(
        IFNULL(
            SUM(
                CASE
                    WHEN c.action='confirmed'
                    THEN 1
                    ELSE 0
                END
            )
            /
            COUNT(c.action),
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

# Interview Tip

A common mistake is

```sql
COUNT(*)
```

After a LEFT JOIN,

```
COUNT(*)
```

counts even the NULL row generated by the join.

Instead use

```sql
COUNT(c.action)
```

because NULL values are ignored.

---

# Common Mistakes

### Wrong

```sql
INNER JOIN
```

Removes users without confirmations.

---

### Wrong

```sql
COUNT(*)
```

after LEFT JOIN.

---

### Wrong

Forgetting

```sql
IFNULL(...)
```

leading to NULL instead of 0.

---

# Complexity

| Operation | Complexity |
|-----------|------------|
| LEFT JOIN | O(n) |
| GROUP BY | O(n) |
| Total | O(n) |

---

---

# Question 4 — Human Traffic of Stadium (Hard)

**LeetCode:** 601

**Difficulty:** Hard

**Primary Topics**

- CASE WHEN
- Window Functions
- Consecutive Rows
- CTE
- Ranking

**Frequently Asked By**

| Company | Frequency |
|----------|-----------|
| Google | ⭐⭐⭐⭐⭐ |
| Amazon | ⭐⭐⭐⭐ |
| Meta | ⭐⭐⭐ |
| Apple | ⭐⭐⭐ |
| Bloomberg | ⭐⭐⭐⭐ |

---

## Problem Statement

Table

```text
Stadium

+-----------+------+
| id        | int  |
| visit_date| date |
| people    | int  |
+-----------+------+
```

Return records belonging to **three or more consecutive ids**
where

```
people >=100
```

---

## Constraints

- id values are increasing.
- Need every row in qualifying sequences.
- Sequence length may exceed 3.

---

## Example

Input

|id|people|
|--:|-----:|
|1|10|
|2|109|
|3|150|
|4|99|
|5|145|
|6|145|
|7|199|

Output

|id|
|--:|
|5|
|6|
|7|

---

# Optimal Solution

```sql
WITH filtered AS
(
    SELECT *
    FROM Stadium
    WHERE people >= 100
),
numbered AS
(
    SELECT *,
           id - ROW_NUMBER() OVER(ORDER BY id) AS grp
    FROM filtered
)
SELECT
    id,
    visit_date,
    people
FROM numbered
WHERE grp IN
(
    SELECT grp
    FROM numbered
    GROUP BY grp
    HAVING COUNT(*) >= 3
)
ORDER BY visit_date;
```

---

# Where CASE WHEN Appears in Interviews

Interviewers often extend the problem.

Instead of filtering immediately,

classify rows first.

```sql
SELECT
id,
people,
CASE
WHEN people>=100
THEN 'High Traffic'
ELSE 'Low Traffic'
END AS traffic_type
FROM Stadium;
```

Now later stages can

- aggregate
- filter
- rank

using this computed label.

---

# Why CASE WHEN Is Useful

Imagine the interviewer changes the requirement.

Instead of

```
>=100
```

they ask

```
VIP
Busy
Normal
Empty
```

CASE handles this naturally.

```sql
CASE
WHEN people>=300 THEN 'VIP'
WHEN people>=200 THEN 'Busy'
WHEN people>=100 THEN 'Normal'
ELSE 'Empty'
END
```

Nested business rules like this appear frequently in production SQL.

---

# Alternative Solution

Using LAG/LEAD.

```sql
LAG()

LEAD()
```

can detect neighboring rows.

However,

for sequences longer than

```
3
```

the

```
id - ROW_NUMBER()
```

grouping trick is cleaner and more scalable.

---

# Interview Insight

Many candidates assume

```
3 consecutive rows
```

means

```
ROW_NUMBER()
```

alone is enough.

It is not.

The key observation is

```
id - row_number
```

remains constant inside consecutive sequences.

Example

|id|row_number|Difference|
|--:|---------:|---------:|
|5|1|4|
|6|2|4|
|7|3|4|

The shared difference identifies one consecutive group.

---

# Common Mistakes

### Wrong

Checking only three neighboring rows.

Fails for sequences of length

```
4
```

or

```
8
```

---

### Wrong

Grouping by

```sql
people
```

instead of consecutive ids.

---

### Wrong

Ignoring missing ids.

The consecutive requirement is based on the **id values**, not the number of returned rows.

---

# Complexity

| Operation | Complexity |
|-----------|------------|
| Window Function | O(n log n) |
| GROUP BY | O(n) |
| Total | O(n log n) |

---

**Next:** Part 3 covers:

- **Question 5 — Count Salary Categories (Medium)**
- **Question 6 — Investments in 2016 (Hard)**
```


---

# Question 5 — Count Salary Categories (Medium)

**LeetCode:** 1907

**Difficulty:** Medium

**Primary Topics**

- CASE WHEN
- Conditional Classification
- UNION ALL
- Aggregate Functions

**Frequently Asked By**

| Company | Frequency |
|----------|-----------|
| Amazon | ⭐⭐⭐⭐ |
| Meta | ⭐⭐⭐ |
| Google | ⭐⭐⭐ |
| Apple | ⭐⭐ |
| Uber | ⭐⭐⭐ |

---

## Problem Statement

Table

```text
Accounts

+--------------+------+
| account_id   | int  |
| income       | int  |
+--------------+------+
```

Classify every account into one of the following categories:

| Income | Category |
|---------|----------|
| `< 20000` | Low Salary |
| `20000 - 50000` | Average Salary |
| `> 50000` | High Salary |

Return the number of accounts in each category.

Even if a category has **zero** accounts, it must still appear in the output.

---

## Constraints

- `account_id` is unique.
- Income is non-negative.
- Output must contain exactly **3 rows**.

---

## Example

### Input

|account_id|income|
|----------|-----:|
|1|15000|
|2|45000|
|3|65000|
|4|52000|

### Output

|category|accounts_count|
|---------|-------------:|
|Low Salary|1|
|Average Salary|1|
|High Salary|2|

---

# Optimal Solution

```sql
SELECT
    'Low Salary' AS category,
    COUNT(*) AS accounts_count
FROM Accounts
WHERE income < 20000

UNION ALL

SELECT
    'Average Salary',
    COUNT(*)
FROM Accounts
WHERE income BETWEEN 20000 AND 50000

UNION ALL

SELECT
    'High Salary',
    COUNT(*)
FROM Accounts
WHERE income > 50000;
```

---

# CASE WHEN Version

Many interviewers specifically ask for a CASE-based solution.

```sql
SELECT
    category,
    COUNT(*) AS accounts_count
FROM
(
    SELECT
        CASE
            WHEN income < 20000 THEN 'Low Salary'
            WHEN income <= 50000 THEN 'Average Salary'
            ELSE 'High Salary'
        END AS category
    FROM Accounts
) t
GROUP BY category;
```

---

## Why Isn't This Enough?

Suppose there are no high-income employees.

Example

|Income|
|-----:|
|10000|
|25000|

Output becomes

|Category|
|---------|
|Low Salary|
|Average Salary|

The **High Salary** row disappears.

The problem explicitly requires **all three categories**, even when the count is zero.

Therefore, the UNION ALL solution is generally preferred.

---

# Why CASE WHEN Is the Right Technique

This problem demonstrates one of the most common interview patterns:

> Convert numeric ranges into business categories.

```sql
CASE
WHEN income < 20000 THEN ...
WHEN income <= 50000 THEN ...
ELSE ...
END
```

This same pattern appears in:

- Credit score grading
- Product segmentation
- Customer tiers
- Risk classification
- Revenue buckets

---

# Alternative Solution (Conditional Aggregation)

```sql
SELECT
SUM(CASE WHEN income < 20000 THEN 1 ELSE 0 END) AS low_salary,
SUM(CASE WHEN income BETWEEN 20000 AND 50000 THEN 1 ELSE 0 END) AS average_salary,
SUM(CASE WHEN income > 50000 THEN 1 ELSE 0 END) AS high_salary
FROM Accounts;
```

Output

|low_salary|average_salary|high_salary|
|----------:|-------------:|-----------:|
|1|1|2|

Although correct, this **does not** match the required output format.

---

# Interview Insight

Candidates often write

```sql
CASE
WHEN income < 20000 ...
WHEN income BETWEEN 20000 AND 50000 ...
WHEN income > 50000 ...
END
```

This works because CASE evaluates conditions from top to bottom and stops at the first match.

You can simplify the second condition:

```sql
CASE
WHEN income < 20000 THEN ...
WHEN income <= 50000 THEN ...
ELSE ...
END
```

This version is slightly cleaner.

---

# Common Mistakes

### Wrong

```sql
GROUP BY income
```

The grouping should be by **category**, not income.

---

### Wrong

Using `UNION` instead of `UNION ALL`.

Since the three result rows are distinct, both produce the same output here, but `UNION ALL` avoids unnecessary duplicate elimination.

---

### Wrong

Forgetting categories with zero rows.

This is the most common interview mistake.

---

# Complexity

| Operation | Complexity |
|-----------|------------|
| Table Scan | O(n) |
| Aggregation | O(n) |
| Total | O(n) |

---

---

# Question 6 — Investments in 2016 (Hard)

**LeetCode:** 585

**Difficulty:** Hard

**Primary Topics**

- CASE WHEN
- GROUP BY
- HAVING
- Self Filtering
- Aggregate Conditions

**Frequently Asked By**

| Company | Frequency |
|----------|-----------|
| Google | ⭐⭐⭐⭐ |
| Amazon | ⭐⭐⭐⭐ |
| Meta | ⭐⭐⭐ |
| Bloomberg | ⭐⭐⭐⭐ |
| Oracle | ⭐⭐⭐ |

---

## Problem Statement

Table

```text
Insurance

+----------+---------+
| pid      | int     |
| tiv_2015 | decimal |
| tiv_2016 | decimal |
| lat      | decimal |
| lon      | decimal |
+----------+---------+
```

Return the sum of `tiv_2016` for policyholders satisfying **both** conditions:

1. Their `tiv_2015` value appears more than once.
2. Their `(lat, lon)` location is unique.

Round the answer to **2 decimal places**.

---

## Constraints

- `(lat, lon)` represents a geographic location.
- Multiple policies may share the same `tiv_2015`.
- Duplicate locations must be excluded.

---

## Example

### Input

|pid|tiv_2015|tiv_2016|lat|lon|
|--:|--------:|--------:|--:|--:|
|1|10|5|1|1|
|2|20|7|2|2|
|3|10|8|3|3|
|4|20|9|2|2|

Duplicate `tiv_2015`

```
10
20
```

Duplicate location

```
(2,2)
```

Only policy

```
1
```

and

```
3
```

qualify.

Output

```
13.00
```

---

# Optimal Solution

```sql
SELECT
ROUND(SUM(tiv_2016),2) AS tiv_2016
FROM Insurance
WHERE tiv_2015 IN
(
    SELECT tiv_2015
    FROM Insurance
    GROUP BY tiv_2015
    HAVING COUNT(*) > 1
)
AND (lat, lon) IN
(
    SELECT
        lat,
        lon
    FROM Insurance
    GROUP BY lat, lon
    HAVING COUNT(*) = 1
);
```

---

# Where CASE WHEN Fits

Interviewers frequently modify the problem by asking you to **label** policies before filtering.

Example

```sql
SELECT
pid,
CASE
WHEN tiv_2015 IN
(
    SELECT tiv_2015
    FROM Insurance
    GROUP BY tiv_2015
    HAVING COUNT(*)>1
)
THEN 'Repeated Value'
ELSE 'Unique Value'
END AS tiv_status
FROM Insurance;
```

Or classify locations:

```sql
CASE
WHEN (lat,lon) IN (...)
THEN 'Unique Location'
ELSE 'Duplicate Location'
END
```

This separates business logic from filtering and makes the query easier to extend.

---

# Why CASE WHEN Is Useful

Real interview questions often evolve.

Instead of simply filtering, you may be asked to report:

- Number of valid policies
- Number of invalid policies
- Percentage of duplicate locations
- Breakdown by policy type

Using CASE allows you to classify rows once and reuse that classification in later aggregations.

---

# Alternative Solution (Using CTEs)

```sql
WITH repeated AS
(
    SELECT tiv_2015
    FROM Insurance
    GROUP BY tiv_2015
    HAVING COUNT(*) > 1
),
unique_location AS
(
    SELECT lat, lon
    FROM Insurance
    GROUP BY lat, lon
    HAVING COUNT(*) = 1
)
SELECT
ROUND(SUM(i.tiv_2016),2) AS tiv_2016
FROM Insurance i
JOIN repeated r
ON i.tiv_2015 = r.tiv_2015
JOIN unique_location u
ON i.lat = u.lat
AND i.lon = u.lon;
```

The CTE version is often preferred for readability during interviews.

---

# Interview Insight

This problem combines **multiple filtering criteria** derived from aggregations.

A common extension is to produce a report such as:

|Policy ID|Status|
|---------:|------|
|1|Eligible|
|2|Rejected|
|3|Eligible|
|4|Rejected|

This is naturally expressed using CASE:

```sql
CASE
WHEN repeated_value
 AND unique_location
THEN 'Eligible'
ELSE 'Rejected'
END
```

---

# Common Mistakes

### Wrong

Filtering duplicate locations with

```sql
DISTINCT
```

`DISTINCT` removes duplicate rows but does **not** identify values that appear exactly once.

---

### Wrong

Using

```sql
COUNT(DISTINCT lat, lon)
```

instead of grouping by both columns.

---

### Wrong

Joining duplicate-value tables incorrectly, leading to duplicate rows in the final sum.

---

# Complexity

| Operation | Complexity |
|-----------|------------|
| GROUP BY | O(n) |
| Subqueries | O(n) |
| Total | O(n log n) |

---

**Next:** Part 4 includes:

- **Key Techniques & Algorithms**
- **LLM-Proof Interview Questions**
- **Company-wise Question Index**
- **Common Interview Mistakes & Revision Checklist**

---

# Key Techniques & Algorithms

The following patterns cover nearly every **CASE WHEN** question asked in SQL interviews. Rather than memorizing solutions, learn to recognize which pattern fits the problem.

---

## 1. Binary Classification

Convert a condition into **1** or **0**.

```sql
CASE
    WHEN salary > 50000 THEN 1
    ELSE 0
END
```

Typical uses:

- Pass / Fail
- Active / Inactive
- Confirmed / Timeout
- Immediate / Scheduled

---

### Example

|Salary|Output|
|------:|-----:|
|60000|1|
|35000|0|

---

## 2. Multi-Level Classification

Assign multiple labels using ordered conditions.

```sql
CASE
    WHEN score >= 90 THEN 'A'
    WHEN score >= 80 THEN 'B'
    WHEN score >= 70 THEN 'C'
    ELSE 'F'
END
```

Typical interview questions:

- Salary categories
- Credit ratings
- Customer segmentation
- Product quality

---

### Execution Order

```
CASE

↓

First WHEN

↓

If True

↓

Stop

↓

Else

↓

Next WHEN
```

CASE stops evaluating after the **first matching condition**.

---

## 3. Conditional Aggregation

One of the most frequently tested interview patterns.

```sql
SUM(
    CASE
        WHEN status='completed'
        THEN 1
        ELSE 0
    END
)
```

Example

|Status|
|------|
|completed|
|pending|
|completed|

Produces

```
2
```

---

### Why It Works

```
completed → 1

pending → 0
```

Then

```
SUM()

adds all successful rows.
```

---

## 4. Conditional Average

Instead of counting successful rows, calculate percentages.

```sql
AVG(
    CASE
        WHEN status='completed'
        THEN 1
        ELSE 0
    END
)
```

Example

Values

```
1
0
1
1
```

Average

```
0.75
```

Meaning

```
75%
```

Appears in

- Confirmation Rate
- Immediate Delivery
- Conversion Rate
- Acceptance Rate

---

## 5. Nested CASE

Business rules often have multiple levels.

```sql
CASE
    WHEN salary >= 100000 THEN
        CASE
            WHEN experience >= 10
            THEN 'Senior'
            ELSE 'High Paid'
        END
    ELSE 'Normal'
END
```

Use nested CASE only when a second decision depends on the first.

---

## 6. CASE Inside ORDER BY

Sort rows using business priority.

```sql
ORDER BY
CASE
    WHEN status='VIP' THEN 1
    WHEN status='Premium' THEN 2
    ELSE 3
END;
```

Instead of alphabetical order

```
Premium
VIP
Regular
```

You obtain

```
VIP
Premium
Regular
```

---

## 7. CASE Inside GROUP BY

Group by derived categories.

```sql
SELECT
CASE
WHEN income<20000 THEN 'Low'
ELSE 'High'
END,
COUNT(*)
FROM Accounts
GROUP BY
CASE
WHEN income<20000 THEN 'Low'
ELSE 'High'
END;
```

Some SQL dialects allow aliases in `GROUP BY`; others require repeating the CASE expression.

---

## 8. CASE with Window Functions

Very common in Hard problems.

```sql
CASE
WHEN ROW_NUMBER()
OVER(
PARTITION BY department
ORDER BY salary DESC
)=1
THEN 'Highest'
END
```

Common combinations

- CASE + ROW_NUMBER()
- CASE + RANK()
- CASE + DENSE_RANK()
- CASE + LAG()
- CASE + LEAD()

---

## 9. CASE with LEFT JOIN

Label missing matches.

```sql
CASE
WHEN orders.order_id IS NULL
THEN 'No Order'
ELSE 'Ordered'
END
```

Frequently tested in:

- Customer reports
- Product inventory
- Employee management

---

## 10. CASE vs WHERE

Many candidates confuse these.

|WHERE|CASE|
|------|----|
|Removes rows|Creates values|
|Filters records|Labels records|
|Executed before SELECT|Evaluated during SELECT|

Example

### WHERE

```sql
WHERE salary>50000
```

Returns only qualifying rows.

---

### CASE

```sql
CASE
WHEN salary>50000
THEN 'High'
ELSE 'Low'
END
```

Returns **every** row with a label.

---

# LLM-Proof Interview Questions

These variations are designed to expose shallow pattern matching. They require careful handling of edge cases and SQL semantics.

---

## Question 1 — Employee Performance Report

### Table

```text
Employees

employee_id
department
salary
performance_score
```

### Requirements

Return

- employee_id
- performance level

Rules

```
salary > 120000

AND

performance >=90

↓

Outstanding
```

```
salary >120000

AND

performance<90

↓

High Salary
```

```
salary<=120000

AND

performance>=90

↓

High Performer
```

Otherwise

```
Standard
```

### Edge Cases

- Salary exactly **120000**
- Performance exactly **90**
- NULL performance scores

Expected solution pattern

```sql
CASE
WHEN ...
WHEN ...
WHEN ...
ELSE ...
END
```

---

## Question 2 — Monthly Revenue Classification

Table

```text
Sales

employee_id
month
revenue
```

Return

|Revenue|Category|
|--------|--------|
|NULL|Missing Data|
|0|No Sales|
|1–9999|Low|
|10000–49999|Medium|
|50000+|High|

### Hidden Pitfalls

Many candidates write

```sql
WHEN revenue=0
```

before checking

```sql
IS NULL
```

Remember:

```sql
NULL = 0
```

is **not** true.

Correct order

```sql
CASE
WHEN revenue IS NULL THEN ...
WHEN revenue=0 THEN ...
WHEN ...
END
```

---

# Company-wise Question Index

| LeetCode | Difficulty | Primary Pattern | Companies |
|-----------|------------|-----------------|-----------|
|620. Not Boring Movies|Easy|Filtering + CASE Labeling|Amazon, Meta, Google, Microsoft|
|1174. Immediate Food Delivery II|Medium|AVG(CASE)|Amazon, Meta, Google|
|1934. Confirmation Rate|Medium|LEFT JOIN + AVG(CASE)|Meta, Amazon, Microsoft|
|601. Human Traffic of Stadium|Hard|Window Functions + CASE Classification|Google, Amazon, Bloomberg|
|1907. Count Salary Categories|Medium|Multi-Level CASE|Amazon, Meta, Google|
|585. Investments in 2016|Hard|Conditional Filtering + CASE Labeling|Google, Amazon, Oracle|

---

# Common Interview Mistakes

## 1. Forgetting ELSE

```sql
CASE
WHEN score>=90
THEN 'A'
END
```

Rows that do not satisfy the condition return `NULL`.

Prefer

```sql
CASE
WHEN score>=90
THEN 'A'
ELSE 'Other'
END
```

---

## 2. Incorrect Condition Order

Wrong

```sql
CASE
WHEN salary>20000
THEN 'Average'
WHEN salary>50000
THEN 'High'
END
```

A salary of **70000** matches the first condition and never reaches the second.

Correct

```sql
CASE
WHEN salary>50000
THEN 'High'
WHEN salary>20000
THEN 'Average'
ELSE 'Low'
END
```

---

## 3. Using `COUNT(CASE...)` Incorrectly

Incorrect

```sql
COUNT(
CASE
WHEN status='completed'
THEN 1
ELSE 0
END
)
```

`COUNT()` counts every non-NULL value, including `0`.

Preferred

```sql
SUM(
CASE
WHEN status='completed'
THEN 1
ELSE 0
END
)
```

Or

```sql
COUNT(
CASE
WHEN status='completed'
THEN 1
END
)
```

---

## 4. Ignoring NULL

Incorrect

```sql
CASE
WHEN salary=NULL
THEN 'Unknown'
END
```

Correct

```sql
CASE
WHEN salary IS NULL
THEN 'Unknown'
END
```

---

## 5. Replacing CASE with WHERE

Incorrect

```sql
SELECT *
FROM Employees
WHERE salary>50000;
```

If the requirement is to **label** employees rather than remove them, use

```sql
CASE
WHEN salary>50000
THEN 'High Salary'
ELSE 'Normal Salary'
END
```

---

# Revision Checklist

Before your interview, ensure you can solve problems involving:

- [ ] Binary classification using `CASE`
- [ ] Multi-level classification
- [ ] `SUM(CASE...)`
- [ ] `AVG(CASE...)`
- [ ] `COUNT(CASE...)`
- [ ] `CASE` with `LEFT JOIN`
- [ ] `CASE` with `GROUP BY`
- [ ] `CASE` with `ORDER BY`
- [ ] `CASE` with window functions
- [ ] Nested `CASE`
- [ ] Proper handling of `NULL`
- [ ] Correct ordering of `WHEN` clauses
- [ ] Business-rule transformations
- [ ] Conditional reporting and dashboards

---

# Final Takeaways

The majority of SQL interview questions involving `CASE WHEN` fall into a small set of reusable patterns:

1. **Classification** — Convert raw values into business categories.
2. **Conditional Aggregation** — Use `SUM(CASE...)` and `AVG(CASE...)` for counts and percentages.
3. **Reporting** — Label records instead of filtering them out.
4. **Business Rules** — Encode priority, thresholds, and multi-step logic directly in SQL.
5. **Composition** — Combine `CASE` with joins, window functions, CTEs, and aggregates to solve more complex interview problems.

If you are comfortable with these patterns and the six LeetCode problems covered in this guide, you will be well prepared for most `CASE WHEN` questions encountered in SQL interviews at FAANG and other major technology companies.
