# 2. Record Counts and Distinct Values

## 2.0 Inspecting the Raw Data

The first step when exploring a dataset is to look at a small sample of the raw data.

```sql
SELECT *
FROM dvd_rentals.film_list
LIMIT 5;
```

---

## 2.1 Record Counts

The first thing to know about a dataset is how many rows it contains.

```sql
SELECT 
  COUNT(*) AS row_count
  -- COUNT(*)
FROM dvd_rentals.film_list;
```

### Column Aliases

Column aliases make SQL code easier to read and maintain.

In the example above, `row_count` is an alias for the result returned by `COUNT(*)`.

---

## 2.2 DISTINCT for Unique Values

The `DISTINCT` keyword is used to obtain unique values from a column by removing duplicate values.

```sql
SELECT DISTINCT rating
FROM dvd_rentals.film_list;
```

For example, if the `rating` column contains many repeated values such as `PG`, `PG-13`, and `R`, `DISTINCT` returns each rating only once.

---

## 2.3 Count of Unique Values

The `COUNT` function can be combined with `DISTINCT` to find the number of unique values in a specific column.

```sql
SELECT 
  COUNT(DISTINCT category) AS unique_category_count
FROM dvd_rentals.film_list;
```

Here:

- `DISTINCT category` identifies the unique category values.
- `COUNT(...)` counts how many unique values were found.
- `AS unique_category_count` gives the result a readable column name.

---

## 2.4 GROUP BY Counts

The `GROUP BY` clause can be used with the `COUNT` aggregate function to generate basic frequency value counts.

`GROUP BY` can be imagined as dividing the dataset into different groups based on the values of selected columns.

### Example

First, look at a simplified version of the table:

```sql
SELECT 
  fid, 
  title, 
  category, 
  rating, 
  price 
FROM dvd_rentals.film_list 
LIMIT 10;
```

### GROUP BY Query Using a Common Table Expression

A Common Table Expression (CTE) can be used to create a temporary result set that is then queried.

```sql
WITH example_table AS (
  SELECT 
    fid, 
    title, 
    category, 
    rating,
    price 
  FROM dvd_rentals.film_list
  LIMIT 10
)

SELECT 
  rating, 
  COUNT(*) AS record_count 
FROM example_table 
GROUP BY rating 
ORDER BY rating DESC;
```

---

## 2.4.1 Dividing Rows

The rows are divided according to the distinct values of the `rating` column.

Conceptually:

```text
All rows
   |
   +-- Rating = PG
   |
   +-- Rating = PG-13
   |
   +-- Rating = R
   |
   +-- Rating = G
   |
   +-- ...
```

Each distinct value becomes a separate group.

---

## 2.4.2 Applying the Aggregate COUNT Function

Once the `GROUP BY` clause splits the dataset into groups, an aggregate function such as `COUNT(*)` is applied within each group.

The `COUNT(*)` function condenses each group into a single result.

### Important

> A `GROUP BY` query with aggregate functions returns only **one row for each group**.

For example, if there are five distinct ratings, the result will contain five rows.

---

## 2.4.3 Outputs Are Condensed and Combined

Once the calculations are completed for each separate group, the condensed single row from each group is combined to form the final output of the SQL statement containing the `GROUP BY` clause.

Conceptually:

```text
Original dataset
       |
       v
Divide rows into groups
       |
       v
Apply COUNT(*) to each group
       |
       v
One result row per group
       |
       v
Combine the grouped results
       |
       v
Final output
```

---

## 2.4.4 Single-Column Value Counts

The following query counts how frequently each rating occurs.

```sql
SELECT 
  rating, 
  COUNT(*) AS frequency
FROM dvd_rentals.film_list
GROUP BY rating;
```

The result contains one row for each distinct `rating` value.

For example:

| rating | frequency |
|---|---:|
| PG | ... |
| PG-13 | ... |
| R | ... |

The exact frequencies depend on the data in `dvd_rentals.film_list`.

---

## 2.4.5 Adding a Percentage Column

A percentage column can be calculated to show what proportion of the total records belongs to each group.

```sql
SELECT 
  rating, 
  COUNT(*) AS frequency,
  COUNT(*)::NUMERIC / SUM(COUNT(*)) OVER () AS percentage
FROM dvd_rentals.film_list
GROUP BY rating
ORDER BY frequency DESC;
```

### How the Percentage Calculation Works

The calculation:

```sql
COUNT(*)::NUMERIC / SUM(COUNT(*)) OVER ()
```

can be understood as:

```text
frequency of the current group
--------------------------------
total frequency of all groups
```

`COUNT(*)::NUMERIC` converts the count to a numeric value so that the division produces a decimal result.

`SUM(COUNT(*)) OVER ()` calculates the total count across all grouped results.

---

### Rounding Off the Percentage

The percentage can be multiplied by `100` and rounded to two decimal places.

```sql
SELECT 
  rating, 
  COUNT(*) AS frequency,
  ROUND(
    100 * COUNT(*)::NUMERIC / SUM(COUNT(*)) OVER (), 
    2
  ) AS percentage
FROM dvd_rentals.film_list
GROUP BY rating
ORDER BY frequency DESC;
```

The resulting `percentage` column represents the percentage of all records belonging to each rating group.

For example:

| rating | frequency | percentage |
|---|---:|---:|
| PG | ... | ... |
| PG-13 | ... | ... |
| R | ... | ... |

---

## 2.5 Counts for Multiple Column Combinations

When `GROUP BY` is used with two or more columns, the `COUNT` function aggregates records based on the **unique combination of values** in those columns.

For example:

```sql
SELECT 
  rating, 
  category, 
  COUNT(*) AS frequency
FROM dvd_rentals.film_list
GROUP BY 
  rating, 
  category
ORDER BY frequency DESC 
LIMIT 5;
```

Here, rows are grouped by the combination of:

```text
rating + category
```

This is different from grouping by `rating` alone.

For example:

```text
rating = PG
category = Action
```

is treated as a different group from:

```text
rating = PG
category = Comedy
```

Therefore, `COUNT(*)` counts the records belonging to each unique `rating` + `category` combination.

---

## 2.5.1 Using Positional Numbers in GROUP BY

PostgreSQL also allows positional numbers to be used instead of column names in the `GROUP BY` clause.

```sql
SELECT 
  rating, 
  category, 
  COUNT(*) AS frequency
FROM dvd_rentals.film_list
GROUP BY 1, 2
ORDER BY frequency DESC 
LIMIT 5;
```

In this query:

```text
1 → rating
2 → category
```

because `rating` is the first selected column and `category` is the second selected column.

Therefore:

```sql
GROUP BY 1, 2
```

is equivalent to:

```sql
GROUP BY rating, category
```

### Note

Using column names can sometimes be easier to understand and maintain, especially when working with longer or more complex queries. Positional references are useful to recognise because they are commonly encountered in SQL code.

---

# Key Concepts

| Concept | Purpose |
|---|---|
| `COUNT(*)` | Counts rows/records |
| `DISTINCT` | Removes duplicate values |
| `COUNT(DISTINCT column)` | Counts unique values in a column |
| `GROUP BY` | Divides rows into groups based on selected column values |
| `COUNT(*)` with `GROUP BY` | Counts records within each group |
| `GROUP BY` on multiple columns | Counts unique combinations of column values |
| `SUM(COUNT(*)) OVER ()` | Calculates the total across grouped counts |
| `::NUMERIC` | Converts a value to a numeric type |
| `ROUND(..., 2)` | Rounds a numeric result to two decimal places |
| `GROUP BY 1, 2` | Groups by the first and second columns in the `SELECT` list |
