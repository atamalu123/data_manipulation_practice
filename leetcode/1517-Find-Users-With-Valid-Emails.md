
# [1517. Find Users With Valid E-Mails](https://leetcode.com/problems/find-users-with-valid-e-mails/)

- [Python Solution](#python)
- [R Solution](#r)

# Problem

    Table: Users

    +---------------+---------+
    | Column Name   | Type    |
    +---------------+---------+
    | user_id       | int     |
    | name          | varchar |
    | mail          | varchar |
    +---------------+---------+
    user_id is the primary key (column with unique values) for this table.

    This table contains information of the users signed up in a website. Some e-mails are invalid.

Write a solution to find the users who have valid emails.

A valid e-mail has a prefix name and a domain where:

- The prefix name is a string that may contain letters (upper or lower
  case), digits, underscore ’\_‘, period’.’, and/or dash ‘-’. The prefix
  name must start with a letter.
- The domain is ‘@leetcode.com’.

Return the result table in any order.

## Data

``` python
import pandas as pd

data = [[1, 'Winston', 'winston@leetcode.com'], [2, 'Jonathan', 'jonathanisgreat'], [3, 'Annabelle', 'bella-@leetcode.com'], [4, 'Sally', 'sally.come@leetcode.com'], [5, 'Marwan', 'quarz#2020@leetcode.com'], [6, 'David', 'david69@gmail.com'], [7, 'Shapiro', '.shapo@leetcode.com']]
users = pd.DataFrame(data, columns=['user_id', 'name', 'mail']).astype({'user_id':'int', 'name':'object', 'mail':'object'})

r.users = users
```

## Python

``` python
def valid_emails(users: pd.DataFrame) -> pd.DataFrame:
    
    return users[users["mail"].str.match(r"^[a-zA-Z][a-zA-Z0-9_.-]*\@leetcode\.com$")]
  
print(valid_emails(users))
```

    ##    user_id       name                     mail
    ## 0        1    Winston     winston@leetcode.com
    ## 2        3  Annabelle      bella-@leetcode.com
    ## 3        4      Sally  sally.come@leetcode.com

## R

``` r
library(dplyr)

valid_users <- function(users){
  
  res <- users[grepl("^[a-zA-Z][a-zA-Z0-9_.-]*@leetcode\\.com$", users$mail), ]
  
  return(res)

}

print(valid_users(users))
```

    ##   user_id      name                    mail
    ## 1       1   Winston    winston@leetcode.com
    ## 3       3 Annabelle     bella-@leetcode.com
    ## 4       4     Sally sally.come@leetcode.com
