# SQL Set Operations (UNION, INTERSECT, EXCEPT) — FAANG LeetCode Interview Guide

> A focused interview-preparation guide centered on **LeetCode SQL problems** that require thinking in terms of **set operations** rather than joins alone.

---

# Table of Contents

- [Interview Pattern](#interview-pattern)
- [Question 1 — Customers Who Never Order](#question-1--customers-who-never-order)
- [Question 2 — Employees Earning More Than Their Managers](#question-2--employees-earning-more-than-their-managers)
- [Question 3 — Friend Requests II: Who Has the Most Friends](#question-3--friend-requests-ii-who-has-the-most-friends)
- [Question 4 — Restaurant Growth](#question-4--restaurant-growth)
- [Question 5 — Human Traffic of Stadium](#question-5--human-traffic-of-stadium)
- [Question 6 — Tree Node (LLM-Proof Variant)](#question-6--tree-node-llm-proof-variant)
- [Interview Cheat Sheet](#interview-cheat-sheet)

---

# Interview Pattern

**Frequently Asked By**

| Company | Frequency |
|----------|-----------|
| Google | ⭐⭐⭐⭐⭐ |
| Meta | ⭐⭐⭐⭐ |
| Amazon | ⭐⭐⭐⭐ |
| Microsoft | ⭐⭐⭐⭐ |
| Apple | ⭐⭐⭐ |
| Uber | ⭐⭐⭐ |
| Airbnb | ⭐⭐⭐ |
| Bloomberg | ⭐⭐⭐ |

---

Most candidates think of SQL as **JOINs**.

FAANG interviewers often expect you to think in **sets**.

Instead of asking:

> "How do I join these tables?"

they expect:

> "Which rows belong to one set but not another?"

That mental shift immediately leads to:

- UNION
- UNION ALL
- INTERSECT
- EXCEPT (MINUS)

Many LeetCode problems don't explicitly require these keywords, but they become dramatically simpler when viewed as set problems.

---

# Question 1 — Customers Who Never Order

**LeetCode:** 183

**Difficulty:** Easy

**Pattern**

Finding rows present in one table but absent in another.

This is the classic **EXCEPT** pattern.

---

## Core Challenge

Given

Customers

```
1 Alice
2 Bob
3 Tom
```

Orders

```
1
3
```

Need

```
Bob
```

Conceptually

```
Customers
──────────────
Alice
Bob
Tom

EXCEPT

Customers with Orders
─────────────────────
Alice
Tom
```

Result

```
Bob
```

---

## Set Thinking

Instead of

```
LEFT JOIN
```

Think

```
All Customers

MINUS

Customers Who Ordered
```

Exactly what EXCEPT does.

---

## SQL

```sql
SELECT name
FROM Customers

EXCEPT

SELECT c.name
FROM Customers c
JOIN Orders o
ON c.id=o.customerId;
```

---

## If EXCEPT Isn't Supported

```sql
SELECT name
FROM Customers
WHERE id NOT IN (
    SELECT customerId
    FROM Orders
);
```

---

## Interview Discussion

The interviewer is testing whether you recognize

```
Universe − Subset
```

instead of immediately reaching for joins.

---

## Optimization

Prefer

```
NOT EXISTS
```

over

```
NOT IN
```

when NULL values are possible.

---

## Common Mistake

Using

```
UNION
```

instead of

```
EXCEPT

```

UNION combines.

EXCEPT removes.

---

# Question 2 — Employees Earning More Than Their Managers

**LeetCode:** 181

**Difficulty:** Easy

---

Although usually solved with a self join, this question can be reframed as comparing two sets.

Set A

```
Employee Salary
```

Set B

```
Manager Salary
```

Need

```
Employees

WHERE

Salary > Manager Salary
```

---

## Why This Matters

Interviewers often extend this problem into

> "Find employees that exist in one salary category but not another."

which becomes a natural EXCEPT problem.

---

## Set Perspective

```
High Salary Employees

EXCEPT

Employees whose salary
doesn't exceed manager
```

Thinking in sets makes later follow-up questions easier.

---

## Optimization Trick

Do comparison before projection.

Avoid selecting unnecessary columns before applying EXCEPT.

---

## FAANG Follow-up

Return

```
Employees

NOT

Managers
```

Now EXCEPT becomes the cleanest solution.

---

# Question 3 — Friend Requests II: Who Has the Most Friends

**LeetCode:** 602

**Difficulty:** Medium

---

This question secretly revolves around UNION ALL.

Table

```
requester accepter
```

Friendship is bidirectional.

Need both columns to represent users.

---

Instead of

```
Requester only
```

Need

```
Requester

UNION ALL

Accepter
```

---

## Core Transformation

Original

```
1 -> 2
```

becomes

```
1

2
```

for counting.

---

## SQL

```sql
SELECT requester_id AS id
FROM RequestAccepted

UNION ALL

SELECT accepter_id
FROM RequestAccepted;
```

Then

```
GROUP BY id
```

---

## Why UNION ALL?

Removing duplicates is wrong.

Each friendship counts.

UNION would silently remove duplicate rows.

---

## Interview Trick

Whenever a graph edge

```
A ---- B
```

must become

```
A

B
```

think

```
UNION ALL
```

---

## Common Pitfall

Candidates instinctively write

```
UNION
```

and lose duplicate friendships.

---

# Question 4 — Restaurant Growth

**LeetCode:** 1321

**Difficulty:** Medium

---

Although famous for window functions, interviewers sometimes modify it.

Example follow-up:

Find

```
Days with revenue

OR

Special promotion days
```

This becomes

```
Revenue Days

UNION

Promotion Days
```

---

## Set Pattern

```
Set A

Revenue Days

UNION

Holiday Days

UNION

Special Event Days
```

Instead of multiple OR conditions.

---

## Why Interviewers Like It

Candidates who know UNION often produce much cleaner queries than enormous WHERE clauses.

---

## Optimization

Multiple OR predicates

↓

can sometimes become

```
UNION ALL
```

↓

allowing indexes to be used separately.

This is a common SQL optimization.

---

# Question 5 — Human Traffic of Stadium

**LeetCode:** 601

**Difficulty:** Hard

---

Most solutions use window functions.

FAANG follow-up:

Return

```
Rows satisfying

Condition A

OR

Condition B
```

without duplicates.

---

Instead of

```
WHERE
conditionA
OR
conditionB
```

use

```
SELECT ...

UNION

SELECT ...
```

---

## Why?

Each branch becomes independently optimized.

Large databases often execute this faster.

---

## Query Flow

```
High Attendance

UNION

Festival Attendance

↓

Distinct Days

↓

Window Logic
```

---

## Optimization

UNION performs duplicate elimination.

If duplicates are impossible,

```
UNION ALL
```

is significantly faster.

---

## Common Mistake

Choosing UNION by habit.

Removing duplicates requires sorting or hashing.

That is expensive.

---

# Question 6 — Tree Node (LLM-Proof Variant)

**LeetCode:** 608

**Difficulty:** Medium

---

This question becomes surprisingly difficult when interviewers forbid CASE expressions.

Instead they ask

Return

```
Root

Inner

Leaf
```

using only set operations.

---

## Solution Strategy

Build three independent sets.

Root

```
parent IS NULL
```

Leaf

```
id

EXCEPT

parent values
```

Inner

```
All Nodes

EXCEPT

(Root UNION Leaf)
```

---

Visualization

```
All Nodes
─────────────

      EXCEPT

Root
UNION
Leaf

─────────────

Inner Nodes
```

---

## Why This Is LLM-Proof

Many language models default to CASE expressions.

Interviewers occasionally forbid CASE to test whether candidates understand relational algebra.

The candidate who thinks in sets finishes immediately.

---

## Optimization

Construct

```
Root

Leaf

Inner
```

independently.

Avoid repeated scans of the same table.

---

# Interview Cheat Sheet

| Problem Pattern | Best Operator |
|-----------------|--------------|
| Merge two result sets | UNION |
| Merge while keeping duplicates | UNION ALL |
| Common rows | INTERSECT |
| Present in first but not second | EXCEPT |
| Symmetric graph transformation | UNION ALL |
| Replace complex OR conditions | UNION ALL (sometimes faster) |
| Remove processed records | EXCEPT |
| Build complementary subsets | EXCEPT + UNION |

---

# Fast Recognition Rules

| If You Hear... | Think... |
|---------------|----------|
| Combine two queries | UNION |
| Keep duplicates | UNION ALL |
| Only common rows | INTERSECT |
| Missing records | EXCEPT |
| One table minus another | EXCEPT |
| Bidirectional relationship | UNION ALL |
| Multiple independent filters | UNION / UNION ALL |
| Complement of a subset | EXCEPT |

---

# Last-Minute FAANG Notes

- Always ask whether duplicates matter.
- Default to `UNION ALL` unless distinct rows are explicitly required.
- `EXCEPT` is usually clearer than `NOT IN` for pure set-difference problems.
- `INTERSECT` communicates intent better than nested joins when finding common rows.
- Rewriting large `OR` predicates as separate queries combined with `UNION ALL` can improve index usage in some database engines.
- Set operations require the same number of columns with compatible data types on both sides.
- Be aware that MySQL versions before 8.0.31 do not support `INTERSECT` and `EXCEPT`; equivalent logic must be written using joins or `EXISTS`.

---

> **FAANG Interview Takeaway:** Strong SQL candidates recognize when a problem is fundamentally about **combining, intersecting, or subtracting sets** rather than defaulting to joins. Interviewers frequently assess this ability through follow-up variations that change the required set relationship without changing the underlying schema.
