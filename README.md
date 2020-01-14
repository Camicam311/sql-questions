# SQL questions
----
### Nth Highest Salary
Write a SQL query to get the nth highest salary from the `Employee` tabble.

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
	SELECT DISTINCT Salary # we want to get only a single value if there are more than one same value
	FROM Employee
	ORDER BY Salary DESC # order by descending order to get the highest 
	LIMIT M, 1 # same as LIMIT 1 OFFSET M (Show first value after disregarding first M entries)
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
### Company Query
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
### Earnings by country
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
