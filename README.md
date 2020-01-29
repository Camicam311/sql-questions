# SQL questions
---
## Exchange Seats
Mary is a teacher in a middle school and she has a table seat storing students' names and their corresponding seat ids.

The column id is continuous increment.


Mary wants to change seats for the adjacent students.


Write a SQL query to output the result for Mary.

**Note**: If the number of students is odd, there is no need to change the last one's seat.

### `Seat` table
|Id|student|
--|--
1|Abbot
2|Doris
3|Emerson
4|Green
5|James

### Output:
|Id|Student|
--|--
1|Doris
2|Abbot
3|Green
4|Emerson
5|James

### Answer
```sql
SELECT
  (CASE
      -- considering when count is odd
      WHEN mod(id,2) != 0 AND id != counts THEN id + 1
      WHEN mod(id,2) != 0 AND id = counts THEN id
      -- considering when count is even
      ELSE id - 1
    END) AS id,
  student
FROM seat, (SELECT COUNT(*) AS counts FROM seat) AS seat_counts
ORDER BY id ASC
;
```
### Explanation
1. First we want to count number of students to see whether the count is odd or even.
```sql
(SELECT COUNT(*) AS counts FROM seat) AS seat_counts
```
2. Then we want to divide into different cases:
```sql
(CASE
      -- case when id is odd and total number is not equal
      WHEN mod(id,2) != 0 AND id != counts THEN id + 1
      -- case when id is odd and total number is equal
      WHEN mod(id,2) != 0 AND id = counts THEN id
      -- case when id is even
      ELSE id - 1
END)
```

<a href="#top">Back to top</a>

---
## Department Highest Salary
The `Employee` table holds all employees. Every employee has an id, salary and there is also a column for the department id.

#### `Employee` table:
|Id|Name|Salary|DepartmentId|
--|--|--|--
1|Joe|70000|1
2|Jim|90000|1
3|Henry|80000|2
4|Sam|60000|2
5|Max|90000|1

#### `Department` table holds all departments of the company:
|Id|Name|
--|--
1|IT
2|Sales

Write a SQL query to find employees who have the highest salary in each of the departments. For the above tables, your SQL query should return the following rows (order of rows does not matter).

#### Sample output:
|Department|Employee|Salary|
--|--|--
IT|Max|90000
IT|Jim|90000
Sales|Henry|80000

Max and Jim both have the highest salary in the IT department and Henry has the highest salary in the Sales department.

#### Answer:
```sql
SELECT d.Name Department, e.Name Employee e.Salary Salary
FROM Employee e
INNER JOIN Department d on e.DepartmentId = d.Id
WHERE (e.DepartmentId, e.Salary)
IN
  (SELECT DepartmentId, MAX(Salary)
  FROM Employee
  GROUP BY DepartmentId)
;
```
#### Explanation:
1. Note we want to first create a table that lists out the maximum salaries for each of the department.
```sql
(SELECT DepartmentId, MAX(Salary)
FROM Employee
GROUP BY DepartmentId)
```
2. Afterwards, we make a query that outputs all employees that are in the table that we created in *step 1*.
```SQL
WHERE (e.DepartmentId, e.Salary) IN
  (SELECT DepartmentId, MAX(Salary)
  FROM Employee
  GROUP BY DepartmentId)
```

<a href="#top">Back to top</a>

---
## Consecutive Numbers
Write a SQL query to find all numbers that appear at least three times consecutively.

#### Logs table:
|Id|Num|
--|--
1|1
2|1
3|1
4|2
5|1
6|2
7|2

#### Sample output:
For example, given the above Logs table, 1 is the only number that appears consecutively for at least three times.

|ConsecutiveNums|
---
|1|

#### Answer:
```SQL
SELECT l1.Num as ConsecutiveNums
FROM logs l1
LEFT JOIN logs l2 on l1.Id = l2.Id -1
LEFT JOIN logs l3 on l1.Id = l3.Id -2
WHERE l1.Num = l2.Num
AND l1.Num = l3.Num
```
#### Explanation:
1. We want to align two tables along side the `Logs` table such that the second table next to `Logs` table will start from the second row of the `Logs` table. The following is an example:

|Id|Num|Id|Num
--|--|--|--
1|1|2|1
2|1|3|1
...|...|...|...|
7|2|null|null

```SQL
LEFT JOIN logs l2 on l1.Id = l2.Id -1
```


2. Similarly, the third table will look like the following:

|Id|Num|Id|Num|Id|Num
--|--|--|--|--|--
1|1|2|1|3|1
2|1|3|1|4|2
...|...|...|...|...|...|
7|2|null|null|null|null

```SQL
LEFT JOIN logs l3 on l1.Id = l3.Id -2
```

3. After we have the three tables lined up, we filter by conditioning by the following:

```SQL
WHERE l1.Num = l2.Num
AND l1.Num = l3.Num
```

<a href="#top">Back to top</a>

---
## Active users retention
Assume you have the below tables on user actions. Write a query to get the active user retention by month.

#### user_actions table:
|column name|type|
--|--
user_id|integer
event_id|string
timestamp|datetime

```SQL
-- DATETIME - format: YYYY-MM-DD HH:MI:SS
SELECT EXTRACT(MONTH FROM timestamp) as month, SUM(DISTINCT user_id)
FROM user_actions
GROUP BY month
```
<a href="#top">Back to top</a>

---
## Rank Scores
Write a SQL query to rank scores. If there is a tie between two scores, both should have the same ranking. Note that after a tie, the next ranking number should be the next consecutive integer value. In other words, there should be no "holes" between ranks.

#### Scores table:
|Id|Score|
---|---
| 1  | 3.50  |
| 2  | 3.65  |
| 3  | 4.00  |
| 4  | 3.85  |
| 5  | 4.00  |
| 6  | 3.65  |

#### Output:
For example, given the above Scores table, your query should generate the following report (order by highest score):

|Score|Rank|
---|---
|4.00   | 1    |
| 4.00  | 1    |
| 3.85  | 2    |
| 3.65  | 3    |
| 3.65  | 3    |
| 3.50  | 4    |

#### Answer 1:
```SQL
SELECT
  Score,
-- We first count the number of distinct scores that are greater than
-- or equal to itself
  (SELECT COUNT(DISTINCT Score) FROM Scores WHERE Score >= s.Score) Rank
FROM Scores s
-- Make sure to order scores in decreasing order
ORDER BY Score DESC
```
#### Explanation
- We want to create a column called `Rank` that ranks the scores in decreasing order.
- We can do this by first selecting the `Score` column with only <strong>DISTINCT</strong> entries.
- To turn this into the `Rank` column that we want, we <strong>COUNT</strong> the number of distinct Scores.
- Now our next issue is to deal with repeated entries from `Score` column, i.e. what if you have more than two scores?
- We can deal with this issue by adding `WHERE Score >= s.Score`.

#### Answer 2:
```SQL
SELECT
  Score,
  @x := @x + (@y <> (@y := Score)) Rank
FROM
  Scores,
  (SELECT @x := 0, @y := -1) init
ORDER BY Score DESC
```
#### Explanation
- We want to create a column called `Rank` that indicates the rank of a score.

```SQL
ORDER BY Score DESC
```
- We will make a table that includes two variables that start from `0` and `-1` accordingly. Let the variables be `x`,`y` accordingly.

```SQL
SELECT @x := 0, @y := -1)
```
- Then we will order `Scores` table in <strong>DECREASING</strong> order by score to start counting the ranks.
- As we move down the `score` column, we will add `1` to `x` and add `0` if the scores are the same.

```SQL
-- rank = rank + (0 if prev == Score else 1) // set prev = Score at the same time
@x := @x + (@y <> (@y := Score))
```

<a href="#top">Back to top</a>

----
## Nth Highest Salary
Write a SQL query to get the nth highest salary from the `Employee` table.

#### Employee table:
|Id|Salary|
---|---
|1|100|
|2|200|
|3|300|

For example, given the above table, the nth highest salary where n = 2 is `200`. If there is no highest salary, output `null`.

#### Output table:
|getHighestSalary(2)|
|---|
|200|

#### Answer:
```sql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
DECLARE M int;
SET M = N - 1;
RETURN (
	SELECT DISTINCT Salary -- we want to get only a single value if there are more than one same value
	FROM Employee
	ORDER BY Salary DESC -- order by descending order to get the highest
	LIMIT M, 1 -- same as LIMIT 1 OFFSET M (Show first value after disregarding first M entries)
);
END
```
#### Explanation:
- We want to first order the `Salary` column in <strong>descending</strong> order and also not to forget to query only <strong> DISTINCT</strong> values.
- Intuitively, if we want to get the nth highest salary, this means that we will have to count down from the highest `Salary` to the nth highest salary.
- For example, if we want to get the 4th highest salary, we will skip the first entries from the `Salary` column and stop at the 4th entry.
- To translate this process into a query, we will make use of `LIMIT A, OFFSET B`. Essentially `LIMIT A, OFFSET B` means that we will skip the first `B` amount of entries and only show the next `A` entries.
- In our case, we will skip the first <strong>N-1</strong> entries and how only the <strong>N</strong>th entry (i.e. `LIMIT 1, OFFSET N-1 OR LIMIT N-1, 1`
- So we start by initializing the variable `M` as `SET M = N-1;` to set the number of entries to skip until the `N`th highest salary.

Steps in summary:
1. Initialize `SET M = N - 1;`
2. Query `Salary` in <strong>descending</strong> order, not forgetting to display only <strong>distinct</strong> values.
3. Skip the first `N-1` entries and show the `N`th entry, i.e. `LIMIT M,1`

<a href="#top">Back to top</a>

----
## Company Query
Given two tables, query out names of people and the names of their previous employers. Limit the list to the people currently working with the companies which were left by the most number of people. Print the name of the employee and the previous employer.

#### People table:

|Name|Type|Description|
---|---|---
ID|STRING|ID of the employee
NAME|STRING|Name of the employee
PREV_COMPANY_ID|STR|ID of the previous company
CUR_COMPANY_ID|STR|ID of the current company

#### Companies table:

|Name|Type|Description|
---|---|---
ID|STRING|ID of the company
NAME|STRING|Name of the company

#### Sample Input: People table

|ID|NAME|CUR_COMPANY_ID|PREV_COMPANY_ID
---|---|---|---
1|Chris Michael|345|123
2|Sandra Park|567|234
3|Ashley Gibon|456|234
4|Matthew Lopez|456|345
5|Pattrck Heinz|234|345
6|Alex Arnolds|123|345
7|Helen Smith|567|456
8|Louisa Sanchez|345|456
9|Clark Henderson|123|456
10|Clara Mayon|123|456


#### Sample Input: Companies table

|ID|NAME|
---|---|
123|Ann-Sullivan
234|Harmon Kardon
345|Smith-McKinsey
456|Google
567|Facebook

#### Sample Output:
```sql
Ashley Gibon Google
Matthew Lopez Google
```
<br/><br/>
#### Answer:
```sql
SELECT tbl2.p_name,tbl2.c_name
FROM
-- query to rank employers with the most number of employees left
	(SELECT c.name c_name, count(p.name) cnt
	FROM people p, companies c
	WHERE p.prev_company_id = c.id
	GROUP BY c.name
	ORDER BY cnt desc
	LIMIT 1) tbl1,
-- join people table with companies in terms of p.cur_company_id = c.id
	(SELECT p.name p_name, c.name c_name
	FROM people p, companies c
	WHERE p.cur_company_id = c.id) tbl2
WHERE tbl1.c_name = tbl2.c_name;
```
#### Explanation
We first divide the query into 3 different parts.
1. We first join <strong>PEOPLE</strong> and <strong>COMPANIES</strong> table by <strong>PREV_COMPANY_ID</strong> to see employees and their previous company names.
2. Then we <strong>GROUP BY</strong> the company names and <strong>COUNT</strong> the number of employees for each companies to <strong>OBTAIN THE NUMBER OF PREVIOUS EMPLOYEES FOR EACH COMPANY</strong>.
3. Make sure to <strong>ORDER BY</strong> the employee count in <strong>DESC</strong>ending order.
4. We <strong>LIMIT</strong> by 1 to only show the <strong>MAX</strong>imum count.
```sql
(SELECT c.name c_name, count(p.name) cnt
FROM people p, companies c
WHERE p.prev_company_id = c.id
GROUP BY c_name
ORDER BY cnt desc
LIMIT 1) tbl1
```
Then, we want to match <strong>CURRENT</strong> company with the result we have from `tbl1`. So we query the employee name and company name joined by `p.cur_company_id = c.id`.
```sql
(SELECT p.name p_name, c.name c_name
FROM people p, companies c
WHERE p.cur_company_id = c.id) tbl2
;
```
Lastly, we will join the two tables, `tbl1` and `tbl2` based on `tbl1.c_name = tbl2.c_name`!

We can also see below that the company that had the most workers leaving was `Google`.

|ID|NAME|CUR_COMPANY_ID|PREV_COMPANY_ID
---|---|---|---
1|Chris Michael|Smith-McKinsey|Ann-Sullivan
2|Sandra Park|Facebook|Harmon Kardon
3|Ashley Gibon|Google|Harmon Kardon
4|Matthew Lopez|Google|Smith-McKinsey
5|Pattrck Heinz|Harmon Kardon|Smith-McKinsey
6|Alex Arnolds|Ann-Sullivan|Smith-Mckinsey
7|Helen Smith|Facbook|Google
8|Louisa Sanchez|Smith-McKinsey|Google
9|Clark Henderson|Ann-Sullivan|Google
10|Clara Mayon|Ann-Sullivan|Google

<a href="#top">Back to top</a>

----
## Earnings by country
Write a query to get the city names and earnings from each city. 'Earnings' are the sum of all the fares from the rides for a given city. Please display the output as the following:
'CITIES.Name EARNINGS'
Sort the output according the earnings in ascending order and city names in ascending order.

We are given three different tables below
#### CITIES:
|Name|Type|Description|
---|---|---
ID|STR|Id of the city
Name|STR|Name of the city

#### USERS:
|Name|Type|Description
---|---|---
ID|STR|User ID
city_id|STR|City Id
name|STR|User Name
email|STR|User email

#### RIDES:
|Name|Type|Description
---|---|---
id|STR|Ride Id
user_id|STR|User ID
distance|INT|Distance traveled
fare|INT|Fare of the ride

#### Sample Input:CITIES
|id|Name|
---|---
1|San Francisco
2|Columbia

#### Sample Input:USERS
|id|city_id|name|email
---|---|---|---
1|2|Roberto Carlos| rc@gmail.com
2|2|Tom Hardy|th@gmail.com
3|1|Jordan Peters|jp@gmail.com
4|1|Bill Gait|bg@gmail.com
5|1|Frank Ribery|frdf2@gmail.com
6|1|Morgan John|mr34@gmail.com

#### Sample Input: RIDES
|id|user_id|distance|fare
---|---|---|---
1|1|21|200
2|3|6|55
3|2|30|230
4|1|21|300
5|2|1234|320
6|4|4352|1000
7|5|43652|300
8|6|343|355

#### Sample Output:
```sql
Columbia 1140
San Francisco 1710
```
<br/><br/>
#### Answer:
```sql
SELECT c.name, SUM(ur.fare) earnings
FROM cities c,
	(SELECT u.city_id,u.name, r.fare
	FROM users u, rides r
	WHERE u.id = r.user_id) ur
WHERE ur.city_id = c.id
GROUP BY c.id,c.name
ORDER BY earnings ASC, c.name ASC;
```

#### Explanation
1. First we want to join `Users` and `Rides` tables together by `u.id = r.user_id` to get ride informatino of the users.
2. Then we join the table from step 1 with `cities` table based on `city_id`.
3. We will group by `cities.id` and `cities.name` and <strong>SUM</strong> fares from the table from step 1.
4. Make sure to order `earnings` and `cities.name` in ascending order


 <a href="#top">Back to top</a>

 ---
