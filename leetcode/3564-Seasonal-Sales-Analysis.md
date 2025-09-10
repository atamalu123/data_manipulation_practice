# [3564. Seasonal Sales Analysis](https://leetcode.com/problems/seasonal-sales-analysis/)

* [SQL Solution](https://leetcode.com/problems/seasonal-sales-analysis/solutions/7173845/5-ctes-by-atamalu123-b5qt/)
* [Python Solution](https://leetcode.com/problems/seasonal-sales-analysis/solutions/7173873/pandas-solution-by-atamalu123-ou65/)

# Problem

Table: `sales`

```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| sale_id       | int     |
| product_id    | int     |
| sale_date     | date    |
| quantity      | int     |
| price         | decimal |
+---------------+---------+
sale_id is the unique identifier for this table.
Each row contains information about a product sale including the product_id, date of sale, quantity sold, and price per unit.
```

Table: `products`

```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| product_id    | int     |
| product_name  | varchar |
| category      | varchar |
+---------------+---------+
product_id is the unique identifier for this table.
Each row contains information about a product including its name and category.
```

Write a solution to find the most popular product category for each season. The seasons are defined as:
  * Winter: December, January, February
  * Spring: March, April, May
  * Summer: June, July, August
  * Fall: September, October, November

The popularity of a category is determined by the total quantity sold in that season. If there is a tie, select the category with the highest total revenue (quantity × price).

Return the result table ordered by season in ascending order.

The result format is in the following example.

 

Example:

```
Input:

sales table:

+---------+------------+------------+----------+-------+
| sale_id | product_id | sale_date  | quantity | price |
+---------+------------+------------+----------+-------+
| 1       | 1          | 2023-01-15 | 5        | 10.00 |
| 2       | 2          | 2023-01-20 | 4        | 15.00 |
| 3       | 3          | 2023-03-10 | 3        | 18.00 |
| 4       | 4          | 2023-04-05 | 1        | 20.00 |
| 5       | 1          | 2023-05-20 | 2        | 10.00 |
| 6       | 2          | 2023-06-12 | 4        | 15.00 |
| 7       | 5          | 2023-06-15 | 5        | 12.00 |
| 8       | 3          | 2023-07-24 | 2        | 18.00 |
| 9       | 4          | 2023-08-01 | 5        | 20.00 |
| 10      | 5          | 2023-09-03 | 3        | 12.00 |
| 11      | 1          | 2023-09-25 | 6        | 10.00 |
| 12      | 2          | 2023-11-10 | 4        | 15.00 |
| 13      | 3          | 2023-12-05 | 6        | 18.00 |
| 14      | 4          | 2023-12-22 | 3        | 20.00 |
| 15      | 5          | 2024-02-14 | 2        | 12.00 |
+---------+------------+------------+----------+-------+
products table:

+------------+-----------------+----------+
| product_id | product_name    | category |
+------------+-----------------+----------+
| 1          | Warm Jacket     | Apparel  |
| 2          | Designer Jeans  | Apparel  |
| 3          | Cutting Board   | Kitchen  |
| 4          | Smart Speaker   | Tech     |
| 5          | Yoga Mat        | Fitness  |
+------------+-----------------+----------+
```

```
Output:

+---------+----------+----------------+---------------+
| season  | category | total_quantity | total_revenue |
+---------+----------+----------------+---------------+
| Fall    | Apparel  | 10             | 120.00        |
| Spring  | Kitchen  | 3              | 54.00         |
| Summer  | Tech     | 5              | 100.00        |
| Winter  | Apparel  | 9              | 110.00        |
+---------+----------+----------------+---------------+
```

# SQL

[[Solution Link]](https://leetcode.com/problems/seasonal-sales-analysis/solutions/7173845/5-ctes-by-atamalu123-b5qt/)

```sql
WITH SalesFixed AS ( -- CTE 1: Create seasons
    SELECT 
      *,
      1.0 * quantity * price AS total_sales,
      CASE
        WHEN EXTRACT(MONTH FROM sale_date) IN (12, 1, 2) THEN 'Winter'
        WHEN EXTRACT(MONTH FROM sale_date) IN (3, 4, 5) THEN 'Spring'
        WHEN EXTRACT(MONTH FROM sale_date) IN (6, 7, 8) THEN 'Summer'
        ELSE 'Fall' END 
        AS season
    FROM 
      sales
), SalesAndProducts AS ( -- CTE 2: Combine sales and products tables
    SELECT
      *
    FROM 
      SalesFixed s
    LEFT JOIN
      products p
    ON 
      s.product_id = p.product_id
), TopProducts AS ( -- CTE 3: Calculate totals
    SELECT
      season,
      category,
      SUM(quantity) AS total_quantity,
      SUM(total_sales) AS total_revenue
    FROM
      SalesAndProducts
    GROUP BY
      season,
      category
), TopProductsRanked AS ( -- CTE 4: Rank top products
    SELECT
      season,
      category,
      total_quantity,
      total_revenue,
      DENSE_RANK() OVER(PARTITION BY season ORDER BY total_quantity DESC) AS quantity_rank
    FROM
      TopProducts
), TopProductsRanked2 AS ( -- CTE 5: Needed for tie-breaking 
    SELECT
      season,
      category,
      total_quantity,
      total_revenue,
      quantity_rank,
      DENSE_RANK() OVER(PARTITION BY season, total_quantity ORDER BY total_revenue DESC) AS revenue_rank
    FROM
      TopProductsRanked
)

-- Use tie-breakers to subset final table
SELECT
  season,
  category,
  total_quantity,
  total_revenue
FROM
  TopProductsRanked2
WHERE
  quantity_rank = 1 AND
  revenue_rank = 1;
```

# Python

[[Solution Link]](https://leetcode.com/problems/seasonal-sales-analysis/solutions/7173873/pandas-solution-by-atamalu123-ou65/)

```python
def seasonal_sales_analysis(products: pd.DataFrame, sales: pd.DataFrame) -> pd.DataFrame:
    
    # Map months to seasons
    month_to_season = {
        12: "Winter", 1: "Winter", 2: "Winter",
        3: "Spring", 4: "Spring", 5: "Spring",
        6: "Summer", 7: "Summer", 8: "Summer",
        9: "Fall", 10: "Fall", 11: "Fall"
    }

    sales["season"] = sales["sale_date"].dt.month.map(month_to_season)

    # Calculate revenue
    sales['revenue'] = sales['quantity'] * sales['price']

    # Combine data frames
    df = sales.merge(products, how = 'left', on = 'product_id')

    # Calculate totals by season & category
    grouped_df = df.groupby(['season', 'category']).agg(
        total_quantity=('quantity', 'sum'),
        total_revenue=('revenue', 'sum')
    ).reset_index()

    # Rank 1 for determining best category within season
    grouped_df["quantity_rank"] = (
        grouped_df.groupby("season")["total_quantity"].rank(method="dense", ascending = False)
    )

    # Rank 2
    grouped_df["revenue_rank"] = (
        grouped_df.groupby(["season", "quantity_rank"])["total_revenue"]
                .rank(method="dense", ascending = False)
    )
    
    # Apply rankings
    result = grouped_df[(grouped_df['quantity_rank'] == 1) & (grouped_df['revenue_rank'] == 1)]

    return result[['season', 'category', 'total_quantity', 'total_revenue']]
```
