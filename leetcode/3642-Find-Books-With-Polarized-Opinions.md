# [3642. Find Books with Polarized Opinions](https://leetcode.com/problems/find-books-with-polarized-opinions/)

* [SQL Solution](https://leetcode.com/problems/find-books-with-polarized-opinions/solutions/7174266/using-cte-by-atamalu123-ynem/)
* [Python Solution](https://leetcode.com/problems/find-books-with-polarized-opinions/solutions/7174278/pandas-solution-by-atamalu123-3h04/)

# Problem

Table: `books`

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| book_id     | int     |
| title       | varchar |
| author      | varchar |
| genre       | varchar |
| pages       | int     |
+-------------+---------+
book_id is the unique ID for this table.
Each row contains information about a book including its genre and page count.
```

Table: `reading_sessions`

```
+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| session_id     | int     |
| book_id        | int     |
| reader_name    | varchar |
| pages_read     | int     |
| session_rating | int     |
+----------------+---------+
session_id is the unique ID for this table.
Each row represents a reading session where someone read a portion of a book. session_rating is on a scale of 1-5.
```

Write a solution to find books that have polarized opinions - books that receive both very high ratings and very low ratings from different readers.

* A book has polarized opinions if it has at least one rating ≥ 4 and at least one rating ≤ 2
* Only consider books that have at least 5 reading sessions
* Calculate the rating spread as (highest_rating - lowest_rating)
* Calculate the polarization score as the number of extreme ratings (ratings ≤ 2 or ≥ 4) divided by total sessions
* Only include books where polarization score ≥ 0.6 (at least 60% extreme ratings)

Return the result table ordered by polarization score in descending order, then by title in descending order.

The result format is in the following example.

 

Example:

```
Input:

books table:

+---------+------------------------+---------------+----------+-------+
| book_id | title                  | author        | genre    | pages |
+---------+------------------------+---------------+----------+-------+
| 1       | The Great Gatsby       | F. Scott      | Fiction  | 180   |
| 2       | To Kill a Mockingbird  | Harper Lee    | Fiction  | 281   |
| 3       | 1984                   | George Orwell | Dystopian| 328   |
| 4       | Pride and Prejudice    | Jane Austen   | Romance  | 432   |
| 5       | The Catcher in the Rye | J.D. Salinger | Fiction  | 277   |
+---------+------------------------+---------------+----------+-------+
reading_sessions table:

+------------+---------+-------------+------------+----------------+
| session_id | book_id | reader_name | pages_read | session_rating |
+------------+---------+-------------+------------+----------------+
| 1          | 1       | Alice       | 50         | 5              |
| 2          | 1       | Bob         | 60         | 1              |
| 3          | 1       | Carol       | 40         | 4              |
| 4          | 1       | David       | 30         | 2              |
| 5          | 1       | Emma        | 45         | 5              |
| 6          | 2       | Frank       | 80         | 4              |
| 7          | 2       | Grace       | 70         | 4              |
| 8          | 2       | Henry       | 90         | 5              |
| 9          | 2       | Ivy         | 60         | 4              |
| 10         | 2       | Jack        | 75         | 4              |
| 11         | 3       | Kate        | 100        | 2              |
| 12         | 3       | Liam        | 120        | 1              |
| 13         | 3       | Mia         | 80         | 2              |
| 14         | 3       | Noah        | 90         | 1              |
| 15         | 3       | Olivia      | 110        | 4              |
| 16         | 3       | Paul        | 95         | 5              |
| 17         | 4       | Quinn       | 150        | 3              |
| 18         | 4       | Ruby        | 140        | 3              |
| 19         | 5       | Sam         | 80         | 1              |
| 20         | 5       | Tara        | 70         | 2              |
+------------+---------+-------------+------------+----------------+
```

```
Output:

+---------+------------------+---------------+-----------+-------+---------------+--------------------+
| book_id | title            | author        | genre     | pages | rating_spread | polarization_score |
+---------+------------------+---------------+-----------+-------+---------------+--------------------+
| 1       | The Great Gatsby | F. Scott      | Fiction   | 180   | 4             | 1.00               |
| 3       | 1984             | George Orwell | Dystopian | 328   | 4             | 1.00               |
+---------+------------------+---------------+-----------+-------+---------------+--------------------+
```

# SQL

```sql
WITH book_stats AS (
    SELECT 
      book_id,
      COUNT(*) AS total_session,
      MIN(session_rating) AS minrating,
      MAX(session_rating) AS maxrating,
      SUM(CASE WHEN session_rating > 3 THEN 1 ELSE 0 END) AS highrating,
      SUM(CASE WHEN session_rating < 3 THEN 1 ELSE 0 END) AS lowrating,
      SUM(CASE WHEN session_rating > 3 OR session_rating < 3 THEN 1 ELSE 0 END) AS extremes
    FROM 
      reading_sessions 
    GROUP BY 
      1
)

SELECT 
  book_id,
  title,
  author,
  genre,
  pages,
  maxrating - minrating AS rating_spread,
  ROUND(extremes * 1.0 / total_session, 2) AS polarization_score 
FROM 
  book_stats
JOIN 
  books
USING(book_id)
WHERE 
  total_session > 4 AND 
  highrating > 0 AND 
  lowrating > 0
HAVING 
  polarization_score >= 0.6
ORDER BY 
  polarization_score DESC,
  title DESC;
```

# Python

```python
def find_polarized_books(books: pd.DataFrame, reading_sessions: pd.DataFrame) -> pd.DataFrame:

    # 1. Fix for leetcode incorrect rounding in pandas
    round2 = lambda x: round(x + 0.00001, 2)

    # 2. Determine the necessary counts and max/min per book_id
    df = reading_sessions.groupby('book_id').agg(
            three_count = ('session_rating', lambda x: (x == 3).sum()),
            total_count = ('session_rating', 'count'),
            max_score   = ('session_rating', 'max'),
            min_score   = ('session_rating', 'min')).reset_index()

    # 3. Compute the required data per book_id
    df['rating_spread'] = df.max_score - df.min_score
    df['polarization_score'] = round2((df.total_count - df.three_count)/df.total_count)
    
    # 4. Filter
    df =  df[(df.min_score < 3) & (df.max_score > 3) & 
             (df.total_count >= 5) & (df.polarization_score >= 0.6)]

    # 5. Sort rows and edit columns
    return df.merge(books).sort_values(['polarization_score', 'title' ], 
                                        ascending = [0,0]).iloc[:,[0,7,8,9,10,5,6]]
```
