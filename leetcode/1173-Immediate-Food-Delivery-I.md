
# [1173. Immediate Food Delivery I](https://leetcode.com/problems/immediate-food-delivery-i/)

- [Python Solution](#python)
- [R Solution](#r)

# Problem

    Table: Delivery

    +-----------------------------+---------+
    | Column Name                 | Type    |
    +-----------------------------+---------+
    | delivery_id                 | int     |
    | customer_id                 | int     |
    | order_date                  | date    |
    | customer_pref_delivery_date | date    |
    +-----------------------------+---------+
    delivery_id is the primary key (column with unique values) of this table.

    The table holds information about food delivery to customers that make orders at some date and specify a preferred delivery date (on the same order date or after it).

If the customer’s preferred delivery date is the same as the order date,
then the order is called immediate; otherwise, it is called scheduled.

Write a solution to find the percentage of immediate orders in the
table, rounded to 2 decimal places.

## Data

``` python
import pandas as pd 

data = [[1, 1, '2019-08-01', '2019-08-02'], [2, 5, '2019-08-02', '2019-08-02'], [3, 1, '2019-08-11', '2019-08-11'], [4, 3, '2019-08-24', '2019-08-26'], [5, 4, '2019-08-21', '2019-08-22'], [6, 2, '2019-08-11', '2019-08-13']]
delivery = pd.DataFrame(data, columns=['delivery_id', 'customer_id', 'order_date', 'customer_pref_delivery_date']).astype({'delivery_id':'int', 'customer_id':'int', 'order_date':'datetime64[ns]', 'customer_pref_delivery_date':'datetime64[ns]'})

r.delivery = delivery
```

## Python

``` python
def food_delivery(delivery: pd.DataFrame) -> pd.DataFrame:
    
    num_immediate = len(delivery[delivery['order_date'] == delivery['customer_pref_delivery_date']])

    percent_immediate = round(100 * (num_immediate / len(delivery)), 2)

    return pd.DataFrame({'immediate_percentage': [percent_immediate]})
  
print(food_delivery(delivery))
```

    ##    immediate_percentage
    ## 0                 33.33

## R

``` r
library(dplyr)

food_delivery <- function(delivery){
  
  delivery_filtered <- delivery %>%
    filter(order_date == customer_pref_delivery_date)
  
  perc <- round(100 * (nrow(delivery_filtered) / nrow(delivery)), 2)
  
  res <- data.frame(immediate_percentage = perc)
  
  return(res)

}

print(food_delivery(delivery))
```

    ##   immediate_percentage
    ## 1                33.33
