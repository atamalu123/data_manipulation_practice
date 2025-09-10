# [3586. Find COVID Recovery Patients](https://leetcode.com/problems/find-covid-recovery-patients/)

* [SQL Solution](https://leetcode.com/problems/find-covid-recovery-patients/solutions/7174031/using-self-join-by-atamalu123-km52/)
* [Python Solution](https://leetcode.com/problems/find-covid-recovery-patients/solutions/7174037/pandas-solution-by-atamalu123-6ppb/)

# Problem

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
