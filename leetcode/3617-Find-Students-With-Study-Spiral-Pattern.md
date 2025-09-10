# [3617. Find Students with Study Spiral Pattern](https://leetcode.com/problems/find-students-with-study-spiral-pattern/)

* [SQL Solution](https://leetcode.com/problems/find-students-with-study-spiral-pattern/solutions/7174302/ctes-subqueries-by-atamalu123-fxi7/)
* [Python Solution](https://leetcode.com/problems/find-students-with-study-spiral-pattern/solutions/7174314/pandas-solution-by-atamalu123-xiz3/)

# Problem

# SQL

```sql
WITH three_unique_subject_2_times AS (
    SELECT 
      student_id 
    FROM 
      study_sessions 
    GROUP BY 
      student_id 
    HAVING 
      COUNT(DISTINCT(subject)) > 2 AND 
      student_id NOT IN (
        SELECT 
          DISTINCT(student_id) 
        FROM 
          study_sessions 
        GROUP BY 
          student_id,
          subject 
        HAVING COUNT(*) < 2
        )
), consecutive_dates AS (
    SELECT 
      student_id
    FROM (
      SELECT 
        student_id,
        subject,
        DATEDIFF(session_date, LAG(session_date, 1) OVER (PARTITION BY student_id ORDER BY session_date)) AS gaps
        FROM 
          study_sessions
    ) AS consecutive_dates
    GROUP BY 
      student_id
    HAVING 
      MAX(gaps) < 3
), study_spiral_pattern AS (
    SELECT 
      s.student_id,
      s.student_name,
      s.major, 
      COUNT(DISTINCT(ss.subject)) AS cycle_length,
      SUM(ss.hours_studied) AS total_study_hours,
      COUNT(s.student_id) OVER() AS unique_students
    FROM 
      students AS s
    JOIN 
      study_sessions AS ss
    ON 
      s.student_id = ss.student_id
    GROUP BY 
      s.student_id
    HAVING 
      s.student_id IN (SELECT * FROM three_unique_subject_2_times) AND 
      s.student_id IN (SELECT * FROM consecutive_dates)
    ORDER BY
      cycle_length DESC, 
      total_study_hours DESC
)

SELECT 
  student_id,
  student_name,
  major,
  cycle_length,
  total_study_hours
FROM 
  study_spiral_pattern
WHERE 
  unique_students > 1;
```

# Python

```python
def find_study_spiral_pattern(students: pd.DataFrame, study_sessions: pd.DataFrame) -> pd.DataFrame:
    #
    ss = study_sessions.copy()
    ss['session_date'] = pd.to_datetime(ss['session_date'])
    ss['hours_studied'] = pd.to_numeric(ss['hours_studied'])

    #
    ssc = ss.groupby(['student_id', 'subject']).size().reset_index(name='cnt')
    s_stats = ssc.groupby('student_id').agg(subject_count=('subject', 'nunique'), min_cnt=('cnt', 'min')).reset_index()

    #
    ids1 = set(s_stats[(s_stats['subject_count'] > 2) & (s_stats['min_cnt'] > 1)]['student_id'].tolist())
    
    #
    ss_sorted = ss.sort_values(['student_id', 'session_date'])
    ss_sorted['prev_date'] = ss_sorted.groupby('student_id')['session_date'].shift(1)
    ss_sorted['day_gap'] = (ss_sorted['session_date'] - ss_sorted['prev_date']).dt.days

    #
    max_gap = ss_sorted.groupby('student_id')['day_gap'].max().fillna(0).reset_index()
    ids2 = set(max_gap[max_gap['day_gap'] < 3]['student_id'].tolist())
    qual_ids = ids1 & ids2

    #
    if not qual_ids:
        return pd.DataFrame(columns=['student_id', 'student_name', 'major', 'cycle_length', 'total_study_hours'])

    #
    res = ss[ss['student_id'].isin(qual_ids)].groupby('student_id').agg(cycle_length=('subject', 'nunique'), total_study_hours=('hours_studied', 'sum')).reset_index()
    res = res.merge(students[['student_id', 'student_name', 'major']], on='student_id', how='left')

    #
    if len(res) <= 1:
        return pd.DataFrame(columns=['student_id', 'student_name', 'major', 'cycle_length', 'total_study_hours'])

    #
    res = res[['student_id', 'student_name', 'major', 'cycle_length', 'total_study_hours']].sort_values(['cycle_length', 'total_study_hours'], ascending=[False, False]).reset_index(drop=True)
    
    return res
```
