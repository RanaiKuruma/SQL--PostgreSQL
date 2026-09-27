# PostgreSQL — Select and Sort Data

## 1.1 Select All Columns

```sql
SELECT * 
FROM dvd_rentals.language;
```

---

## 1.2 Select Specific Columns

When selecting specific columns, separate each column name with a comma.

> **Tip:** If you run into issues with SQL queries in the future, common causes include typos, missing commas, or extra trailing commas. On macOS, use **Command + F** to quickly find and check commas.

```sql
SELECT 
  language_id, 
  name 
FROM dvd_rentals.language;
```

---

## 1.3 Limit Output Rows

The `LIMIT` clause restricts the number of rows returned by the query.

```sql
SELECT * 
FROM dvd_rentals.actor 
LIMIT 10;
```

---

## 1.4 Sorting Query Results

`ORDER BY` is used to sort the output of a query.

- Sorting is **ascending (`ASC`)** by default.
- Multiple levels of sorting can be performed by specifying more than one column.
- In PostgreSQL, `NULL` values are placed **last by default** when sorting in ascending order. You can explicitly use `NULLS FIRST` when required.

### Sort in Ascending Order

```sql
SELECT country 
FROM dvd_rentals.country 
ORDER BY country 
LIMIT 5;
```

---

## 1.5 Sort by Numeric or Date Column

Columns containing numbers, dates, and timestamps can also be sorted.

For numeric values, ascending order goes from **lowest to highest**.

For date/time values, the ordering depends on whether `ASC` or `DESC` is specified.

### Sort Numeric Values

```sql
SELECT total_sales 
FROM dvd_rentals.sales_by_film_category
ORDER BY total_sales
LIMIT 5;
```

### Sort in Descending Order

Use `DESC` to sort from highest to lowest, or from latest to earliest for date/time values.

```sql
SELECT country 
FROM dvd_rentals.country 
ORDER BY country DESC 
LIMIT 5;
```

### Lowest Total Sales

```sql
SELECT 
    category, 
    total_sales
FROM dvd_rentals.sales_by_film_category
ORDER BY total_sales
LIMIT 1;
```

### Latest Payment Dates

```sql
SELECT 
  payment_date
FROM dvd_rentals.payment 
ORDER BY payment_date DESC 
LIMIT 5;
```

---

# Multiple-Column Sorting

`ORDER BY` can contain two or more columns.

The columns are evaluated **from left to right**. The first column determines the primary ordering, while the next column is used to sort rows that have the same value in the previous column.

## Create a Sample Table

The following example creates a temporary table to demonstrate multiple-column sorting.

```sql
DROP TABLE IF EXISTS sample_table; 

CREATE TEMP TABLE sample_table AS 
WITH raw_data (id, column_a, column_b) AS (
  VALUES 
  (1, 0, 'A'),
  (2, 0, 'B'),
  (3, 1, 'C'),
  (4, 1, 'D'),
  (5, 2, 'D'),
  (6, 3, 'D')
)
SELECT *
FROM raw_data;
```

> **Note:** The `SELECT * FROM raw_data` statement is included above to populate `sample_table`.

---

## Both Columns Ascending

Both `column_a` and `column_b` are sorted in ascending order.

```sql
SELECT * 
FROM sample_table 
ORDER BY column_a, column_b;
```

---

## First Column Descending, Second Ascending

Each column can have its own sort direction.

Here, `column_a` is sorted in descending order, while `column_b` remains ascending.

```sql
SELECT * 
FROM sample_table 
ORDER BY 
  column_a DESC, 
  column_b;
```

---

## Both Columns Descending

Both columns are sorted in descending order.

```sql
SELECT * 
FROM sample_table 
ORDER BY 
  column_a DESC, 
  column_b DESC;
```

---

## Different Column Order

The order in which columns appear in the `ORDER BY` clause changes the priority of the sorting.

### `column_b` First, Then `column_a`

```sql
SELECT * 
FROM sample_table 
ORDER BY 
  column_b DESC, 
  column_a;
```

### `column_b` First, Then `column_a DESC`

```sql
SELECT * 
FROM sample_table 
ORDER BY 
  column_b, 
  column_a DESC;
```

---

# Practical Examples

## Sort Rentals by Inventory and Rental Date

This query first sorts by `inventory_id` and then sorts rental dates in descending order within each inventory item.

```sql
SELECT 
  customer_id,
  inventory_id, 
  rental_date 
FROM dvd_rentals.rental 
ORDER BY 
  inventory_id,
  rental_date DESC 
LIMIT 5;
```

## Display the Five Highest-Selling Categories

Sorting `total_sales` in descending order places the highest sales values first.

```sql
SELECT 
  category, 
  total_sales
FROM dvd_rentals.sales_by_film_category
ORDER BY total_sales DESC
LIMIT 5;
```

---

# Key PostgreSQL Concepts

| Clause / Keyword | Purpose |
|---|---|
| `SELECT *` | Selects all columns |
| `SELECT column1, column2` | Selects specific columns |
| `FROM` | Specifies the table or view to query |
| `LIMIT` | Restricts the number of rows returned |
| `ORDER BY` | Sorts query results |
| `ASC` | Sorts in ascending order; this is the default |
| `DESC` | Sorts in descending order |
| `NULLS FIRST` | Places `NULL` values before non-`NULL` values |
| `NULLS LAST` | Places `NULL` values after non-`NULL` values |
| `,` in `ORDER BY` | Adds another sorting level |

## General Syntax

### Select and Limit

```sql
SELECT column1, column2
FROM table_name
LIMIT number_of_rows;
```

### Sort in Ascending Order

```sql
SELECT column1, column2
FROM table_name
ORDER BY column1 ASC;
```

### Sort in Descending Order

```sql
SELECT column1, column2
FROM table_name
ORDER BY column1 DESC;
```

### Multiple-Column Sorting

```sql
SELECT column1, column2
FROM table_name
ORDER BY 
  column1 ASC,
  column2 DESC;
```
