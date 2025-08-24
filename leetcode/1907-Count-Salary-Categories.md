
# [1907. Count Salary Categories](https://leetcode.com/problems/count-salary-categories/)

- [Python Solution](#python)
- [R Solution](#r)
- [SQL Solution](https://leetcode.com/problems/count-salary-categories/solutions/7115764/stacking-using-union-by-atamalu123-5b9y/)

# Problem

    Table: Accounts

    +-------------+------+
    | Column Name | Type |
    +-------------+------+
    | account_id  | int  |
    | income      | int  |
    +-------------+------+
    account_id is the primary key (column with unique values) for this table.

    Each row contains information about the monthly income for one bank account.

Write a solution to calculate the number of bank accounts for each
salary category. The salary categories are:

- “Low Salary”: All the salaries strictly less than \$20000.
- “Average Salary”: All the salaries in the inclusive range \[\$20000,
  \$50000\].
- “High Salary”: All the salaries strictly greater than \$50000.

The result table must contain all three categories. If there are no
accounts in a category, return 0.

Return the result table in any order.

## Data

``` python
import pandas as pd 

data = [[3, 108939], [2, 12747], [8, 87709], [6, 91796]]
accounts = pd.DataFrame(data, columns=['account_id', 'income']).astype({'account_id':'int', 'income':'int'})

r.accounts = accounts
```

## Python

``` python
def count_salary_categories(accounts: pd.DataFrame) -> pd.DataFrame:
    
    low_salary = len(accounts[accounts['income'] < 20000])
    average_salary = len(accounts[(accounts['income'] >= 20000) & (accounts['income'] <= 50000)])
    high_salary = len(accounts[accounts['income'] > 50000])

    return pd.DataFrame({'category': ['Low Salary', 'Average Salary', 'High Salary'], 'accounts_count': [low_salary, average_salary, high_salary]})
  
print(count_salary_categories(accounts))
```

    ##          category  accounts_count
    ## 0      Low Salary               1
    ## 1  Average Salary               0
    ## 2     High Salary               3

## R

``` r
library(dplyr)

count_salary_categories <- function(accounts){
  
  low_salary <- accounts %>%
    filter(income < 20000) %>%
    nrow()
  
  average_salary <- accounts %>%
    filter(income >= 20000 & income <= 50000) %>%
    nrow()
  
  high_salary <- accounts %>%
    filter(income > 50000) %>%
    nrow()
  
  res <- data.frame(category = c('Low Salary', 'Average Salary', 'High Salary'),
                    accounts_count = c(low_salary, average_salary, high_salary))
  
  return(res)

}

print(count_salary_categories(accounts))
```

    ##         category accounts_count
    ## 1     Low Salary              1
    ## 2 Average Salary              0
    ## 3    High Salary              3
