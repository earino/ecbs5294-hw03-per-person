# SQL reference — one page

Everything a lab in this course asks you to write, in the order it was taught: each line is on a slide of the block
named beside it, or in the lab named beside it, and a block's section is for that block on (a sheet can carry a block
you have not had yet). The examples read the files of the project they were taught in (Lab 1's `owid_co2.csv`, Lab
2's `online_retail.parquet`, Lab 3's Olist files) and the views those labs make (`countries`, `sales_2010`); to run
one in another project, change the file, the key columns and the column you count. Keep it open in a second tab. Paths are always `'data/raw/<file>'`
from the project folder, and every query is `con.sql("""...""").df()`; a single number is
`con.sql("...").fetchone()[0]`. The full reference, with a task index in plain English ("I want the first
character of a code…"), joins, JSON, pipelines and checks, is on the course site:
https://earino.github.io/ecbs5294-2026/site/reference.html

## Session 3: what you want to do, and where it is

Every shape here is from the Session 3 lectures' files; the full examples are in the sections named.

| You want to | The shape | Section |
|---|---|---|
| look at a few rows, and say what one row is | `SELECT * FROM bronze_long LIMIT 5` | Block 1 |
| count the rows of a table | `SELECT COUNT(*) FROM gdp` | Block 5 |
| know how many records the document says it holds | `SELECT DISTINCT total FROM gdp_metadata` | Block 5 |
| keep places, not groups such as World | `JOIN places AS pl ON pl.iso3 = g.iso3` | Block 5 |
| count the rows that hold a number | `COUNT(gdp)`: it skips `NULL`; `COUNT(*)` does not | Block 6 |
| count the cells of bronze that hold a number | `WHERE raw_value NOT LIKE ':%'` (bronze is text) | Block 6 |
| make text a number | `CAST(… AS DOUBLE)`; `TRY_CAST` makes `NULL` of what it cannot read | Block 6 |
| join two silvers on two columns | `ON p.country = g.country AND p.year = g.year` | Block 6 |
## Block 1 — one table, one row at a time

**The four inspection recipes** (on Lab 1's file; in another project change the file, the key columns in the third, and the column counted in the fourth; the grammar is Block 2's).

```sql
DESCRIBE SELECT * FROM 'data/raw/owid_co2.csv';                      -- one row per column: its name and type
SUMMARIZE SELECT * FROM 'data/raw/owid_co2.csv';                      -- min, max, nulls, distinct, per column
SELECT COUNT(*), COUNT(DISTINCT (country, year)) FROM 'data/raw/owid_co2.csv';   -- rows against distinct keys
SELECT country, COUNT(*) AS n FROM 'data/raw/owid_co2.csv'
GROUP BY country ORDER BY n DESC;                                     -- the census: one row per value, read to the end
```

**The `SELECT` and its clauses.** They go in this order and no other.

```sql
SELECT country, co2              -- the columns you want, or * for all
FROM 'data/raw/owid_co2.csv'     -- the file
WHERE year = 2024                -- keep the rows where this is true
ORDER BY co2 DESC                -- DESC largest first; ASC (the default) smallest first
LIMIT 10;                        -- the first n rows of that order
```

**Comparisons in `WHERE`.** `=`, `!=`, `<`, `<=`, `>`, `>=`; text in single quotes, exactly as the file spells it
(`'International aviation'`, not `'international aviation'`). `IN ('A', 'B')` is "any of these".

**Missing values.** A missing value is `NULL`. It is not a value: `= NULL` and `!= NULL` are never true, so a
`WHERE` with either keeps nothing and says nothing. The only tests are `IS NULL` and `IS NOT NULL`.
pandas prints a `NULL` as `NaN` or `None`, neither of which is in the file: do not put those words in a query.

**Two conditions.** `AND` keeps a row when both are true; `OR` when either is.

```sql
WHERE year = 2024 AND country = 'World'
WHERE Description != 'Manual' OR Description IS NULL
```

**A view: a saved query with a name.** Write the filter once, read from it everywhere. It is not a copy of the
data; it re-runs its query each time it is read.

```sql
CREATE OR REPLACE VIEW products AS
SELECT * FROM 'data/raw/online_retail.parquet'
WHERE StockCode NOT IN ('POST', 'DOT', 'C2', 'BANK CHARGES');
-- then:  SELECT COUNT(*) FROM products;
```

**One number into Python.** `.fetchone()` returns the first row of the result; `[0]` takes its first column.

```python
world = con.sql("SELECT co2 FROM 'data/raw/owid_co2.csv' WHERE year = 2024 AND country = 'World'").fetchone()[0]
```

## Block 2 — one table, groups

**Aggregates** collapse many rows into one number. `COUNT(*)` counts rows; `COUNT(col)` counts the rows where
`col` is not `NULL`; `COUNT(DISTINCT col)` counts its different values. `SUM`, `AVG`, `MIN`, `MAX` skip `NULL`.
An average over a column with gaps is the average of the rows that have a value. (`countries`, below, is the view
you built in Lab 1.)


```sql
SELECT COUNT(*) AS n_rows, COUNT(co2) AS n_co2, COUNT(DISTINCT country) AS n_countries,
       SUM(co2) AS total, AVG(co2) AS mean, MIN(co2), MAX(co2)
FROM countries WHERE year = 2024;
```

**`GROUP BY`**: one output row per distinct value of the grouped columns. Every column in the `SELECT` that is
not inside an aggregate must be in the `GROUP BY`. A `NULL` in the grouped column is a group of its own.

```sql
SELECT year, SUM(co2) AS total
FROM countries
GROUP BY year
ORDER BY year;
```

**The order a query runs**: `FROM` → `WHERE` → `GROUP BY` → aggregates in `SELECT` → `ORDER BY` → `LIMIT`.
`WHERE` runs before any group exists, so it cannot test a count. A name you make with `AS` is born in
`SELECT`; this course repeats the expression in `WHERE` rather than the name.

**Calculated columns and aliases.** `Quantity * Price AS revenue`. `ROUND(x, 2)` is for display: round when you
print, never while you are still adding up.

**`COALESCE(col, 0)`**: the first of its values that is not `NULL`.

**Dates.** `year(InvoiceDate)`, `month(InvoiceDate)`, `date_trunc('month', InvoiceDate)` (the first day of that
month, so months group and sort correctly).

```sql
SELECT date_trunc('month', InvoiceDate) AS month, SUM(Quantity * Price) AS revenue
FROM sales_2010
GROUP BY month
ORDER BY month;
```

**Text.** `left(Invoice, 1)` is the first character.

**A query inside a query.** The inner `SELECT` makes a table; the outer one reads it.

```sql
SELECT SUM(revenue) FROM (
    SELECT year, SUM(co2) AS revenue FROM countries GROUP BY year
);
```

**Comparing two sums.** Never `==`: two correct sums of the same rows can differ in the last decimal.
`abs(a - b) < 0.005` is "equal to the penny"; the identity is that a breakdown adds up to its total, so the two
numbers tie out.
## Block 3 — two tables: how to join

**A join** puts columns from two tables in one row, matched on a key. Give each table a short name (`AS o`) and
say which table every column comes from (`o.order_id`).

```sql
SELECT o.order_id, o.order_status, c.customer_state
FROM orders AS o
JOIN customers AS c ON c.customer_id = o.customer_id;
```

**After every join, say what one row is now, and count:** the rows that went in, the rows that came out, and the
distinct keys that came out. With `JOIN`, all three equal means every row found exactly one partner.

```sql
SELECT (SELECT COUNT(*) FROM orders) AS rows_in,
       COUNT(*)                      AS rows_out,
       COUNT(DISTINCT o.order_id)    AS distinct_orders_out
FROM orders AS o
JOIN customers AS c ON c.customer_id = o.customer_id;
```

A key made of two columns is counted as a pair: `COUNT(DISTINCT (order_id, order_item_id))` (DuckDB's spelling; other
databases write it differently).

**Join, then group.** The result of a join is a table; `GROUP BY` works on it as on any other.

```sql
SELECT c.customer_state, COUNT(*) AS orders
FROM orders AS o
JOIN customers AS c ON c.customer_id = o.customer_id
GROUP BY c.customer_state
ORDER BY orders DESC;
```

**Two kinds of join.** `JOIN` (its full name is `INNER JOIN`) keeps only the rows that found a partner. `LEFT JOIN`
keeps every row of the table on the left; where a row has no partner, the right table's columns are `NULL`. So with
`LEFT JOIN`, three equal counts say only that every left row is there once: they do not say it found a partner.

```sql
SELECT o.order_id, i.price
FROM orders AS o
LEFT JOIN order_items AS i ON i.order_id = o.order_id;
```

**The rows with no partner:** a `LEFT JOIN`, then keep the rows whose right side is `NULL`. Test the side that may
be missing, the right table's key: it is `NULL` exactly when the row found no partner (a left-side key that is
itself `NULL` finds no partner either, and the right side is `NULL` for it too).

```sql
SELECT o.order_id, o.order_status
FROM orders AS o
LEFT JOIN order_items AS i ON i.order_id = o.order_id
WHERE i.order_id IS NULL;
```

**Joining texts** (Lab 3, section F; not on a slide). `'a' || 'b'` is `'ab'`; a text joined to `NULL` is `NULL`,
so `COALESCE(t.name_english, 'untranslated: ' || c.name, 'no category')` labels three kinds of row apart.

## Block 4 — when one row has several partners

**After `orders JOIN order_items`, one row of the result is one item, not one order**, because an order can have
several items. So `COUNT(*)` counts items and `AVG` averages over items, whatever the column is called.

**`HAVING`**: a `WHERE` on groups. It runs after `GROUP BY`, so it can test a count. This finds the keys that repeat.

```sql
SELECT order_id, COUNT(*) AS n FROM order_items
GROUP BY order_id
HAVING COUNT(*) > 1;
```

**One row per order first, then join.** `WITH name AS (...)` names a query you write first and read below.

```sql
WITH order_size AS (
    SELECT order_id, COUNT(*) AS n_items
    FROM order_items
    GROUP BY order_id
)
SELECT c.customer_state, COUNT(*) AS orders, AVG(s.n_items) AS items_per_order
FROM order_size AS s
JOIN orders AS o    ON o.order_id = s.order_id
JOIN customers AS c ON c.customer_id = o.customer_id
GROUP BY c.customer_state;
```
## Block 5 — did every record arrive? (Session 3)

Every example here is the Block 5 lecture's: the World Bank's GDP pages, `NY.GDP.MKTP.PP.KD`.

**JSON in plain words:** `{…}` is a record of named values; `[…]` is a list. **Any JSON, in DuckDB:**
`read_json_auto` reads it; `->>'name'` takes a named value out, as text; `->'name'` goes one level into a record and is
still JSON; `unnest` turns a list into rows. **The World Bank's pages:** each page is a list of two things, the
metadata (`page`, `pages`, `per_page`, `total`) and up to 1,000 records.

**Supplied, in the first cells: the pages as tables.** You read it; you never write it.
`make_table("gdp", "data/raw/api/worldbank/NY.GDP.MKTP.PP.KD_page*.json", "gdp", "DOUBLE")` makes `gdp` (`iso3`,
`country`, `year`, `gdp`: one row per record, from every page), `gdp_metadata` (one row per page: `page`, `pages`,
`per_page`, `total`) and `gdp_records`; the country list makes `places` (the World Bank's places) and `groups` (World,
the regions, the income groups).

**The `*` in a file name reads every file that matches:** `page*.json` reads every page; `page1.json` only the first,
with no error.

**Count against the total:** the rows of the table, beside what the pages say the whole answer holds.

```sql
SELECT COUNT(*) FROM gdp;                -- the rows you have
SELECT DISTINCT total FROM gdp_metadata; -- what the pages say the whole answer holds
```

**Places and groups:** the table holds groups too (`World` is the first row of a ranking of `gdp`). A question about
places joins to `places`.

## Block 6 — did every value survive? Bronze, silver, gold

Every example here is the Block 6 lecture's: the World Bank's GDP and population pages, as in Block 5.

**Bronze** is the data exactly as it arrived, never edited: every later table can be rebuilt from it. **Silver** is
cleaned and checked: the right types, consistent and explicit units, duplicates and missing values handled, sometimes
joined; still at the detail of the records. **Gold** is business-ready: shaped for the reports and the questions
people ask, often aggregated. A convention, not a law; and a layer is not a guarantee, so count between them.

**In the lecture:** bronze is the pages, read by the setup into `gdp_bronze` and `population_bronze` (`country`,
`year`, `value`: every column text); silver is `gdp_silver` and `population_silver` (the same rows, the value made a
number with `CAST`); gold is GDP per person, 2024.

**Silver: the value made a number.**

```sql
CREATE OR REPLACE VIEW gdp_silver AS
SELECT country,
       CAST(year AS INTEGER) AS year,
       CAST(value AS DOUBLE) AS gdp
FROM gdp_bronze;
```

**Gold: two silvers joined.** The `ON` line has two conditions: the same place and the same year.

```sql
CREATE OR REPLACE VIEW gold AS
SELECT g.country, g.gdp / p.population AS gdp_per_person
FROM gdp_silver AS g
JOIN population_silver AS p
  ON p.country = g.country AND p.year = g.year
WHERE g.year = 2024 AND g.gdp IS NOT NULL;
```

**Rows and numbers.** `COUNT(*)` counts rows; `COUNT(gdp)` counts the rows that hold a number: a `NULL` is still a row.

```sql
SELECT COUNT(*) - COUNT(gdp) AS no_number FROM gdp_silver WHERE year = 2024;   -- 23
SELECT COUNT(*) AS no_number FROM gdp_silver WHERE year = 2024 AND gdp IS NULL; -- 23
```

**`CAST` against `TRY_CAST`.** `CAST` refuses a value it cannot read, and names it: population made an `INTEGER`
(which holds up to 2,147,483,647) stops at `'3552306404'`. `TRY_CAST` never errors: what it cannot read becomes
`NULL`, silently: 3,810 numbers of bronze's 3,960. World's is one of them, so gold's World row loses its number.

**A Eurostat file (Lab 6):** the first column packs several codes; one column per year; every cell is text: a number
(`806`), a number and a flag after a space (`755 p`), or `:` (not available). **Supplied, in the first cells:**
`bronze` (the file, every cell as text), `bronze_long` (one row per series, place and year; `raw_value` is the cell as
Eurostat wrote it) and `silver_split` (`number_text`, the number still as text, `NULL` where the cell is `:`; and
`flag`). In bronze, a number is a cell that does not start with a colon: `raw_value NOT LIKE ':%'`.

A count that drops shows a loss; counts that agree cannot rule every loss out, so read the rows too. The first layer
where a count drops, with no reason you can name, is the layer to repair.
