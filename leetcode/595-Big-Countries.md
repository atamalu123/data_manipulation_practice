
# [595. Big Countries](https://leetcode.com/problems/big-countries/)

- [Python Solution](#python)
- [R Solution](#r)
- [SQL Solution](https://leetcode.com/problems/big-countries/solutions/7167020/filtering-by-atamalu123-zzss/)

# Problem

A country is big if:

- It has an area of at least three million (i.e., 3000000 km2), or
- It has a population of at least twenty-five million (i.e., 25000000).

Write a solution to find the name, population, and area of the big
countries. Return the result table in any order.

## Data

``` python
import pandas as pd

data = [['Afghanistan', 'Asia', 652230, 25500100, 20343000000], ['Albania', 'Europe', 28748, 2831741, 12960000000], ['Algeria', 'Africa', 2381741, 37100000, 188681000000], ['Andorra', 'Europe', 468, 78115, 3712000000], ['Angola', 'Africa', 1246700, 20609294, 100990000000]]
world = pd.DataFrame(data, columns=['name', 'continent', 'area', 'population', 'gdp']).astype({'name':'object', 'continent':'object', 'area':'int', 'population':'int', 'gdp':'int'})

r.world = world
```

## Python

``` python
def big_countries(world: pd.DataFrame) -> pd.DataFrame:
    
    df = world[(world['area'] >= 3000000) | (world['population'] >= 25000000)][['name', 'population', 'area']]

    return df
  
print(big_countries(world))
```

    ##           name  population     area
    ## 0  Afghanistan    25500100   652230
    ## 2      Algeria    37100000  2381741

## R

``` r
library(dplyr)

big_countries <- function(world){
  res <- world %>%
    filter(area >= 3000000 | population >= 25000000) %>%
    select(name, population, area)
  
  return(res)
}

print(big_countries(world))
```

    ##          name population    area
    ## 1 Afghanistan   25500100  652230
    ## 2     Algeria   37100000 2381741
