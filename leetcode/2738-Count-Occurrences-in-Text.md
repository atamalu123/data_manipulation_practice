
# [2738. Count Occurrences in Text](https://leetcode.com/problems/count-occurrences-in-text/)

- [Python Solution](#python)
- [R Solution](#r)

# Problem

    Table: Files

    +-------------+---------+
    | Column Name | Type    |
    +-- ----------+---------+
    | file_name   | varchar |
    | content     | text    |
    +-------------+---------+
    file_name is the column with unique values of this table. 

    Each row contains file_name and the content of that file.

Write a solution to find the number of files that have at least one
occurrence of the words ‘bull’ and ‘bear’ as a standalone word,
respectively, disregarding any instances where it appears without space
on either side (e.g. ‘bullet’, ‘bears’, ‘bull.’, or ‘bear’ at the
beginning or end of a sentence will not be considered)

Return the word ‘bull’ and ‘bear’ along with the corresponding number of
occurrences in any order.

## Data

``` python
import pandas as pd

data = [['draft1.txt', 'The stock exchange predicts a bull market which would make many investors happy.'], ['draft2.txt', 'The stock exchange predicts a bull market which would make many investors happy, but analysts warn of possibility of too much optimism and that in fact we are awaiting a bear market.'], ['final.txt', 'The stock exchange predicts a bull market which would make many investors happy, but analysts warn of possibility of too much optimism and that in fact we are awaiting a bear market. As always predicting the future market is an uncertain game and all investors should follow their instincts and best practices.']]
files = pd.DataFrame(data, columns=['file_name', 'content']).astype({'file_name':'object', 'content':'object'})

r.files = files
```

## Python

``` python
def count_occurrences(files: pd.DataFrame) -> pd.DataFrame:
    
    num_bulls = len(files[files['content'].str.contains('.* bull .*', regex = True)])
    num_bears = len(files[files['content'].str.contains('.* bear .*', regex = True)])

    res = pd.DataFrame({'word': ['bull', 'bear'], 'count': [num_bulls, num_bears]})

    return res
  
print(count_occurrences(files))
```

    ##    word  count
    ## 0  bull      3
    ## 1  bear      2

## R

``` r
library(dplyr)

count_occurrences <- function(files){
  
  num_bulls <- nrow(files[grepl(".* bull .*", files$content), ])
  num_bears <- nrow(files[grepl(".* bear .*", files$content), ])
  
  res <- data.frame(word = c('bull', 'bear'), 
                    count = c(num_bulls, num_bears))
  
  return(res)

}

print(count_occurrences(files))
```

    ##   word count
    ## 1 bull     3
    ## 2 bear     2
