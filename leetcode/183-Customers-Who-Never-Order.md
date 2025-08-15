
# [183. Customers Who Never Order](https://leetcode.com/problems/customers-who-never-order/)

- [Python Solution](#python)
- [R Solution](#r)
- [SQL Solution](https://leetcode.com/problems/customers-who-never-order/solutions/7082081/two-versions-in-mysql-by-atamalu123-plha/)

# Problem

    Table: Customers

    +-------------+---------+
    | Column Name | Type    |
    +-------------+---------+
    | id          | int     |
    | name        | varchar |
    +-------------+---------+
    id is the primary key (column with unique values) for this table.
    Each row of this table indicates the ID and name of a customer.

    Table: Orders

    +-------------+------+
    | Column Name | Type |
    +-------------+------+
    | id          | int  |
    | customerId  | int  |
    +-------------+------+
    id is the primary key (column with unique values) for this table.
    customerId is a foreign key (reference columns) of the ID from the Customers table.
    Each row of this table indicates the ID of an order and the ID of the customer who ordered it.

Write a solution to find all customers who never order anything.

Return the result table in any order.

## Data

``` python
import pandas as pd

data = [[1, 'Joe'], [2, 'Henry'], [3, 'Sam'], [4, 'Max']]
customers = pd.DataFrame(data, columns=['id', 'name']).astype({'id':'int', 'name':'object'})
data = [[1, 3], [2, 1]]
orders = pd.DataFrame(data, columns=['id', 'customerId']).astype({'id':'int', 'customerId':'int'})

r.customers = customers
r.orders = orders
```

## Python

``` python
def find_customers(customers: pd.DataFrame, orders: pd.DataFrame) -> pd.DataFrame:
    
    ordered_customers = orders['customerId'].unique()

    filtered_customers = customers[~customers['id'].isin(ordered_customers)]

    filtered_customers = filtered_customers.rename(columns={'name': 'Customers'})

    return(filtered_customers[['Customers']])
  
print(find_customers(customers, orders))
```

    ##   Customers
    ## 1     Henry
    ## 3       Max

## R

``` r
library(dplyr)

find_customers <- function(customers, orders){
  
  ordered_customers <- unique(orders$customerId)
  
  res <- customers %>%
    filter(!id %in% ordered_customers) %>%
    select(name)
  
  return(res)
}

print(find_customers(customers, orders))
```

    ##    name
    ## 1 Henry
    ## 2   Max
