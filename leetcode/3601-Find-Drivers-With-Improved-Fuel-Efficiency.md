# [3601. Find Drivers with Improved Fuel Efficiency](https://leetcode.com/problems/find-drivers-with-improved-fuel-efficiency/)

* [SQL Solution](https://leetcode.com/problems/find-drivers-with-improved-fuel-efficiency/solutions/7174096/using-ctes-by-atamalu123-8763/)
* [Python Solution](https://leetcode.com/problems/find-drivers-with-improved-fuel-efficiency/solutions/7174175/11-steps-by-atamalu123-171e/)

# Problem

Table: `drivers`

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| driver_id   | int     |
| driver_name | varchar |
+-------------+---------+
driver_id is the unique identifier for this table.
Each row contains information about a driver.
```

Table: `trips`

```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| trip_id       | int     |
| driver_id     | int     |
| trip_date     | date    |
| distance_km   | decimal |
| fuel_consumed | decimal |
+---------------+---------+
trip_id is the unique identifier for this table.
Each row represents a trip made by a driver, including the distance traveled and fuel consumed for that trip.
```

Write a solution to find drivers whose fuel efficiency has improved by comparing their average fuel efficiency in the first half of the year with the second half of the year.

* Calculate fuel efficiency as distance_km / fuel_consumed for each trip
* First half: January to June, Second half: July to December
* Only include drivers who have trips in both halves of the year
* Calculate the efficiency improvement as (second_half_avg - first_half_avg)
* Round all results to 2 decimal places

Return the result table ordered by efficiency improvement in descending order, then by driver name in ascending order.

The result format is in the following example.

 

Example:

```
Input:

drivers table:

+-----------+---------------+
| driver_id | driver_name   |
+-----------+---------------+
| 1         | Alice Johnson |
| 2         | Bob Smith     |
| 3         | Carol Davis   |
| 4         | David Wilson  |
| 5         | Emma Brown    |
+-----------+---------------+
trips table:

+---------+-----------+------------+-------------+---------------+
| trip_id | driver_id | trip_date  | distance_km | fuel_consumed |
+---------+-----------+------------+-------------+---------------+
| 1       | 1         | 2023-02-15 | 120.5       | 10.2          |
| 2       | 1         | 2023-03-20 | 200.0       | 16.5          |
| 3       | 1         | 2023-08-10 | 150.0       | 11.0          |
| 4       | 1         | 2023-09-25 | 180.0       | 12.5          |
| 5       | 2         | 2023-01-10 | 100.0       | 9.0           |
| 6       | 2         | 2023-04-15 | 250.0       | 22.0          |
| 7       | 2         | 2023-10-05 | 200.0       | 15.0          |
| 8       | 3         | 2023-03-12 | 80.0        | 8.5           |
| 9       | 3         | 2023-05-18 | 90.0        | 9.2           |
| 10      | 4         | 2023-07-22 | 160.0       | 12.8          |
| 11      | 4         | 2023-11-30 | 140.0       | 11.0          |
| 12      | 5         | 2023-02-28 | 110.0       | 11.5          |
+---------+-----------+------------+-------------+---------------+
```
```
Output:

+-----------+---------------+------------------+-------------------+------------------------+
| driver_id | driver_name   | first_half_avg   | second_half_avg   | efficiency_improvement |
+-----------+---------------+------------------+-------------------+------------------------+
| 2         | Bob Smith     | 11.24            | 13.33             | 2.10                   |
| 1         | Alice Johnson | 11.97            | 14.02             | 2.05                   |
+-----------+---------------+------------------+-------------------+------------------------
```

# SQL

```sql
WITH TripsFinal AS ( -- CTE 1: Calculate fuel efficiency and create halves
    SELECT
      *,
      distance_km / fuel_consumed AS fuel_efficiency,
      CASE 
        WHEN EXTRACT(MONTH FROM trip_date) IN (1, 2, 3, 4, 5, 6) THEN '1'
        ELSE '2' END
        AS half
    FROM
      trips
), DriversWithBoth AS ( -- CTE 2: Extract IDs with both halves
    SELECT
      driver_id
    FROM
      TripsFinal
    GROUP BY
      driver_id
    HAVING
      COUNT(DISTINCT half) = 2
), FirstHalf AS ( -- CTE 3: First half average
    SELECT
      driver_id,
      AVG(fuel_efficiency) AS first_half_avg
    FROM
      TripsFinal
    WHERE
      driver_id IN (SELECT driver_id FROM DriversWithBoth) AND
      half = '1'
    GROUP BY
      driver_id
), SecondHalf AS ( -- CTE 4: Second half average
    SELECT
      driver_id,
      AVG(fuel_efficiency) AS second_half_avg
    FROM
      TripsFinal
    WHERE
      driver_id IN (SELECT driver_id FROM DriversWithBoth) AND
      half = '2'
    GROUP BY
      driver_id
)

SELECT
  f.driver_id,
  d.driver_name,
  ROUND(f.first_half_avg, 2) AS first_half_avg,
  ROUND(s.second_half_avg, 2) AS second_half_avg,
  ROUND(s.second_half_avg - f.first_half_avg, 2) AS efficiency_improvement
FROM
  FirstHalf f
LEFT JOIN
  SecondHalf s
ON 
  f.driver_id = s.driver_id
LEFT JOIN
  drivers d
ON
  s.driver_id = d.driver_id
WHERE
  s.second_half_avg - f.first_half_avg > 0 -- keep only those who have improved
ORDER BY
  efficiency_improvement DESC,
  d.driver_name ASC;
```

# Python

```python
def find_improved_efficiency_drivers(drivers: pd.DataFrame, trips: pd.DataFrame) -> pd.DataFrame:
    trips['trip_date'] = pd.to_datetime(trips['trip_date'])

    # 1. Combine data frames
    trips = trips.merge(drivers, how = 'left', on = 'driver_id')

    # 2. Calculate fuel efficiency
    trips['fuel_efficiency'] = trips['distance_km'] / trips['fuel_consumed']

    # 3. Create halves
    half_map = {
        1: '1', 2: '1', 3: '1', 4: '1', 5: '1', 6: '1',
        7: '2', 8: '2', 9: '2', 10: '2', 11: '2', 12: '2'
    }

    trips['half'] = trips['trip_date'].dt.month.map(
        half_map
    )

    # 4. Extract drivers who have values from both halves
    qualified_drivers = trips.groupby('driver_id').agg(
        num_halves=('half', 'nunique')
    ).reset_index()
    qualified_drivers = qualified_drivers[qualified_drivers['num_halves'] == 2]['driver_id']

    # 5. Filter only drivers with both halves
    trips = trips[trips['driver_id'].isin(qualified_drivers)]

    # 6. Create dataframes for each half
    first_half = trips[trips['half'] == '1'].groupby(['driver_id', 'driver_name'])['fuel_efficiency'].mean().reset_index(name = 'first_half_avg')
    second_half = trips[trips['half'] == '2'].groupby(['driver_id', 'driver_name'])['fuel_efficiency'].mean().reset_index(name = 'second_half_avg')

    # 7. Combine data frames
    result = first_half.merge(second_half, how = 'left', on = ['driver_id', 'driver_name'])

    # 8. Calculate fuel efficiency improvement
    result['efficiency_improvement'] = result['second_half_avg'] - result['first_half_avg']
    
    # 9. Filter out non-improved
    result = result[result['efficiency_improvement'] > 0]

    # 10. Round variables
    result['first_half_avg'] = round(result['first_half_avg'], 2)
    result['second_half_avg'] = round(result['second_half_avg'], 2)
    result['efficiency_improvement'] = round(result['efficiency_improvement'], 2)

    # 11. Reorder
    result = result.sort_values(by=['efficiency_improvement', 'driver_name'], ascending = [False, True])

    return result[['driver_id', 'driver_name', 'first_half_avg', 'second_half_avg', 'efficiency_improvement']]

```
