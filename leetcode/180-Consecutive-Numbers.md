# [180. Consecutive Numbers](https://leetcode.com/problems/consecutive-numbers/)

* [SQL Solution](https://leetcode.com/problems/consecutive-numbers/solutions/7167324/moving-window-by-atamalu123-uejn/)
* [Python Solution](https://leetcode.com/problems/consecutive-numbers/solutions/7167391/sliding-window-hash-set-by-atamalu123-3p5f/)

# Problem

Table: `Logs`

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| num         | varchar |
+-------------+---------+
In SQL, id is the primary key for this table.
id is an autoincrement column starting from 1.
```

Find all numbers that appear at least three times consecutively.

Return the result table in any order.

The result format is in the following example.
 
Example 1:

```
Input: 
Logs table:
+----+-----+
| id | num |
+----+-----+
| 1  | 1   |
| 2  | 1   |
| 3  | 1   |
| 4  | 2   |
| 5  | 1   |
| 6  | 2   |
| 7  | 2   |
+----+-----+
```
```
Output: 
+-----------------+
| ConsecutiveNums |
+-----------------+
| 1               |
+-----------------+
```
Explanation: 1 is the only number that appears consecutively for at least three times.
