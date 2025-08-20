# E-commerce Data Analysis — SQL & Python

Fifteen business questions answered against a ~100k-order Brazilian e-commerce dataset,
using MySQL for the analysis and Python for the loading and presentation.

The questions start at basic aggregation and work up through CTEs and window functions —
`DENSE_RANK`, `LAG`, and a windowed moving average. The SQL is the point of the project;
pandas, matplotlib and seaborn are there to display the results.

## The dataset

The [Target / Olist Brazilian e-commerce dataset](https://www.kaggle.com/datasets/devarajv88/target-dataset)
from Kaggle — seven CSVs covering customers, orders, order items, products, sellers,
payments and geolocation. The exact link is also in `Ecommerce_Dataset_Link.txt`.

The CSVs are **not** committed to this repository. Download them yourself with the link above.

## Notebooks

| File | What it does |
|---|---|
| **`Ecommerce.ipynb`** | The loader. Reads the seven CSVs, infers a MySQL column type for each pandas dtype, creates a table per file and inserts the rows. Run this once, first. |
| **`Questions.ipynb`** | The analysis. Fifteen questions, each with its SQL, the pandas handling, and a chart where a chart helps. |

## The fifteen questions

**Aggregation and filtering**

1. Unique customer cities
2. Orders placed in 2017
3. Total sales per product category
4. Share of orders paid in installments
5. Customers per state
6. Orders per month in 2018

**Subqueries and CTEs**

7. Average products per order, by customer city
8. Revenue share by product category
9. Correlation between product price and purchase count

**Window functions**

10. Sellers ranked by revenue — `DENSE_RANK()`
11. Moving average of order value per customer — `AVG() OVER (... ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)`
12. Cumulative sales per month — running `SUM() OVER`
13. Year-over-year sales growth — `LAG()`
14. Customer retention rate — two chained CTEs and a `LEFT JOIN`
15. Top 3 customers by spend per year — `DENSE_RANK()` with `PARTITION BY`

## Running it yourself

**1. Clone and install**

```bash
git clone https://github.com/pratik-j-patel/Data_Analysis_Project.git
cd Data_Analysis_Project
pip install -r requirements.txt
```

**2. Set up MySQL**

```sql
CREATE DATABASE Ecommerce;
```

**3. Configure credentials**

```bash
cp .env.example .env
```

Then edit `.env` with your MySQL host, user, password, and the folder where you unzipped
the Kaggle CSVs. `.env` is gitignored and should never be committed.

**4. Load, then analyse**

Run `Ecommerce.ipynb` first to populate the database, then work through `Questions.ipynb`.

The loader checks each table for existing rows and skips it if it already holds data, so
re-running the notebook is safe. To genuinely reload, drop the tables or the database first.

## Tools

MySQL 8 · Python 3.12 · pandas · numpy · matplotlib · seaborn · Jupyter

## One thing worth knowing about this dataset

`customer_id` is **not** a customer. It is a surrogate key minted fresh for every order — 99,441
of them against 96,096 real people. The column that identifies a person across orders is
`customer_unique_id`.

This matters more than it sounds. A retention query joined on `customer_id` cannot find a repeat
purchase, because every `customer_id` has exactly one order by construction — it returns 0% on any
data, and looks like a finding rather than a bug. Questions 5, 14 and 15 all use
`customer_unique_id` for that reason; question 7 deliberately does not, because it is measuring
orders rather than people.

## Notes and limitations

- **The loader has no uniqueness constraints.** Tables are created from inferred dtypes with
  no primary keys, so nothing in the schema prevents duplicate rows — the row-count check in
  the loader is the only guard. An earlier version had no guard at all, and the development
  database accumulated six copies of the dataset before a row count caught it.
- **Types are inferred, not declared.** Every non-numeric column becomes `TEXT`, so dates
  are stored as text and parsed with MySQL date functions at query time rather than being
  stored as `DATETIME`.
- **No test data or CI.** Correctness was checked by reading the output of each query.
- The project began as a follow-along from a YouTube SQL tutorial and was extended with
  additional questions — the window function work in particular — to practise the harder
  SQL.
