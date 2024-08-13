## Lag function in SQL (with example)

[Lag function explanation](https://www.sqlshack.com/sql-lag-function-overview-and-examples/)

LAG function is an analytic function in SQL that allows you to access a value from a previous row in the same result set without needing to join on itself. It is particularly useful for comparisons within a sequential set of rows, such as calculating differences between consecutive rows.

### Step 1: Create the Table

First, let's create the `daily_sales` table.

```sql
CREATE TABLE daily_sales (
    id INT PRIMARY KEY,
    sale_date DATE,
    sales_amount DECIMAL(10, 2)
);
```

### Step 2: Insert Sample Data

Next, we'll insert some sample data into the `daily_sales` table.

```sql
INSERT INTO daily_sales (id, sale_date, sales_amount) VALUES
(1, '2023-10-01', 150.00),
(2, '2023-10-02', 200.50),
(3, '2023-10-03', 180.75),
(4, '2023-10-04', 220.00),
(5, '2023-10-05', 190.25);
```

### Step 3: Use the LAG Function

Now, let's use the `LAG` function to compare each day's sales with the previous day's sales.

```sql
SELECT
    id,
    sale_date,
    sales_amount,
    LAG(sales_amount, 1) OVER (ORDER BY sale_date) AS previous_sales_amount,
    sales_amount - LAG(sales_amount, 1) OVER (ORDER BY sale_date) AS sales_difference
FROM
    daily_sales;
```

### Result

This query will produce the following result:

```
+----+------------+--------------+----------------------+------------------+
| id | sale_date  | sales_amount | previous_sales_amount | sales_difference |
+----+------------+--------------+----------------------+------------------+
| 1  | 2023-10-01 |      150.00  |          NULL         |       NULL       |
| 2  | 2023-10-02 |      200.50  |         150.00        |       50.50      |
| 3  | 2023-10-03 |      180.75  |         200.50        |      -19.75      |
| 4  | 2023-10-04 |      220.00  |         180.75        |       39.25      |
| 5  | 2023-10-05 |      190.25  |         220.00        |      -29.75      |
+----+------------+--------------+----------------------+------------------+
```

### Explanation

- `LAG(sales_amount, 1) OVER (ORDER BY sale_date) AS previous_sales_amount`: This part of the query retrieves the `sales_amount` from the previous row based on the order of `sale_date`. If there is no previous row (like for the first row), it returns `NULL`.
- `sales_amount - LAG(sales_amount, 1) OVER (ORDER BY sale_date) AS sales_difference`: This calculates the difference between the current day's sales and the previous day's sales.

This example shows how you can use the `LAG` function to perform row-wise comparisons in table data, which is very useful for analyzing trends and changes over time.
