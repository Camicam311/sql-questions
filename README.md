# SQL questions
### 1.Company Query
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
 ---
