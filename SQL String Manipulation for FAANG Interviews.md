# SQL String Manipulation for FAANG Interviews
## LeetCode SQL Study Guide

> A practical interview guide focused on SQL string manipulation patterns frequently tested in LeetCode and FAANG SQL interviews.

---

# Table of Contents

- [How to Use This Guide](#how-to-use-this-guide)
- [Problem 1 — Patients With a Condition (Easy)](#problem-1--patients-with-a-condition-easy)
- [Problem 2 — Fix Names in a Table (Easy)](#problem-2--fix-names-in-a-table-easy)
- [Problem 3 — Find Users With Valid Emails (Medium)](#problem-3--find-users-with-valid-emails-medium)
- [Problem 4 — Rearrange Products Table (Medium)](#problem-4--rearrange-products-table-medium)
- [Problem 5 — Group Sold Products By Date (Hard Pattern)](#problem-5--group-sold-products-by-date-hard-pattern)
- [Problem 6 — User Mention Analytics (Interview Variant)](#problem-6--user-mention-analytics-interview-variant)
- [Interview Pattern Summary](#interview-pattern-summary)
- [Common Mistakes](#common-mistakes)
- [FAANG Interview Notes](#faang-interview-notes)
- [Revision Checklist](#revision-checklist)

---

# How to Use This Guide

Unlike theory-first tutorials, every concept is introduced **inside a real interview problem**.

Throughout the guide you'll repeatedly encounter these SQL string functions naturally:

| Function | Purpose |
|----------|----------|
| CONCAT() | Join strings |
| CONCAT_WS() | Join with separator |
| LOWER() | Convert to lowercase |
| UPPER() | Convert to uppercase |
| SUBSTRING() | Extract characters |
| LEFT() | Left portion |
| RIGHT() | Right portion |
| LENGTH() | Count characters |
| TRIM() | Remove spaces |
| LTRIM() | Remove leading spaces |
| RTRIM() | Remove trailing spaces |
| REPLACE() | Replace characters |
| LIKE | Pattern matching |
| REGEXP | Regular expressions |
| LOCATE()/INSTR() | Find substring |
| POSITION() | Position of substring |

Each problem teaches when these functions should—and should **not**—be used.

---

# Problem 1 — Patients With a Condition (Easy)

**LeetCode Difficulty:** Easy

**Pattern**

- LIKE
- Wildcard Matching
- Word Boundary Detection

**Frequently Asked By**

| Company | Frequency |
|----------|-----------|
| Google | ★★★★☆ |
| Amazon | ★★★★★ |
| Meta | ★★★★☆ |
| Apple | ★★★☆☆ |

---

## Problem Statement

Table:

```text
Patients
+------------+--------------+
| patient_id | conditions   |
+------------+--------------+
```

The **conditions** column contains diseases separated by spaces.

Return every patient whose disease list contains **DIAB1**.

Example:

```text
DIAB1
ASTHMA DIAB1
DIAB100
DIAB1 FLU
FLU DIAB1
```

Expected:

```text
DIAB1
ASTHMA DIAB1
DIAB1 FLU
FLU DIAB1
```

Notice that **DIAB100** should NOT match.

---

## Interview Insight

Many candidates write

```sql
LIKE '%DIAB1%'
```

This incorrectly returns

```text
DIAB100
```

because it simply searches for characters, **not words**.

---

## Optimal Solution

```sql
SELECT *
FROM Patients
WHERE conditions = 'DIAB1'
   OR conditions LIKE 'DIAB1 %'
   OR conditions LIKE '% DIAB1'
   OR conditions LIKE '% DIAB1 %';
```

---

## Why This Works

Each condition checks one possible position.

```text
Beginning

DIAB1 Flu

^^^^^^^^

----------------------------

Middle

Flu DIAB1 Cold

    ^^^^^

----------------------------

End

Flu DIAB1

    ^^^^^

----------------------------

Only word

DIAB1
```

Together they cover every legal occurrence while preventing partial matches.

---

## Query Execution

```
Patients
     │
     ▼

Read conditions

     │

Check position of DIAB1

     │

Exact?
Beginning?
Middle?
Ending?

     │

Return matching rows
```

---

## Alternative (Using REGEXP)

Many databases support:

```sql
SELECT *
FROM Patients
WHERE conditions REGEXP '(^| )DIAB1( |$)';
```

This checks

```
Start OR Space
        DIAB1
Space OR End
```

Much cleaner.

---

## Technique Learned

Pattern matching with boundaries.

Instead of

```text
contains substring
```

think

```text
contains complete word
```

This distinction appears repeatedly in interviews.

---

## Complexity

| Metric | Complexity |
|---------|------------|
| Time | O(n × L) |
| Space | O(1) |

where

- n = rows
- L = string length

---

## Common Mistakes

### Mistake 1

```sql
LIKE '%DIAB1%'
```

Matches

```
DIAB10
DIAB11
DIAB100
```

Wrong.

---

### Mistake 2

Ignoring leading spaces

Data sometimes contains

```
" DIAB1"
```

Interviewers may intentionally include dirty data.

Safer:

```sql
TRIM(conditions)
```

before comparison.

---

### Mistake 3

Assuming one disease only.

Always inspect delimiter.

---

## LLM-Proof Variation

Instead of spaces, diseases are separated by commas.

Example

```
Cold,Diab1,Flu
```

Now

```sql
LIKE '%DIAB1%'
```

again becomes incorrect.

Correct approach:

```sql
REGEXP '(^|,)DIAB1(,|$)'
```

or use delimiter-aware parsing.

---

## Pattern Library

```
Need exact token?

↓

Delimiter?

↓

Space
Comma
Pipe
Semicolon

↓

Check boundaries

↓

Never search blindly with
LIKE '%word%'
```

---

# Problem 2 — Fix Names in a Table (Easy)

**LeetCode Difficulty:** Easy

**Pattern**

- LOWER
- UPPER
- CONCAT
- SUBSTRING

**Frequently Asked By**

| Company | Frequency |
|----------|-----------|
| Google | ★★★★★ |
| Meta | ★★★★☆ |
| Amazon | ★★★★☆ |
| Netflix | ★★★☆☆ |

---

## Problem Statement

Names are stored inconsistently.

Example

```text
ALICE
aLiCe
bOB
CHARLIE
```

Return

```text
Alice
Alice
Bob
Charlie
```

---

## Interview Goal

Can you normalize text without procedural code?

---

## Optimal Solution

```sql
SELECT
    user_id,
    CONCAT(
        UPPER(LEFT(name,1)),
        LOWER(SUBSTRING(name,2))
    ) AS name
FROM Users
ORDER BY user_id;
```

---

## Execution Walkthrough

Input

```
aLiCe
```

Step 1

```
LEFT(name,1)

↓

a
```

Step 2

```
UPPER(a)

↓

A
```

Step 3

```
SUBSTRING(name,2)

↓

LiCe
```

Step 4

```
LOWER()

↓

lice
```

Step 5

```
CONCAT

↓

Alice
```

---

## Visualization

```
Original

a L i C e

│

├── First letter

│      │

│      ▼

│      A

│

└── Remaining letters

       │

       ▼

     lice

↓

Alice
```

---

## Technique Learned

This problem combines four operations.

```
Split

↓

Transform

↓

Merge
```

That workflow appears frequently in interview SQL.

---

## Why CONCAT?

Instead of

```sql
UPPER(name)
```

which returns

```
ALICE
```

we separately transform

```
First character

+

Remaining characters
```

---

## Alternative Using SUBSTRING

```sql
SELECT
CONCAT(
UPPER(SUBSTRING(name,1,1)),
LOWER(SUBSTRING(name,2))
)
FROM Users;
```

Works identically.

---

## Complexity

| Metric | Complexity |
|---------|------------|
| Time | O(n × L) |
| Space | O(1) |

---

## Common Mistakes

### Forgetting LOWER()

Produces

```
ALiCe
```

instead of

```
Alice
```

---

### Forgetting CONCAT()

Returning

```sql
UPPER(...)
LOWER(...)
```

as separate columns.

---

### Using LEFT()

Some SQL dialects don't support

```sql
LEFT()
```

Use

```sql
SUBSTRING(name,1,1)
```

instead.

---

## LLM-Proof Interview Variant

Normalize names containing spaces.

Input

```
john smith
```

Expected

```
John Smith
```

Simple

```sql
UPPER(first letter)
```

fails.

Need to capitalize every word.

Interviewers may expect recursive SQL, REGEXP functions, or string splitting depending on the database.

---

## Pattern Library

```
Need formatting?

↓

Split string

↓

Apply functions independently

↓

Join pieces

↓

Return formatted text
```

---

## Interview Takeaways from Problems 1 & 2

| Pattern | Functions |
|----------|-----------|
| Word boundary search | LIKE, REGEXP |
| Capitalization | LOWER, UPPER |
| Split string | SUBSTRING, LEFT |
| Merge string | CONCAT |
| Cleaning | TRIM |
| Exact matching | LIKE with delimiters |

---

**End of Part 1**

The next section covers two medium-level interview problems focused on **email validation using `LIKE`/`REGEXP`** and **advanced string aggregation/transformation patterns**, which are common in Google, Meta, and Amazon SQL interviews.



# Problem 3 — Find Users With Valid Emails (Medium)

**LeetCode Difficulty:** Medium

**Pattern**

- REGEXP
- LIKE
- Character Validation
- Pattern Matching

**Frequently Asked By**

| Company | Frequency |
|----------|-----------|
| Google | ★★★★★ |
| Meta | ★★★★★ |
| Amazon | ★★★★☆ |
| Apple | ★★★★☆ |
| Netflix | ★★★☆☆ |

---

## Problem Statement

Table

```text
Users
+---------+---------------------------+
| user_id | mail                      |
+---------+---------------------------+
```

A valid email must satisfy:

- Starts with a letter.
- May contain letters, digits, underscores (`_`), periods (`.`), and hyphens (`-`) before the `@`.
- Domain must be exactly:

```text
@leetcode.com
```

Return only users with valid emails.

Example

| mail | Valid? |
|------|--------|
| alice@leetcode.com | ✅ |
| Alice_99@leetcode.com | ✅ |
| 1alice@leetcode.com | ❌ |
| alice@gmail.com | ❌ |
| alice@leetcode.co | ❌ |

---

## Interview Goal

This problem tests whether you understand:

- Exact pattern matching
- Character classes
- Anchors
- Validation using SQL instead of application code

Many companies use it to assess familiarity with `REGEXP`.

---

## Optimal Solution

```sql
SELECT *
FROM Users
WHERE mail REGEXP '^[A-Za-z][A-Za-z0-9_.-]*@leetcode\\.com$';
```

---

## Breaking Down the Regular Expression

```
^

Start of string

[A-Za-z]

First character must be a letter

[A-Za-z0-9_.-]*

Remaining username characters

@

Literal @

leetcode

Fixed domain

\.com

Literal ".com"

$

End of string
```

---

## Visual Breakdown

Input

```
Alice_99@leetcode.com
```

```
A

↓

Letter ✓

lice_99

↓

Allowed characters ✓

@

↓

Required ✓

leetcode.com

↓

Correct domain ✓

↓

Valid
```

---

## Query Execution

```
Users

 │

 ▼

Read Email

 │

 ▼

Apply REGEXP

 │

 ├───────────────┐

 │               │

Match         No Match

 │               │

 ▼               ▼

Return       Ignore
```

---

## Why Not LIKE?

Some candidates attempt

```sql
WHERE mail LIKE '%@leetcode.com'
```

Problem:

This incorrectly accepts

```
123@leetcode.com

@leetcode.com

!!!!@leetcode.com

```

LIKE checks only the suffix—not the username format.

---

## Technique Learned

Use `REGEXP` whenever validation involves:

- allowed characters
- forbidden characters
- exact format
- minimum/maximum occurrences

If the interviewer says:

> "Validate..."

think:

```
REGEXP
```

before

```
LIKE
```

---

## Complexity

| Metric | Complexity |
|---------|------------|
| Time | O(n × L) |
| Space | O(1) |

where

- n = rows
- L = email length

---

## Common Mistakes

### Missing Start Anchor

Wrong

```sql
REGEXP '[A-Za-z]'
```

Matches anywhere.

---

### Missing End Anchor

Wrong

```sql
@leetcode.com
```

Matches

```
abc@leetcode.com123
```

---

### Forgetting to Escape '.'

Wrong

```sql
.com
```

`.` means

```
Any Character
```

Correct

```sql
\\.com
```

---

### Using LIKE

LIKE cannot verify:

- first character
- character classes
- multiple rules simultaneously

---

## LLM-Proof Interview Variant

Company emails now follow:

```
firstname.lastname@company.ai
```

Requirements

- first name lowercase
- last name lowercase
- exactly one period
- no digits
- domain fixed

Can you write one regular expression?

Expected pattern

```
^[a-z]+\\.[a-z]+@company\\.ai$
```

Interviewers often extend the original problem this way.

---

## Pattern Library

```
Need validation?

↓

Exact format?

↓

Allowed characters?

↓

Use REGEXP

↓

Anchor beginning

↓

Anchor ending

↓

Return only complete matches
```

---

# Problem 4 — Rearrange Products Table (Medium)

**LeetCode Difficulty:** Medium

**Pattern**

- String Transformation
- UNPIVOT Pattern
- CONCAT
- NULL Filtering

**Frequently Asked By**

| Company | Frequency |
|----------|-----------|
| Google | ★★★★☆ |
| Amazon | ★★★★★ |
| Meta | ★★★★☆ |
| Microsoft | ★★★★☆ |

---

## Problem Statement

Table

```text
Products

+------------+-------+-------+-------+
| product_id | store1| store2| store3|
+------------+-------+-------+-------+
```

Example

| product_id | store1 | store2 | store3 |
|------------|--------|--------|--------|
| 0 | 95 | 100 | NULL |
| 1 | NULL | 50 | 80 |

Return

```text
product_id
store
price
```

Expected

| product_id | store | price |
|------------|-------|------|
|0|store1|95|
|0|store2|100|
|1|store2|50|
|1|store3|80|

---

## Interview Goal

Convert

```
Wide Table

↓

Long Table
```

This transformation appears frequently in analytics interviews.

---

## Optimal Solution

```sql
SELECT product_id,
       'store1' AS store,
       store1 AS price
FROM Products
WHERE store1 IS NOT NULL

UNION ALL

SELECT product_id,
       'store2',
       store2
FROM Products
WHERE store2 IS NOT NULL

UNION ALL

SELECT product_id,
       'store3',
       store3
FROM Products
WHERE store3 IS NOT NULL;
```

---

## Visualization

Original

```
ID

Store1

Store2

Store3

↓

0

95

100

NULL

↓

1

NULL

50

80
```

After transformation

```
0 store1 95

0 store2 100

1 store2 50

1 store3 80
```

---

## Execution Flow

```
Products

 │

 ├──────────────┐

 │              │

Store1      Store2

 │              │

Store3

 │

 ▼

UNION ALL

 │

 ▼

Final Table
```

---

## Why UNION ALL?

Each SELECT creates one store-specific view.

```
Store1 Rows

+

Store2 Rows

+

Store3 Rows
```

becomes

```
Single Result Set
```

---

## Where Is String Manipulation?

Notice

```sql
'store1'
```

```sql
'store2'
```

```sql
'store3'
```

These are literal strings injected into the output.

Interviewers often extend this by asking for

```sql
CONCAT('Store-', store_number)
```

or

```sql
UPPER(store_name)
```

after transformation.

---

## Complexity

If

- n rows
- k stores

Time

```
O(n × k)
```

Space

```
O(1)
```

excluding output.

---

## Common Mistakes

### Using UNION

Wrong

```sql
UNION
```

Duplicates are removed.

The problem never asks for duplicate elimination.

Use

```sql
UNION ALL
```

---

### Forgetting NULL Filter

Produces

```
store3 NULL
```

which should not appear.

---

### Wrong Literal

Wrong

```sql
store1
```

Correct

```sql
'store1'
```

Without quotes SQL interprets it as a column.

---

## LLM-Proof Interview Variant

Instead of three stores, there are

```
store1

store2

...

store100
```

Hard-coding 100 UNION statements is impossible.

Expected discussion:

- Dynamic SQL
- UNPIVOT (SQL Server)
- CROSS APPLY
- JSON_TABLE (MySQL 8)
- UNNEST (BigQuery)
- LATERAL VIEW (Hive)

Interviewers are testing scalability rather than syntax.

---

## Pattern Library

```
Wide Table

↓

One column becomes many rows

↓

UNION ALL

↓

Filter NULL

↓

Optional string formatting

↓

Return normalized table
```

---

## Interview Takeaways from Problems 3 & 4

| Pattern | Functions / Features |
|---------|-----------------------|
| Email validation | `REGEXP` |
| Character classes | `[A-Za-z0-9]` |
| Anchors | `^`, `$` |
| Literal matching | `\\.` |
| Fixed domain validation | `REGEXP` |
| Wide → Long transformation | `UNION ALL` |
| String literals | `'store1'` |
| Optional formatting | `CONCAT`, `UPPER`, `LOWER` |

---

**End of Part 2**

The next section covers:

- **Problem 5 — Group Sold Products By Date** (advanced `GROUP_CONCAT` / `STRING_AGG` interview pattern)
- **Problem 6 — User Mention Analytics** (hard interview-style string parsing with `SUBSTRING`, `INSTR`, `REPLACE`, and aggregation)
- FAANG interview notes
- Complete revision checklist

# Problem 5 — Group Sold Products By Date (Hard Pattern)

> **LeetCode:** Group Sold Products By The Date (Medium)  
> **Interview Pattern:** String Aggregation

**Difficulty:** Medium (Interview Hard Pattern)

**Pattern**

- GROUP_CONCAT / STRING_AGG
- DISTINCT
- ORDER BY
- Aggregation + String Manipulation

**Frequently Asked By**

| Company | Frequency |
|----------|-----------|
| Google | ★★★★★ |
| Meta | ★★★★★ |
| Amazon | ★★★★★ |
| Apple | ★★★★☆ |
| Netflix | ★★★★☆ |

---

## Problem Statement

Table

```text
Activities

+------------+--------------+
| sell_date  | product       |
+------------+--------------+
```

Example

| sell_date | product |
|----------|---------|
|2020-05-30|Headphone|
|2020-05-30|Basketball|
|2020-06-01|T-Shirt|
|2020-06-01|Headphone|
|2020-06-01|T-Shirt|

Return

| sell_date | num_sold | products |
|-----------|----------|----------|
|2020-05-30|2|Basketball,Headphone|
|2020-06-01|2|Headphone,T-Shirt|

Products must

- be unique
- appear alphabetically
- be comma separated

---

## Interview Goal

This problem combines

```
GROUP BY

+

String Aggregation

+

Sorting

+

Duplicate Removal
```

Instead of producing numbers,

SQL now produces **one formatted string**.

---

## MySQL Solution

```sql
SELECT
    sell_date,
    COUNT(DISTINCT product) AS num_sold,
    GROUP_CONCAT(
        DISTINCT product
        ORDER BY product
        SEPARATOR ','
    ) AS products
FROM Activities
GROUP BY sell_date
ORDER BY sell_date;
```

---

## PostgreSQL Version

```sql
SELECT
    sell_date,
    COUNT(DISTINCT product),
    STRING_AGG(
        DISTINCT product,
        ',' ORDER BY product
    ) AS products
FROM Activities
GROUP BY sell_date;
```

---

## SQL Server Version

```sql
SELECT
    sell_date,
    COUNT(DISTINCT product),
    STRING_AGG(product, ',')
FROM
(
    SELECT DISTINCT sell_date, product
    FROM Activities
) t
GROUP BY sell_date;
```

---

## Step-by-Step Execution

Original

```
2020-06-01

Headphone

T-Shirt

T-Shirt
```

---

### Step 1

Remove duplicates

```
Headphone

T-Shirt
```

---

### Step 2

Sort alphabetically

```
Headphone

T-Shirt
```

---

### Step 3

Join into one string

```
Headphone,T-Shirt
```

---

### Step 4

Count products

```
2
```

---

### Final Row

```
2020-06-01

2

Headphone,T-Shirt
```

---

## Visual Flow

```
Activities

        │

        ▼

GROUP BY sell_date

        │

        ▼

DISTINCT product

        │

        ▼

Alphabetical Sort

        │

        ▼

GROUP_CONCAT()

        │

        ▼

Final Output
```

---

## Technique Learned

Most SQL interview questions aggregate numbers.

This one aggregates

```
Rows

↓

String
```

That's a completely different interview pattern.

---

## Complexity

Suppose

- n rows
- k products per day

Sorting each group

```
O(k log k)
```

Overall

```
O(n log k)
```

Space

```
O(k)
```

---

## Common Mistakes

### Forgetting DISTINCT

Wrong output

```
Headphone,T-Shirt,T-Shirt
```

---

### Forgetting ORDER BY

Output becomes

```
T-Shirt,Headphone
```

which fails the judge.

---

### Wrong Separator

Default separator differs across SQL dialects.

Always specify

```sql
SEPARATOR ','
```

---

### Counting Without DISTINCT

Wrong

```sql
COUNT(product)
```

Returns

```
3
```

Correct answer

```
2
```

---

## LLM-Proof Interview Variant

Instead of

```
Comma-separated
```

return

```
Headphone | Basketball | T-Shirt
```

or

```
JSON Array
```

Expected discussion

- `GROUP_CONCAT`
- `STRING_AGG`
- `JSON_ARRAYAGG`
- Custom separators
- Ordered aggregation

Interviewers are testing adaptability—not memorization.

---

## Pattern Library

```
Need one row

↓

Need one string

↓

GROUP BY

↓

DISTINCT

↓

Sort

↓

GROUP_CONCAT /
STRING_AGG
```

---

# Problem 6 — User Mention Analytics (Interview Variant)

> Not an official LeetCode problem, but a very common Google/Meta interview pattern inspired by production analytics.

**Difficulty:** Hard

**Pattern**

- SUBSTRING
- LOCATE / INSTR
- REPLACE
- LENGTH
- String Parsing
- Aggregation

**Frequently Asked By**

| Company | Frequency |
|----------|-----------|
| Google | ★★★★★ |
| Meta | ★★★★★ |
| Amazon | ★★★★☆ |
| Apple | ★★★★☆ |

---

## Problem Statement

Table

```text
Messages

+------------+-------------------------------+
| id         | message                       |
+------------+-------------------------------+
```

Messages contain mentions.

Example

```
Hello @john

Hi @alice @bob

Morning @john @alice

No mentions
```

Find

```
Each username

Number of mentions
```

Expected

| username | mentions |
|----------|----------|
|alice|2|
|bob|1|
|john|2|

---

## Interview Goal

Can you extract structured information from free-form text?

Real production logs frequently look like this.

---

## Basic Extraction

Extract everything after '@'

```sql
SUBSTRING(
    message,
    LOCATE('@', message) + 1
)
```

Produces

```
john

alice @bob

john @alice
```

Still not enough.

We now need to isolate individual usernames.

---

## Finding '@'

```
Hello @john

      ^

LOCATE('@')
```

returns

```
7
```

Then

```
SUBSTRING()

↓

john
```

---

## Counting Mentions

A classic trick

```sql
LENGTH(message)
-
LENGTH(REPLACE(message,'@',''))
```

Example

```
@john @alice
```

Length

```
13
```

Remove

```
john alice
```

Length

```
11
```

Difference

```
2
```

Exactly two mentions.

---

## Visualization

```
Original

@john @alice @bob

 │

 ▼

REPLACE(@,'')

 │

 ▼

john alice bob

 │

 ▼

Length Difference

 │

 ▼

3 Mentions
```

---

## Why This Matters

This pattern appears everywhere

- hashtag counting
- URL counting
- emoji analytics
- log analysis
- CSV parsing
- delimiter counting

Interviewers often expect this trick.

---

## Full Production Solution

In real systems you'd typically combine:

- Recursive CTE
- Numbers table
- REGEXP_SUBSTR()
- STRING_SPLIT()
- JSON_TABLE()
- SPLIT_PART()

depending on the SQL dialect.

The interviewer usually evaluates your **approach**, not a vendor-specific implementation.

---

## Complexity

Suppose

- n rows
- L characters

Time

```
O(n × L)
```

Space

```
O(1)
```

excluding output.

---

## Common Mistakes

### Assuming One Mention

Wrong

```
Hello @john @alice
```

Only extracting

```
john
```

---

### Using REPLACE Incorrectly

Removing

```
john
```

instead of

```
@
```

changes the count.

---

### Ignoring Consecutive Spaces

```
@john    @alice
```

Tokenization must still work.

---

### Ignoring Punctuation

```
@john,
```

Should the comma belong to the username?

Clarify assumptions with the interviewer.

---

## LLM-Proof Interview Variant

Messages may contain

```
@john

#python

https://abc.com

@alice

#sql
```

Return

- mentions
- hashtags
- URLs

in separate columns.

Expected discussion

- REGEXP
- Recursive parsing
- Tokenization
- Delimiter-based extraction

This tests pattern recognition rather than memorized SQL.

---

## Pattern Library

```
Need to parse text?

↓

Locate delimiter

↓

Extract substring

↓

Remove unwanted symbols

↓

Aggregate
```

---

# Interview Pattern Summary

| Pattern | Typical Functions | Common Interview Questions |
|---------|-------------------|----------------------------|
| Word Boundary Search | `LIKE`, `REGEXP` | Disease codes, keywords |
| Name Formatting | `UPPER`, `LOWER`, `SUBSTRING`, `CONCAT` | User profiles |
| Email Validation | `REGEXP` | Data cleaning |
| String Aggregation | `GROUP_CONCAT`, `STRING_AGG` | Reports, analytics |
| Counting Characters | `LENGTH`, `REPLACE` | Hashtags, mentions |
| Substring Extraction | `SUBSTRING`, `LEFT`, `RIGHT`, `LOCATE`, `INSTR` | Log parsing |
| String Cleaning | `TRIM`, `LTRIM`, `RTRIM`, `REPLACE` | ETL pipelines |
| Pattern Matching | `LIKE`, `REGEXP` | Search, filtering |

---

# Common Mistakes

| Mistake | Better Approach |
|----------|-----------------|
| `LIKE '%abc%'` for whole words | Use boundaries or `REGEXP` |
| Forgetting `DISTINCT` | Remove duplicates before aggregation |
| Forgetting `ORDER BY` in `GROUP_CONCAT` | Produce deterministic output |
| Ignoring spaces | Apply `TRIM()` where appropriate |
| Using `UNION` instead of `UNION ALL` | Preserve all rows unless deduplication is required |
| Ignoring SQL dialect differences | Know MySQL, PostgreSQL, SQL Server equivalents |

---

# FAANG Interview Notes

## Google

Focus areas:

- Regular expressions
- Log parsing
- Text normalization
- Analytics queries

---

## Meta

Frequently combines:

- Window functions
- String manipulation
- Aggregation

Expect multi-step SQL questions.

---

## Amazon

Often tests

- Data cleaning
- Customer email validation
- Product string aggregation
- ETL-style transformations

---

## Apple

Common themes:

- Formatting
- Parsing identifiers
- Production-quality SQL

---

## Netflix

Questions frequently involve

- Viewing history
- Search keywords
- Recommendation analytics
- String aggregation for reporting

---

# Revision Checklist

Before an interview, ensure you can solve problems involving:

- ✅ `CONCAT()` / `CONCAT_WS()`
- ✅ `LOWER()` / `UPPER()`
- ✅ `SUBSTRING()`
- ✅ `LEFT()` / `RIGHT()`
- ✅ `TRIM()`
- ✅ `REPLACE()`
- ✅ `LENGTH()`
- ✅ `LOCATE()` / `INSTR()`
- ✅ `LIKE`
- ✅ `REGEXP`
- ✅ `GROUP_CONCAT()` / `STRING_AGG()`
- ✅ `DISTINCT` inside string aggregation
- ✅ Delimiter-aware parsing
- ✅ Pattern matching with anchors
- ✅ Text normalization
- ✅ SQL dialect differences

---

# Final Takeaways

Across FAANG SQL interviews, string manipulation questions generally fall into six recurring categories:

1. **Pattern Matching** — `LIKE`, `REGEXP`, word boundaries.
2. **String Formatting** — capitalization, trimming, concatenation.
3. **Validation** — emails, usernames, product codes, identifiers.
4. **Parsing** — extracting tokens, delimiters, mentions, hashtags.
5. **Aggregation** — converting multiple rows into formatted strings.
6. **Cleaning & Transformation** — preparing raw text for analytics.

Mastering these patterns provides coverage for the majority of SQL string-manipulation interview questions encountered on LeetCode and in production-oriented technical interviews.
