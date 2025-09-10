# [3642. Find Books with Polarized Opinions](https://leetcode.com/problems/find-books-with-polarized-opinions/)

* [SQL Solution](https://leetcode.com/problems/find-books-with-polarized-opinions/solutions/7174266/using-cte-by-atamalu123-ynem/)
* [Python Solution](https://leetcode.com/problems/find-books-with-polarized-opinions/solutions/7174278/pandas-solution-by-atamalu123-3h04/)

# Problem

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
