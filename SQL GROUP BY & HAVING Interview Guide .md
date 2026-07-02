# SQL GROUP BY & HAVING Interview Guide (LeetCode Edition)

> A FAANG-focused interview preparation guide covering **GROUP BY** and **HAVING** through real LeetCode SQL questions. Every section focuses on problem-solving strategies rather than isolated SQL theory.

---

# Table of Contents

- [Question 1 — Customer Placing the Largest Number of Orders (Easy)](#question-1--customer-placing-the-largest-number-of-orders)
- [Question 2 — Classes More Than 5 Students (Easy)](#question-2--classes-more-than-5-students)
- [Question 3 — Department Highest Salary (Medium)](#question-3--department-highest-salary)
- [Question 4 — Confirmation Rate (Medium)](#question-4--confirmation-rate)
- [Question 5 — Managers with at Least 5 Direct Reports (Hard)](#question-5--managers-with-at-least-5-direct-reports)
- [Question 6 — Human Traffic of Stadium (Hard)](#question-6--human-traffic-of-stadium)
- [GROUP BY & HAVING Optimization Patterns](#group-by--having-optimization-patterns)
- [LLM-Proof Questions](#llm-proof-questions)
- [Company Tags Reference](#company-tags-reference)

---

# Question 1 — Customer Placing the Largest Number of Orders

**LeetCode:** 586 (Easy)

## Problem Statement

Table:

```text
Orders
+-------------+------+
| Column Name | Type |
+-------------+------+
| order_number| int  |
| customer_number|int |
+-------------+------+
```

Find the customer who placed the highest number of orders.

Return only the customer_number.

---

## Interview Pattern

This is one of the simplest aggregation problems.

The interviewer wants to check whether you know how to:

- Count records
- Group rows
- Sort aggregated results
- Return the top group

Instead of explaining GROUP BY theoretically, remember this interview template:

```
Rows
      ↓
GROUP BY
      ↓
Aggregate Function
      ↓
ORDER BY Aggregate
      ↓
LIMIT
```

---

## Approach 1 — GROUP BY + COUNT + ORDER BY

This is the expected interview solution.

### Strategy

Step 1

Group every order by customer.

Step 2

Count number of rows in every group.

Step 3

Sort descending.

Step 4

Return first row.

---

### SQL Solution

```sql
SELECT customer_number
FROM Orders
GROUP BY customer_number
ORDER BY COUNT(*) DESC
LIMIT 1;
```

---

## Dry Run

Suppose

|customer|orders|
|--------|------|
|1|3|
|2|6|
|3|2|

After GROUP BY

|customer|COUNT(*)|
|---------|--------|
|1|3|
|2|6|
|3|2|

Sort descending

|customer|count|
|---------|-----|
|2|6|
|1|3|
|3|2|

Return

```
2
```

---

## Approach 2 — Subquery using MAX()

Some interviewers don't like ORDER BY LIMIT because ties are ignored.

Alternative:

```sql
SELECT customer_number
FROM
(
    SELECT customer_number,
           COUNT(*) AS total_orders
    FROM Orders
    GROUP BY customer_number
) t
WHERE total_orders =
(
    SELECT MAX(total_orders)
    FROM
    (
        SELECT COUNT(*) AS total_orders
        FROM Orders
        GROUP BY customer_number
    ) x
);
```

---

### Advantages

- Handles ties naturally
- ANSI SQL compatible
- Easy to extend

---

### Disadvantages

- Two aggregation passes
- More verbose

---

## Approach 3 — Window Function

Modern SQL databases support ranking.

```sql
SELECT customer_number
FROM
(
    SELECT customer_number,
           DENSE_RANK() OVER(
               ORDER BY COUNT(*) DESC
           ) rnk
    FROM Orders
    GROUP BY customer_number
) t
WHERE rnk = 1;
```

Useful if interviewer later says

> Return every customer tied for first place.

---

# Common Interview Mistakes

### Wrong

```sql
SELECT customer_number
FROM Orders
ORDER BY COUNT(*) DESC;
```

COUNT cannot be used before aggregation.

---

### Wrong

```sql
SELECT customer_number,
COUNT(*)
FROM Orders;
```

Missing GROUP BY.

---

### Wrong

```sql
WHERE COUNT(*) > 5
```

Aggregate functions cannot appear inside WHERE.

HAVING must be used.

---

## Optimization

Instead of

```sql
SELECT customer_number,
COUNT(*)
FROM Orders
GROUP BY customer_number
ORDER BY COUNT(*) DESC;
```

Store COUNT once.

```sql
SELECT customer_number,
COUNT(*) AS total_orders
FROM Orders
GROUP BY customer_number
ORDER BY total_orders DESC;
```

Cleaner and easier for optimizer.

---

## Complexity

|Operation|Complexity|
|----------|----------|
|Grouping|O(n)|
|Sorting Groups|O(k log k)|
|Overall|O(n + k log k)|

Where

- n = rows
- k = distinct customers

---

## Follow-up Questions

### If millions of customers exist?

Sorting every customer becomes expensive.

Possible improvement

- Index customer_number
- Use distributed aggregation
- Partial aggregation

---

### What if ties should be returned?

Replace

```sql
LIMIT 1
```

with

```sql
DENSE_RANK()
```

or MAX() subquery.

---

### What if cancelled orders should not count?

```sql
WHERE status='Completed'
GROUP BY customer_number
```

Notice filtering happens **before** grouping.

---

## Companies

|Company|Frequency|
|---------|---------|
|Google|★★★★★|
|Amazon|★★★★★|
|Meta|★★★★☆|
|Microsoft|★★★★☆|
|Apple|★★★☆☆|
|Uber|★★★★☆|

---

# Question 2 — Classes More Than 5 Students

**LeetCode:** 596 (Easy)

## Problem Statement

Table

```text
Courses
+---------+---------+
|student  |class    |
+---------+---------+
```

Each row represents one student enrolled in one class.

Return every class having **at least five students**.

---

## Interview Pattern

This question exists primarily to test whether you know when to use

```
WHERE
```

versus

```
HAVING
```

Interviewers intentionally expect candidates to confuse them.

---

## Approach 1 — GROUP BY + HAVING

### Strategy

Group all students by class.

Count students.

Filter groups.

---

### SQL Solution

```sql
SELECT class
FROM Courses
GROUP BY class
HAVING COUNT(*) >= 5;
```

---

## Dry Run

Input

|Student|Class|
|--------|-----|
|A|Math|
|B|Math|
|C|Math|
|D|Math|
|E|Math|
|F|Physics|

After GROUP BY

|Class|Students|
|------|--------|
|Math|5|
|Physics|1|

HAVING removes Physics.

Output

```
Math
```

---

# Why HAVING?

Incorrect

```sql
SELECT class
FROM Courses
WHERE COUNT(*) >=5
GROUP BY class;
```

COUNT doesn't exist until after grouping.

Execution order

```
FROM

↓

WHERE

↓

GROUP BY

↓

Aggregate

↓

HAVING

↓

SELECT

↓

ORDER BY
```

Understanding this execution order is a common interview checkpoint.

---

## Approach 2 — Using Derived Table

```sql
SELECT class
FROM
(
    SELECT class,
           COUNT(*) AS students
    FROM Courses
    GROUP BY class
) t
WHERE students >=5;
```

Advantages

- Easier debugging
- Helpful when multiple filters exist

---

## Approach 3 — Window Function

```sql
SELECT DISTINCT class
FROM
(
    SELECT class,
           COUNT(*) OVER(PARTITION BY class) cnt
    FROM Courses
) t
WHERE cnt>=5;
```

Useful if later interviewer asks

> Show every student belonging to qualifying classes.

Window functions preserve rows instead of collapsing them.

---

# Optimization

Instead of

```sql
SELECT class
FROM Courses
GROUP BY class
HAVING COUNT(student)>=5;
```

Prefer

```sql
COUNT(*)
```

because

- avoids NULL checks
- usually marginally faster
- communicates intent clearly

---

## Handling Duplicate Enrollments

Suppose

```
Student A

Math

appears twice.
```

Then

```sql
COUNT(*)
```

counts both rows.

If duplicates must be ignored

```sql
SELECT class
FROM Courses
GROUP BY class
HAVING COUNT(DISTINCT student)>=5;
```

Interviewers often introduce this as a follow-up.

---

## Before vs After Optimization

### Before

```sql
SELECT class
FROM Courses
GROUP BY class
HAVING COUNT(student)>=5;
```

### After

```sql
SELECT class
FROM Courses
GROUP BY class
HAVING COUNT(DISTINCT student)>=5;
```

Safer for imperfect datasets.

---

## Complexity

|Operation|Complexity|
|----------|----------|
|Grouping|O(n)|
|Filtering|O(k)|
|Overall|O(n)|

---

## Interview Follow-ups

### Return number of students too

```sql
SELECT class,
       COUNT(*) AS total_students
FROM Courses
GROUP BY class
HAVING COUNT(*)>=5;
```

---

### Ignore NULL student IDs

```sql
COUNT(student)
```

instead of

```sql
COUNT(*)
```

---

### Ignore duplicate enrollments

```sql
COUNT(DISTINCT student)
```

---

### Show top three largest classes

```sql
SELECT class,
COUNT(*) total_students
FROM Courses
GROUP BY class
ORDER BY total_students DESC
LIMIT 3;
```

---

## Common Mistakes

Using WHERE instead of HAVING

```sql
WHERE COUNT(*)>=5
```

Incorrect.

---

Grouping by student

```sql
GROUP BY student
```

Wrong grouping key.

---

Using COUNT(class)

If class is NOT NULL, it works, but

```sql
COUNT(*)
```

is the preferred interview answer.

---

## Companies

|Company|Frequency|
|---------|---------|
|Google|★★★★★|
|Amazon|★★★★★|
|Meta|★★★★★|
|Microsoft|★★★★☆|
|Adobe|★★★★☆|
|Apple|★★★☆☆|

---

# Patterns Learned So Far

|Pattern|Question|
|---------|--------|
|Simple GROUP BY|Customer Placing Largest Number of Orders|
|GROUP BY + ORDER BY Aggregate|Customer Orders|
|GROUP BY + HAVING|Classes More Than 5 Students|
|COUNT(DISTINCT)|Duplicate handling|
|Derived Tables|Both problems|
|Window Functions|Alternative solutions|
|Aggregate Filtering|HAVING|
|Pre-group Filtering|WHERE before GROUP BY|

---

**Next:** **Part 2** covers two of the most frequently asked FAANG medium-level aggregation problems:

1. **Department Highest Salary (LeetCode 184)** — GROUP BY with joins, subqueries, and window functions.
2. **Confirmation Rate (LeetCode 1934)** — conditional aggregation, `LEFT JOIN`, `AVG(CASE WHEN ...)`, handling `NULL`, and advanced interview follow-ups.


# Question 3 — Department Highest Salary

**LeetCode:** 184 (Medium)

---

## Problem Statement

Tables

### Employee

```text
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| name        | varchar |
| salary      | int     |
| departmentId| int     |
+-------------+---------+
```

### Department

```text
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| name        | varchar |
+-------------+---------+
```

Return every employee having the **highest salary** in each department.

Output format

|Department|Employee|Salary|
|----------|--------|------|

---

# Interview Pattern

This is one of the most frequently asked SQL interview questions.

It combines

- GROUP BY
- MAX()
- JOIN
- Correlated Subquery
- Window Functions

Interviewers usually extend it with questions like

- Return Top 3 salaries.
- Handle salary ties.
- Ignore inactive employees.
- Return average salary alongside highest salary.

---

# Approach 1 — GROUP BY + JOIN (Most Common)

### Strategy

First determine the maximum salary inside every department.

Then join back to Employee.

This preserves employees tied for the highest salary.

---

## SQL Solution

```sql
SELECT
    d.name AS Department,
    e.name AS Employee,
    e.salary AS Salary
FROM Employee e
JOIN Department d
    ON e.departmentId = d.id
JOIN
(
    SELECT
        departmentId,
        MAX(salary) AS max_salary
    FROM Employee
    GROUP BY departmentId
) t
ON e.departmentId = t.departmentId
AND e.salary = t.max_salary;
```

---

## Dry Run

Employee

|Employee|Department|Salary|
|---------|----------|------|
|Alice|IT|90000|
|Bob|IT|95000|
|John|HR|70000|
|Mary|HR|70000|

After aggregation

|Department|Max Salary|
|----------|----------|
|IT|95000|
|HR|70000|

Join back

|Department|Employee|Salary|
|----------|--------|------|
|IT|Bob|95000|
|HR|John|70000|
|HR|Mary|70000|

Notice salary ties are preserved automatically.

---

# Approach 2 — Correlated Subquery

Some interviewers specifically ask for a correlated solution.

```sql
SELECT
    d.name AS Department,
    e.name AS Employee,
    e.salary
FROM Employee e
JOIN Department d
ON e.departmentId=d.id
WHERE salary=
(
    SELECT MAX(salary)
    FROM Employee
    WHERE departmentId=e.departmentId
);
```

---

## Advantages

Very readable.

Easy to explain.

---

## Disadvantages

Correlated subqueries may execute repeatedly depending on the optimizer.

Large datasets can become expensive.

---

# Approach 3 — Window Function (Modern SQL)

```sql
SELECT
    Department,
    Employee,
    Salary
FROM
(
    SELECT
        d.name AS Department,
        e.name AS Employee,
        e.salary AS Salary,
        DENSE_RANK() OVER
        (
            PARTITION BY departmentId
            ORDER BY salary DESC
        ) rnk
    FROM Employee e
    JOIN Department d
    ON e.departmentId=d.id
) t
WHERE rnk=1;
```

---

## Why DENSE_RANK instead of ROW_NUMBER?

Suppose

|Employee|Salary|
|---------|------|
|John|90000|
|Mary|90000|

ROW_NUMBER()

```
John → 1

Mary → 2
```

Only John survives.

DENSE_RANK()

```
John → 1

Mary → 1
```

Both survive.

Interviewers often ask this follow-up.

---

# Optimization Comparison

## Less Efficient

```sql
WHERE salary=
(
SELECT MAX(salary)
FROM Employee
WHERE departmentId=e.departmentId
)
```

Repeated lookup.

---

## Better

```sql
JOIN
(
SELECT departmentId,
MAX(salary)
FROM Employee
GROUP BY departmentId
)
```

Aggregate once.

Join once.

---

# Complexity

|Approach|Time|Space|
|---------|----|-----|
|Correlated Subquery|O(n²) worst-case|O(1)|
|GROUP BY + JOIN|O(n)|O(k)|
|Window Function|O(n log n)|O(n)|

k = number of departments.

---

# Follow-up Questions

### Highest salary excluding CEO?

```sql
WHERE designation!='CEO'
```

before aggregation.

---

### Highest salary among active employees?

```sql
WHERE status='Active'
```

before GROUP BY.

---

### Top 3 salaries?

Replace

```sql
DENSE_RANK()=1
```

with

```sql
rnk<=3
```

---

### Department average salary also?

```sql
AVG(salary)
```

inside grouped subquery.

---

# Common Mistakes

Grouping by salary

```sql
GROUP BY departmentId,salary
```

Incorrect.

---

Using MAX without join

```sql
SELECT departmentId,
MAX(salary),
name
```

Illegal SQL.

---

Using ROW_NUMBER()

Fails for salary ties.

---

# Companies

|Company|Frequency|
|---------|---------|
|Google|★★★★★|
|Amazon|★★★★★|
|Meta|★★★★★|
|Microsoft|★★★★★|
|Apple|★★★★☆|
|Adobe|★★★★☆|
|Oracle|★★★★★|

---

# Question 4 — Confirmation Rate

**LeetCode:** 1934 (Medium)

---

## Problem Statement

Tables

### Signups

```text
+---------+------+
|user_id  |time  |
+---------+------+
```

### Confirmations

```text
+-----------+-----------+-----------+
|user_id    |time       |action     |
+-----------+-----------+-----------+
```

action is either

```
confirmed

or

timeout
```

Return the confirmation rate of every user.

Formula

```
confirmed actions

--------------------

all confirmation actions
```

If a user never requested confirmation,

```
rate = 0
```

Round to two decimal places.

---

# Interview Pattern

This problem tests

- GROUP BY
- LEFT JOIN
- Conditional Aggregation
- AVG()
- CASE WHEN
- NULL handling

One of Meta's favorite SQL interview questions.

---

# Approach 1 — AVG(CASE WHEN...)

This is the cleanest interview solution.

---

## Strategy

LEFT JOIN

↓

One row per confirmation

↓

Confirmed

```
1
```

Timeout

```
0
```

Average

```
Sum / Count
```

automatically.

---

## SQL Solution

```sql
SELECT
    s.user_id,
    ROUND(
        AVG
        (
            CASE
                WHEN action='confirmed'
                THEN 1
                ELSE 0
            END
        ),
        2
    ) AS confirmation_rate
FROM Signups s
LEFT JOIN Confirmations c
ON s.user_id=c.user_id
GROUP BY s.user_id;
```

---

## Why AVG Works

Rows

|Action|Converted|
|------|---------|
|confirmed|1|
|confirmed|1|
|timeout|0|

Average

```
(1+1+0)/3

=

0.67
```

Exactly the required answer.

---

# Problem

Users without confirmations return

```
NULL
```

Need

```
0
```

---

## Correct Solution

```sql
SELECT
    s.user_id,
    ROUND
    (
        COALESCE
        (
            AVG
            (
                CASE
                    WHEN action='confirmed'
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
ON s.user_id=c.user_id
GROUP BY s.user_id;
```

---

# Approach 2 — SUM / COUNT

Equivalent solution.

```sql
SELECT
    s.user_id,
    ROUND
    (
        COALESCE
        (
            SUM
            (
                CASE
                    WHEN action='confirmed'
                    THEN 1
                    ELSE 0
                END
            )
            /
            COUNT(c.action),
            0
        ),
        2
    ) confirmation_rate
FROM Signups s
LEFT JOIN Confirmations c
ON s.user_id=c.user_id
GROUP BY s.user_id;
```

---

## Comparison

|AVG(CASE)|SUM/COUNT|
|----------|---------|
|Cleaner|More flexible|
|Shorter|Explicit formula|
|Less error-prone|Easy to customize|

Interview preference

✅ AVG(CASE)

---

# Alternative Using CTE

```sql
WITH UserStats AS
(
SELECT
user_id,
AVG
(
CASE
WHEN action='confirmed'
THEN 1
ELSE 0
END
) rate
FROM Confirmations
GROUP BY user_id
)

SELECT
s.user_id,
ROUND(COALESCE(rate,0),2)
FROM Signups s
LEFT JOIN UserStats u
ON s.user_id=u.user_id;
```

Easy to debug.

Useful in production.

---

# Before vs After Optimization

### Before

```sql
SUM(CASE...)
/
COUNT(CASE...)
```

---

### After

```sql
AVG(CASE...)
```

Fewer expressions.

Cleaner execution plan.

---

# Handling NULL Values

Without COALESCE

```
NULL
```

Returned.

With COALESCE

```
0
```

Returned.

Very common interview trap.

---

# Complexity

|Operation|Complexity|
|----------|----------|
|LEFT JOIN|O(n)|
|Grouping|O(n)|
|Overall|O(n)|

---

# Interview Follow-ups

### Return percentage instead of decimal

```sql
ROUND
(
AVG(...)
*100,
2
)
```

---

### Ignore expired confirmations

```sql
WHERE expired=0
```

before grouping.

---

### Calculate daily confirmation rate

```sql
GROUP BY
user_id,
confirmation_date
```

---

### Return users with confirmation rate above 80%

```sql
HAVING AVG(...)>0.8
```

---

# Common Mistakes

Using INNER JOIN

Removes users without confirmations.

Incorrect.

---

Forgetting COALESCE

Returns NULL.

---

Using WHERE action='confirmed'

Removes timeout rows.

Denominator becomes incorrect.

---

# Companies

|Company|Frequency|
|---------|---------|
|Meta|★★★★★|
|Google|★★★★★|
|Amazon|★★★★☆|
|Microsoft|★★★★☆|
|Apple|★★★★☆|
|LinkedIn|★★★★☆|
|Uber|★★★★☆|

---

# Patterns Learned So Far

|Pattern|Problem|
|---------|-------|
|GROUP BY + ORDER BY|Largest Number of Orders|
|GROUP BY + HAVING|Classes More Than 5 Students|
|GROUP BY + JOIN|Department Highest Salary|
|Conditional Aggregation|Confirmation Rate|
|AVG(CASE WHEN)|Confirmation Rate|
|MAX() per Group|Department Highest Salary|
|LEFT JOIN + GROUP BY|Confirmation Rate|
|Window Functions|Department Highest Salary|
|Handling Ties|DENSE_RANK()|
|Handling NULL Groups|COALESCE()|

---

**Next (Part 3):**

- **Question 5 — Managers with at Least 5 Direct Reports (Hard)** (GROUP BY, HAVING, Self Join)
- **Question 6 — Human Traffic of Stadium (Hard)** (consecutive rows, window functions, grouping tricks, interview variants)

# Question 5 — Managers with at Least 5 Direct Reports

**LeetCode:** 570 (Hard)

---

## Problem Statement

Table

### Employee

```text
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| name        | varchar |
| department  | varchar |
| managerId   | int     |
+-------------+---------+
```

Each employee reports to exactly one manager (or `NULL` if they have no manager).

Return the names of managers who have **at least five direct reports**.

---

# Interview Pattern

This question combines several interview fundamentals:

- Self Join
- GROUP BY
- HAVING
- Aggregate Filtering
- Employee Hierarchies

A common follow-up is:

> What if the company hierarchy contains millions of employees?

Understanding how aggregation works with self-joins is more important than memorizing syntax.

---

# Approach 1 — Self Join + GROUP BY (Expected Solution)

## Strategy

Think of the Employee table twice.

```
Employee

↓

Employee acting as Manager

↓

Employee acting as Direct Report
```

Join employees to their manager.

Count reports for every manager.

Filter managers having at least five reports.

---

## SQL Solution

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

## Dry Run

Employee Table

|Employee|Manager|
|---------|-------|
|A|John|
|B|John|
|C|John|
|D|John|
|E|John|
|F|Mike|

After Join

|Manager|Employee|
|--------|--------|
|John|A|
|John|B|
|John|C|
|John|D|
|John|E|
|Mike|F|

Grouping

|Manager|Reports|
|--------|-------|
|John|5|
|Mike|1|

HAVING removes Mike.

Output

```
John
```

---

# Approach 2 — GROUP BY First

Instead of joining immediately, count reports first.

```sql
SELECT
    name
FROM Employee
WHERE id IN
(
    SELECT
        managerId
    FROM Employee
    GROUP BY managerId
    HAVING COUNT(*) >= 5
);
```

---

## Advantages

- Cleaner aggregation
- Smaller intermediate result
- Often easier for optimizers

---

## Disadvantages

Requires nested query.

---

# Approach 3 — Common Table Expression

```sql
WITH ManagerCounts AS
(
    SELECT
        managerId,
        COUNT(*) AS reports
    FROM Employee
    GROUP BY managerId
)

SELECT
    e.name
FROM Employee e
JOIN ManagerCounts m
ON e.id = m.managerId
WHERE reports >= 5;
```

Very readable.

Preferred in production SQL.

---

# Optimization

### Less Efficient

```sql
SELECT name
FROM Employee
WHERE id IN (...)
```

The optimizer may rewrite it, but explicit joins are generally easier to reason about.

---

### Better

```sql
JOIN

↓

GROUP BY

↓

HAVING
```

Single aggregation.

Single join.

---

# Handling NULL Managers

Suppose

```
CEO

managerId = NULL
```

Grouping directly gives

|managerId|count|
|----------|-----|
|NULL|1|

That group is meaningless.

Filter before grouping.

```sql
SELECT
    managerId,
    COUNT(*)
FROM Employee
WHERE managerId IS NOT NULL
GROUP BY managerId;
```

Small optimization that interviewers appreciate.

---

# Complexity

|Approach|Time|Space|
|---------|----|-----|
|Self Join + GROUP BY|O(n)|O(k)|
|Subquery|O(n)|O(k)|
|CTE|O(n)|O(k)|

k = distinct managers.

---

# Common Mistakes

### Grouping by employee id

```sql
GROUP BY e.id
```

Wrong.

Need one group per manager.

---

### Using WHERE COUNT(*)

```sql
WHERE COUNT(*) >= 5
```

Aggregate filters belong inside HAVING.

---

### Forgetting Self Join

Returning

```
managerId
```

instead of manager name.

---

# Follow-up Questions

### Managers with exactly five reports

```sql
HAVING COUNT(*) = 5
```

---

### Managers with more than ten reports

```sql
HAVING COUNT(*) > 10
```

---

### Count only active employees

```sql
WHERE status='Active'
```

before GROUP BY.

---

### Count employees hired after 2023

```sql
WHERE hire_date >= '2023-01-01'
```

before aggregation.

---

### Return report count too

```sql
SELECT
    m.name,
    COUNT(*) AS reports
```

---

# Companies

|Company|Frequency|
|---------|---------|
|Google|★★★★★|
|Amazon|★★★★★|
|Meta|★★★★★|
|Microsoft|★★★★★|
|Apple|★★★★☆|
|Salesforce|★★★★☆|
|Oracle|★★★★☆|

---

# Question 6 — Human Traffic of Stadium

**LeetCode:** 601 (Hard)

---

## Problem Statement

Table

```text
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| visit_date  | date    |
| people      | int     |
+-------------+---------+
```

Return all rows belonging to **three or more consecutive ids** where

```
people >= 100
```

Output should include every row from qualifying sequences.

---

# Interview Pattern

This problem is famous because it mixes

- Filtering
- Window Functions
- Consecutive Groups
- Gap-and-Island Technique

Many candidates know GROUP BY but fail to recognize that GROUP BY alone cannot identify consecutive rows.

---

# Approach 1 — Gap and Island (Recommended)

## Strategy

Step 1

Keep only rows

```
people >=100
```

Step 2

Generate

```
ROW_NUMBER()
```

Step 3

Compute

```
id - row_number
```

Rows belonging to the same consecutive block produce the same value.

Example

|id|Row Number|Difference|
|--|----------|----------|
|5|1|4|
|6|2|4|
|7|3|4|

Same difference

↓

Same island.

---

## SQL Solution

```sql
WITH Filtered AS
(
    SELECT
        *,
        ROW_NUMBER() OVER(ORDER BY id) AS rn
    FROM Stadium
    WHERE people >= 100
),

Groups AS
(
    SELECT
        *,
        id - rn AS grp
    FROM Filtered
)

SELECT
    id,
    visit_date,
    people
FROM Groups
WHERE grp IN
(
    SELECT
        grp
    FROM Groups
    GROUP BY grp
    HAVING COUNT(*) >= 3
)
ORDER BY id;
```

---

# Dry Run

Input

|id|people|
|--|------|
|1|10|
|2|150|
|3|170|
|4|120|
|5|80|

Filtered

|id|
|--|
|2|
|3|
|4|

Row Numbers

|id|rn|
|--|--|
|2|1|
|3|2|
|4|3|

Difference

|id-rn|
|-----|
|1|
|1|
|1|

One island.

Count

```
3

↓

Return all rows.
```

---

# Why GROUP BY Works Here

Notice we are **not grouping by id**.

Instead we group by

```
id - row_number
```

That transformed value identifies one consecutive sequence.

This is one of the most useful SQL interview tricks.

---

# Alternative Approach — LAG / LEAD

Another solution uses

```sql
LAG()
```

and

```sql
LEAD()
```

to inspect neighboring rows.

Although possible, it becomes much more complicated when sequences exceed three rows.

Gap-and-Island is preferred.

---

# Optimization

### Before

Repeated self joins

```sql
id+1

id+2
```

Works only for exactly three rows.

Fails for longer sequences.

---

### After

Gap-and-Island

Automatically handles

- 3 rows
- 4 rows
- 10 rows
- 100 rows

without code changes.

---

# Complexity

|Approach|Time|Space|
|---------|----|-----|
|Gap-and-Island|O(n log n)|O(n)|
|Multiple Self Joins|O(n²)|O(1)|

Sorting dominates due to the window function.

---

# Common Mistakes

### Grouping by id

Produces one row per group.

No consecutive detection.

---

### Using LIMIT 3

Returns only three rows.

Question requires **every row** in qualifying sequences.

---

### Filtering after grouping

Always filter

```sql
people >=100
```

before building consecutive groups.

---

# Follow-up Questions

### Four consecutive rows

Simply change

```sql
HAVING COUNT(*) >=4
```

---

### Consecutive dates instead of ids

Replace

```sql
id
```

with

```sql
DATEDIFF()
```

or date arithmetic.

---

### Threshold becomes dynamic

Example

```
people >= variable_limit
```

Only the WHERE clause changes.

Gap-and-Island logic remains identical.

---

### Return only starting date

```sql
MIN(visit_date)
```

per island.

---

### Return longest sequence

Use

```sql
COUNT(*)
```

and

```sql
ORDER BY COUNT(*) DESC
LIMIT 1
```

after grouping.

---

# Companies

|Company|Frequency|
|---------|---------|
|Google|★★★★★|
|Meta|★★★★★|
|Amazon|★★★★☆|
|Microsoft|★★★★★|
|Apple|★★★★☆|
|Snowflake|★★★★☆|
|Databricks|★★★★☆|

---

# Patterns Learned So Far

|Pattern|Question|
|---------|--------|
|Simple GROUP BY|Largest Number of Orders|
|GROUP BY + HAVING|Classes More Than 5 Students|
|MAX() per Group|Department Highest Salary|
|Conditional Aggregation|Confirmation Rate|
|AVG(CASE WHEN)|Confirmation Rate|
|Self Join + GROUP BY|Managers with Direct Reports|
|Hierarchy Aggregation|Managers with Direct Reports|
|Gap-and-Island|Human Traffic of Stadium|
|ROW_NUMBER()|Human Traffic of Stadium|
|Window Functions + GROUP BY|Human Traffic of Stadium|
|Handling Salary Ties|Department Highest Salary|
|LEFT JOIN Aggregation|Confirmation Rate|
|COUNT(DISTINCT)|Classes More Than 5 Students|

---

**Next (Final Part):**

- GROUP BY & HAVING optimization patterns
- WHERE vs HAVING decision framework
- Aggregate function cheat sheet
- Performance optimization and indexing
- Before vs After query optimizations
- Common interview pitfalls
- Complexity comparison table
- LLM-Proof Questions
- Company Tags Reference

  # GROUP BY & HAVING Optimization Patterns

This section summarizes the interview patterns that repeatedly appear in FAANG SQL interviews. Instead of memorizing individual solutions, learn **when** each pattern applies.

---

# Execution Order Cheat Sheet

Understanding SQL execution order explains why many interview mistakes happen.

```text
FROM

↓

JOIN

↓

WHERE

↓

GROUP BY

↓

Aggregate Functions

↓

HAVING

↓

SELECT

↓

DISTINCT

↓

ORDER BY

↓

LIMIT
```

---

# WHERE vs HAVING

One of the most common interview questions.

|WHERE|HAVING|
|------|-------|
|Filters individual rows|Filters groups|
|Runs before GROUP BY|Runs after GROUP BY|
|Cannot use aggregates|Can use aggregates|
|Usually faster|Usually more expensive|

---

## Example

### Before (Incorrect)

```sql
SELECT departmentId
FROM Employee
WHERE COUNT(*) >= 5
GROUP BY departmentId;
```

`COUNT()` is not available during the `WHERE` phase.

---

### After (Correct)

```sql
SELECT departmentId
FROM Employee
GROUP BY departmentId
HAVING COUNT(*) >= 5;
```

---

# Push Filters Before GROUP BY

Reduce the number of rows before aggregation whenever possible.

### Less Efficient

```sql
SELECT departmentId,
       COUNT(*)
FROM Employee
GROUP BY departmentId
HAVING SUM(CASE WHEN status='Active' THEN 1 ELSE 0 END) > 5;
```

---

### Better

```sql
SELECT departmentId,
       COUNT(*)
FROM Employee
WHERE status='Active'
GROUP BY departmentId
HAVING COUNT(*) > 5;
```

Advantages

- Smaller input
- Faster grouping
- Lower memory usage

---

# COUNT(*) vs COUNT(column)

Interviewers frequently ask the difference.

Example

|id|salary|
|--|------|
|1|50000|
|2|NULL|
|3|60000|

### COUNT(*)

```sql
SELECT COUNT(*)
FROM Employee;
```

Result

```
3
```

---

### COUNT(salary)

```sql
SELECT COUNT(salary)
FROM Employee;
```

Result

```
2
```

NULL values are ignored.

---

### COUNT(DISTINCT)

```sql
SELECT COUNT(DISTINCT departmentId)
FROM Employee;
```

Counts unique values only.

---

# Aggregate Function Cheat Sheet

|Function|Purpose|
|----------|-------|
|COUNT(*)|Total rows|
|COUNT(column)|Non-NULL values|
|COUNT(DISTINCT)|Unique values|
|SUM()|Total|
|AVG()|Average|
|MIN()|Smallest value|
|MAX()|Largest value|

---

# Conditional Aggregation

Instead of filtering rows, count conditions.

### Pattern

```sql
SUM(
CASE
WHEN condition
THEN 1
ELSE 0
END
)
```

Example

```sql
SELECT
departmentId,
SUM(
CASE
WHEN salary>100000
THEN 1
ELSE 0
END
)
FROM Employee
GROUP BY departmentId;
```

---

# AVG(CASE WHEN)

Useful for percentages.

```sql
AVG(
CASE
WHEN status='Success'
THEN 1
ELSE 0
END
)
```

Equivalent to

```
Successful Rows

-----------------

Total Rows
```

This pattern appeared in **Confirmation Rate**.

---

# Multiple Aggregates Together

```sql
SELECT
departmentId,
COUNT(*) AS employees,
AVG(salary) AS average_salary,
MIN(salary) AS minimum_salary,
MAX(salary) AS maximum_salary
FROM Employee
GROUP BY departmentId;
```

Common interview follow-up.

---

# GROUP BY Multiple Columns

```sql
GROUP BY

departmentId,

designation
```

Produces groups like

|Department|Role|
|-----------|----|
|IT|Developer|
|IT|Manager|
|HR|Recruiter|

Useful for multidimensional reports.

---

# Nested Aggregation Pattern

Sometimes aggregate twice.

Example

```
Highest average salary among departments.
```

```sql
SELECT MAX(avg_salary)
FROM
(
SELECT
departmentId,
AVG(salary) avg_salary
FROM Employee
GROUP BY departmentId
) t;
```

Interviewers often test this pattern.

---

# HAVING Without GROUP BY

Valid SQL.

```sql
SELECT COUNT(*)
FROM Employee
HAVING COUNT(*) > 100;
```

Entire table becomes one implicit group.

Rare, but worth knowing.

---

# Window Functions vs GROUP BY

|GROUP BY|Window Function|
|----------|---------------|
|Collapses rows|Keeps rows|
|One row per group|Original rows preserved|
|Used for summaries|Used for rankings and analytics|

---

## GROUP BY

```sql
SELECT
departmentId,
AVG(salary)
FROM Employee
GROUP BY departmentId;
```

Output

One row per department.

---

## Window Function

```sql
SELECT
name,
salary,
AVG(salary)
OVER(PARTITION BY departmentId)
FROM Employee;
```

Output

Every employee remains visible.

---

# Subquery vs Window Function

|Subquery|Window Function|
|----------|---------------|
|Portable|Modern SQL|
|Readable|Often shorter|
|Extra joins|Usually fewer joins|
|Good for aggregates|Excellent for rankings|

---

# Indexing Tips

Indexes cannot eliminate aggregation, but they reduce scanning costs.

Useful indexes

```text
departmentId

managerId

customer_number

class

salary
```

Composite indexes

```text
(departmentId, salary)

(managerId, employeeId)

(customer_number, order_date)
```

These commonly appear in interview discussions.

---

# Common Interview Pitfalls

## Mistake 1

Selecting non-grouped columns.

Incorrect

```sql
SELECT
departmentId,
name,
COUNT(*)
FROM Employee
GROUP BY departmentId;
```

`name` is neither grouped nor aggregated.

---

## Mistake 2

Using aggregate inside WHERE.

Incorrect

```sql
WHERE AVG(salary)>50000
```

Correct

```sql
HAVING AVG(salary)>50000
```

---

## Mistake 3

Grouping too early.

Filter rows first.

Aggregate later.

---

## Mistake 4

Ignoring NULL values.

Remember

```sql
COUNT(column)
```

ignores NULL.

---

## Mistake 5

Using ROW_NUMBER instead of DENSE_RANK.

ROW_NUMBER removes ties.

DENSE_RANK preserves them.

---

# Complexity Comparison

|Operation|Average Complexity|
|----------|------------------|
|GROUP BY (Hash Aggregate)|O(n)|
|GROUP BY (Sort Aggregate)|O(n log n)|
|COUNT()|O(n)|
|AVG()|O(n)|
|SUM()|O(n)|
|MAX()|O(n)|
|MIN()|O(n)|
|Window Function|O(n log n)|
|Correlated Subquery|O(n²) worst-case|

---

# Choosing the Right Pattern

|Problem Type|Recommended Pattern|
|-------------|-------------------|
|Count per group|GROUP BY + COUNT|
|Filter groups|HAVING|
|Highest value per group|MAX() + JOIN or DENSE_RANK()|
|Percentages|AVG(CASE WHEN)|
|Conditional counts|SUM(CASE WHEN)|
|Hierarchies|Self Join + GROUP BY|
|Top N per group|DENSE_RANK()|
|Consecutive rows|Gap-and-Island|
|Keep original rows|Window Functions|

---

# LLM-Proof Questions

These follow-up questions are commonly used to distinguish genuine understanding from memorized or AI-generated solutions.

---

## Question 1

You solved **Customer Placing the Largest Number of Orders** using `ORDER BY COUNT(*) DESC LIMIT 1`.

Now modify it to:

- Return **all customers tied** for the highest number of orders.
- Do **not** use `LIMIT`.
- Avoid scanning the table more than necessary.

Expected concepts

- `DENSE_RANK()`
- Derived tables
- `MAX()`
- Tie handling

---

## Question 2

For **Classes More Than 5 Students**:

The `Courses` table contains duplicate enrollments because of a data bug.

Example

|Student|Class|
|--------|-----|
|Alice|Math|
|Alice|Math|
|Bob|Math|

Tasks

- Count each student only once.
- Ignore NULL student IDs.
- Return classes with at least five unique students.

Expected concepts

- `COUNT(DISTINCT student)`
- NULL handling
- Aggregate filtering

---

## Question 3

For **Department Highest Salary**:

Requirements change.

Return

- Top 3 salaries per department.
- Preserve salary ties.
- Exclude inactive employees.
- Display department average salary alongside each employee.

Expected concepts

- `DENSE_RANK()`
- `PARTITION BY`
- Window aggregates
- Pre-filtering with `WHERE`

---

## Question 4

For **Confirmation Rate**:

The interviewer asks:

- Compute confirmation rate **per month**.
- Ignore requests older than one year.
- Users with zero requests should still appear.
- Return users whose rate exceeds 80%.

Expected concepts

- `LEFT JOIN`
- `AVG(CASE WHEN)`
- `COALESCE`
- `GROUP BY user_id, month`
- `HAVING`

---

## Question 5

For **Managers with at Least 5 Direct Reports**:

Return

- Manager name
- Report count
- Average salary of direct reports
- Highest-paid report
- Alphabetically first report

Expected concepts

- Multiple aggregates
- Self Join
- `GROUP BY`
- `MIN()`
- `MAX()`
- `AVG()`

---

## Question 6

For **Human Traffic of Stadium**:

The requirement changes.

Instead of consecutive IDs,

return

- Consecutive dates.
- Ignore weekends.
- Minimum sequence length becomes configurable.
- Return only the first and last day of each qualifying sequence.

Expected concepts

- Gap-and-Island
- Date arithmetic
- Window functions
- Aggregate summaries

---

# Company Tags Reference

|Question|Google|Meta|Amazon|Microsoft|Apple|Oracle|Uber|Adobe|Salesforce|
|----------|:----:|:---:|:-----:|:--------:|:----:|:-----:|:---:|:----:|:---------:|
|Customer Placing the Largest Number of Orders|✔|✔|✔|✔|✔|—|✔|—|—|
|Classes More Than 5 Students|✔|✔|✔|✔|✔|—|—|✔|—|
|Department Highest Salary|✔|✔|✔|✔|✔|✔|—|✔|—|
|Confirmation Rate|✔|✔|✔|✔|✔|—|✔|—|—|
|Managers with at Least 5 Direct Reports|✔|✔|✔|✔|✔|✔|—|—|✔|
|Human Traffic of Stadium|✔|✔|✔|✔|✔|—|—|—|—|

> **Note:** Company tags are aggregated from public interview reports and community discussion forums. Interview content changes over time, so treat these as indicative rather than exhaustive.

---

# Final Revision Checklist

Before a SQL interview, ensure you can solve the following without reference material:

- GROUP BY with one or multiple columns
- WHERE vs HAVING
- COUNT(*), COUNT(column), COUNT(DISTINCT)
- MAX(), MIN(), AVG(), SUM()
- Conditional aggregation using `CASE`
- `AVG(CASE WHEN ...)` for percentages
- GROUP BY with JOINs
- Self Join with aggregation
- Derived tables and CTEs
- Window functions (`ROW_NUMBER`, `RANK`, `DENSE_RANK`)
- Handling salary ties correctly
- `COALESCE` for NULL-safe aggregation
- Gap-and-Island for consecutive sequence problems
- Choosing between subqueries, joins, and window functions based on readability and performance

---

# Summary

Mastering `GROUP BY` and `HAVING` is less about memorizing syntax and more about recognizing recurring patterns. Across FAANG interviews, the same themes appear repeatedly:

- Aggregate first, then filter with `HAVING`.
- Push row-level filters into `WHERE` whenever possible.
- Use conditional aggregation for counts and percentages.
- Prefer `DENSE_RANK()` when ties must be preserved.
- Combine `GROUP BY` with joins, CTEs, and window functions to solve more complex reporting problems.

These six LeetCode problems collectively cover the majority of aggregation patterns encountered in technical SQL interviews and provide a strong foundation for medium-to-hard SQL coding rounds.
