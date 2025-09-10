# [3586. Find COVID Recovery Patients](https://leetcode.com/problems/find-covid-recovery-patients/)

* [SQL Solution](https://leetcode.com/problems/find-covid-recovery-patients/solutions/7174031/using-self-join-by-atamalu123-km52/)
* [Python Solution](https://leetcode.com/problems/find-covid-recovery-patients/solutions/7174037/pandas-solution-by-atamalu123-6ppb/)

# Problem

Table: `patients`

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| patient_id  | int     |
| patient_name| varchar |
| age         | int     |
+-------------+---------+
patient_id is the unique identifier for this table.
Each row contains information about a patient.
```

Table: `covid_tests`

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| test_id     | int     |
| patient_id  | int     |
| test_date   | date    |
| result      | varchar |
+-------------+---------+
test_id is the unique identifier for this table.
Each row represents a COVID test result. The result can be Positive, Negative, or Inconclusive.
```

Write a solution to find patients who have recovered from COVID - patients who tested positive but later tested negative.

A patient is considered recovered if they have at least one Positive test followed by at least one Negative test on a later date
Calculate the recovery time in days as the difference between the first positive test and the first negative test after that positive test
Only include patients who have both positive and negative test results
Return the result table ordered by recovery_time in ascending order, then by patient_name in ascending order.

The result format is in the following example.

 

Example:

```
Input:

patients table:

+------------+--------------+-----+
| patient_id | patient_name | age |
+------------+--------------+-----+
| 1          | Alice Smith  | 28  |
| 2          | Bob Johnson  | 35  |
| 3          | Carol Davis  | 42  |
| 4          | David Wilson | 31  |
| 5          | Emma Brown   | 29  |
+------------+--------------+-----+
covid_tests table:

+---------+------------+------------+--------------+
| test_id | patient_id | test_date  | result       |
+---------+------------+------------+--------------+
| 1       | 1          | 2023-01-15 | Positive     |
| 2       | 1          | 2023-01-25 | Negative     |
| 3       | 2          | 2023-02-01 | Positive     |
| 4       | 2          | 2023-02-05 | Inconclusive |
| 5       | 2          | 2023-02-12 | Negative     |
| 6       | 3          | 2023-01-20 | Negative     |
| 7       | 3          | 2023-02-10 | Positive     |
| 8       | 3          | 2023-02-20 | Negative     |
| 9       | 4          | 2023-01-10 | Positive     |
| 10      | 4          | 2023-01-18 | Positive     |
| 11      | 5          | 2023-02-15 | Negative     |
| 12      | 5          | 2023-02-20 | Negative     |
+---------+------------+------------+--------------+
```

```
Output:

+------------+--------------+-----+---------------+
| patient_id | patient_name | age | recovery_time |
+------------+--------------+-----+---------------+
| 1          | Alice Smith  | 28  | 10            |
| 3          | Carol Davis  | 42  | 10            |
| 2          | Bob Johnson  | 35  | 11            |
+------------+--------------+-----+---------------+
```

# SQL

```sql
SELECT 
  c.patient_id, 
  p.patient_name, 
  p.age, 
  DATEDIFF(MIN(c1.test_date), MIN(c.test_date)) AS recovery_time
FROM 
  covid_tests c
INNER JOIN 
  covid_tests c1 
  ON c.patient_id = c1.patient_id AND 
  c.test_date < c1.test_date AND 
  c.result = 'Positive' AND c1.result = 'Negative'
INNER JOIN 
  patients p 
  ON c.patient_id = p.patient_id
GROUP BY 
  c.patient_id, 
  p.patient_name, 
  p.age
ORDER BY 
  recovery_time, 
  p.patient_name;
```

# Python

```python
def find_covid_recovery_patients(patients: pd.DataFrame, 
                                 covid_tests: pd.DataFrame) -> pd.DataFrame:
  
    # 1. Determine first positive per patient_id
    frstPos = (covid_tests[covid_tests.result == 'Positive']
        .groupby('patient_id').agg( Pos_date = ('test_date', 'min')).reset_index())
    df = covid_tests.merge(frstPos)

    # 2. Determine first negative test after first positive test
    frstNeg = (df[(df.test_date > df.Pos_date) & (df.result == 'Negative')]
        .groupby('patient_id').agg(Neg_date = ('test_date', 'min')).reset_index())
    df = frstPos.merge(frstNeg).dropna()

    # 3. Compute recovery time
    df['recovery_time'] = (pd.to_datetime(df.Neg_date) -
                           pd.to_datetime(df.Pos_date)).dt.days

    # 4. Merge patient info, sort rows and rearrange columns
    df = df.merge(patients).sort_values(['recovery_time', 'patient_name'])
    return df.iloc[:,[0,4,5,3]]
```
