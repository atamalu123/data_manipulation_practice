# [3580. Find Consistently Improving Employees](https://leetcode.com/problems/find-consistently-improving-employees/)

* [SQL Solution](https://leetcode.com/problems/find-consistently-improving-employees/solutions/7173953/cte-lag-dense_rank-by-atamalu123-3evd/)
* [Python Solution](https://leetcode.com/problems/find-consistently-improving-employees/solutions/7174006/pandas-solution-by-atamalu123-fz6z/)

# Problem

Table: `employees`

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| employee_id | int     |
| name        | varchar |
+-------------+---------+
employee_id is the unique identifier for this table.
Each row contains information about an employee.
```

Table: `performance_reviews`

```
+-------------+------+
| Column Name | Type |
+-------------+------+
| review_id   | int  |
| employee_id | int  |
| review_date | date |
| rating      | int  |
+-------------+------+
review_id is the unique identifier for this table.
Each row represents a performance review for an employee. The rating is on a scale of 1-5 where 5 is excellent and 1 is poor.
```
Write a solution to find employees who have consistently improved their performance over their last three reviews.

An employee must have at least 3 review to be considered
  * The employee's last 3 reviews must show strictly increasing ratings (each review better than the previous)
  * Use the most recent 3 reviews based on review_date for each employee

Calculate the improvement score as the difference between the latest rating and the earliest rating among the last 3 reviews
Return the result table ordered by improvement score in descending order, then by name in ascending order.

The result format is in the following example.

 

Example:

```
Input:

employees table:

+-------------+----------------+
| employee_id | name           |
+-------------+----------------+
| 1           | Alice Johnson  |
| 2           | Bob Smith      |
| 3           | Carol Davis    |
| 4           | David Wilson   |
| 5           | Emma Brown     |
+-------------+----------------+
performance_reviews table:

+-----------+-------------+-------------+--------+
| review_id | employee_id | review_date | rating |
+-----------+-------------+-------------+--------+
| 1         | 1           | 2023-01-15  | 2      |
| 2         | 1           | 2023-04-15  | 3      |
| 3         | 1           | 2023-07-15  | 4      |
| 4         | 1           | 2023-10-15  | 5      |
| 5         | 2           | 2023-02-01  | 3      |
| 6         | 2           | 2023-05-01  | 2      |
| 7         | 2           | 2023-08-01  | 4      |
| 8         | 2           | 2023-11-01  | 5      |
| 9         | 3           | 2023-03-10  | 1      |
| 10        | 3           | 2023-06-10  | 2      |
| 11        | 3           | 2023-09-10  | 3      |
| 12        | 3           | 2023-12-10  | 4      |
| 13        | 4           | 2023-01-20  | 4      |
| 14        | 4           | 2023-04-20  | 4      |
| 15        | 4           | 2023-07-20  | 4      |
| 16        | 5           | 2023-02-15  | 3      |
| 17        | 5           | 2023-05-15  | 2      |
+-----------+-------------+-------------+--------+
```
```
Output:

+-------------+----------------+-------------------+
| employee_id | name           | improvement_score |
+-------------+----------------+-------------------+
| 2           | Bob Smith      | 3                 |
| 1           | Alice Johnson  | 2                 |
| 3           | Carol Davis    | 2                 |
+-------------+----------------+-------------------+
```

# SQL

```sql
WITH ThreePlusReviews AS ( -- CTE 1: IDs where customers have 3+ reviews
    SELECT
      employee_id
    FROM 
      performance_reviews
    GROUP BY
      employee_id
    HAVING 
      COUNT(*) >= 3
), PerformanceRanked AS ( -- CTE 2: Rank performances so we can filter most recent 3
    SELECT
      *,
      DENSE_RANK() OVER(PARTITION BY employee_id ORDER BY review_date DESC) AS rnk
    FROM
      performance_reviews
), PerformanceWithLags AS ( -- CTE 3: add lags to performance_reviews table
    SELECT
      employee_id,
      review_date,
      rating,
      LAG(rating, 1) OVER(PARTITION BY employee_id ORDER BY review_date ASC) AS prev1_rating,
      LAG(rating, 2) OVER(PARTITION BY employee_id ORDER BY review_date ASC) AS prev2_rating
    FROM
      PerformanceRanked
    WHERE 
      employee_id IN (SELECT employee_id FROM ThreePlusReviews) AND
      rnk <= 3
)

SELECT
  p.employee_id,
  e.name,
  p.rating - p.prev2_rating AS improvement_score
FROM
  PerformanceWithLags p
LEFT JOIN
  employees e
ON 
  p.employee_id = e.employee_id
WHERE
  p.prev1_rating > p.prev2_rating AND
  p.rating > p.prev1_rating
ORDER BY
  improvement_score DESC,
  e.name ASC
```

# Python

```python
def find_consistently_improving_employees(employees: pd.DataFrame,
                                          performance_reviews: pd.DataFrame) -> pd.DataFrame:
    pr = performance_reviews.copy()
    pr['review_date'] = pd.to_datetime(pr['review_date'])

    # Keep only employees with >= 3 reviews
    eligible_ids = pr['employee_id'].value_counts()
    pr = pr[pr['employee_id'].isin(eligible_ids[eligible_ids >= 3].index)].copy()

    # Sort ascending by date, then keep the last 3 per employee (the most recent 3)
    pr = pr.sort_values(['employee_id', 'review_date'])
    last3 = pr.groupby('employee_id').tail(3).copy()

    # Ensure we truly have 3 for each employee
    valid_ids = last3['employee_id'].value_counts()
    last3 = last3[last3['employee_id'].isin(valid_ids[valid_ids == 3].index)].copy()

    # Label the 3 rows as oldest(0), mid(1), latest(2) within each employee
    last3['pos'] = last3.groupby('employee_id').cumcount()

    # Pivot to wide to compare ratings directly
    wide = (last3.pivot(index='employee_id', columns='pos', values='rating')
                 .rename(columns={0:'oldest', 1:'mid', 2:'latest'})
                 .reset_index())

    # Strictly increasing over the last 3; improvement = latest - oldest
    improving = wide[(wide['oldest'] < wide['mid']) & (wide['mid'] < wide['latest'])].copy()
    improving['improvement_score'] = improving['latest'] - improving['oldest']

    # Join names and sort as required
    out = (improving[['employee_id', 'improvement_score']]
           .merge(employees, on='employee_id', how='left')
           .sort_values(['improvement_score', 'name'], ascending=[False, True])
           .reset_index(drop=True))

    return out[['employee_id', 'name', 'improvement_score']]
```
