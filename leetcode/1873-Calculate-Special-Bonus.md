
# [1873. Calculate Special Bonus](https://leetcode.com/problems/calculate-special-bonus/)

- [Python Solution](#python)
- [R Solution](#r)

# Problem

    Table: Employees

    +-------------+---------+
    | Column Name | Type    |
    +-------------+---------+
    | employee_id | int     |
    | name        | varchar |
    | salary      | int     |
    +-------------+---------+
    employee_id is the primary key (column with unique values) for this table.

    Each row of this table indicates the employee ID, employee name, and salary.

Write a solution to calculate the bonus of each employee. The bonus of
an employee is 100% of their salary if the ID of the employee is an odd
number and the employee’s name does not start with the character ‘M’.
The bonus of an employee is 0 otherwise.

Return the result table ordered by `employee_id`.

## Data

``` python
import pandas as pd

data = [[2, 'Meir', 3000], [3, 'Michael', 3800], [7, 'Addilyn', 7400], [8, 'Juan', 6100], [9, 'Kannon', 7700]]
employees = pd.DataFrame(data, columns=['employee_id', 'name', 'salary']).astype({'employee_id':'int', 'name':'object', 'salary':'int'})

r.employees = employees
```

## Python

``` python
def calculate_special_bonus(employees: pd.DataFrame) -> pd.DataFrame:
    
    employees['bonus'] = employees.apply(
        lambda x: x['salary'] if x['employee_id'] % 2 != 0 and not x['name'].startswith('M') else 0,
        axis = 1
    )

    return employees[['employee_id', 'bonus']].sort_values(by = 'employee_id')
  
print(calculate_special_bonus(employees))
```

    ##    employee_id  bonus
    ## 0            2      0
    ## 1            3      0
    ## 2            7   7400
    ## 3            8      0
    ## 4            9   7700

## R

``` r
library(dplyr)

calculate_special_bonus <- function(employees){
  
  employees$bonus <- ifelse(employees$employee_id %% 2 != 0 & substring(employees$name, 1, 1) != 'M', employees$salary, 0)
  
  res = employees %>%
    select(employee_id, bonus) %>%
    arrange(employee_id)
  
  return(res)
  
}

print(calculate_special_bonus(employees))
```

    ##   employee_id bonus
    ## 1           2     0
    ## 2           3     0
    ## 3           7  7400
    ## 4           8     0
    ## 5           9  7700
