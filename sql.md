180. Consecutive Numbers
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
连续三个相同的数字
选三个表 然后连等

```sql
select distinct(l1.num) as ConsecutiveNums
from logs l1, logs l2, logs l3
where l1.num=l2.num and l1.num=l3.num and l2.id=l1.id+1 and l3.id=l2.id+1
```

lead lag 函数
lead函数 (结果包含null  外面陶一层子查询 去掉NULL)
```sql
    SELECT 
		DISTINCT CASE 
		WHEN num = LEAD(num, 1) OVER() AND num = LEAD(num, 2) OVER() THEN num 
		END AS ConsecutiveNums
	FROM logs
```


181. Employees Earning More Than Their Managers


# Write your MySQL query statement below
自连接
```sql
select a.name as "Employee" from Employee a inner join Employee b on a.managerId = b.id and a.salary > b.salary
```

子查询 
#####  父查询的条件可以带到子查询里面去

```sql
Select name as Employee from Employee e1 where 
salary > (select salary from Employee where id=e1.managerId)
```




in 和exist 的区别 exist 是loop 子查询能用索引  in 是hash 连接 先连接  子查询用不到索引
子查询小的用in  大的用exist
子查询  优化 没啥区别




184. Department Highest Salary
按照部门分类 , 每个部门最大的salary 和那个最大的salary 关联的职工名以及职工其他信息
如果用group by 那么 每个部门都只会返回一条数据, 而且这条数据没法关联到那个最大的

max over 这个查询的当前字段只跟 当前partition有关 和其他字段无关,每条都会返回 然后附上当前所属的部门的最大薪水

```sql
select Department, Employee,Salary
from 
(
    select  a.name as "Employee" , a.salary as Salary,  b.name  as "Department", max(salary) over(partition by departmentId) as max_salary 
        from Employee a inner join Department b 
            on a.departmentId = b.id
)c 
where Salary = max_salary
```

185. Department Top Three Salaries
分组top n

partition by departmentId order by salary desc




```sql
select Department,Employee,Salary from (
    select a.name as "Employee" ,salary as Salary, b.name as "Department",  
        dense_rank() over(partition by departmentId order by salary desc) as r  
            from Employee a inner join Department b 
                on a.departmentId = b.id

)c 
    where r <= 3 
        order by Department asc, Salary desc

```

row_number 严格标号 
rank 同样的数字排名 同样的标号
denserank 下一个排名是上一个+1 rank 是严格第几名


196. Delete Duplicate Emails
Input: 
Person table:
+----+------------------+
| id | email            |
+----+------------------+
| 1  | john@example.com |
| 2  | bob@example.com  |
| 3  | john@example.com |
+----+------------------+
Output: 
+----+------------------+
| id | email            |
+----+------------------+
| 1  | john@example.com |
| 2  | bob@example.com  |
+----+------------------+


mysql 不能直接在表里查询然后在表里删除
要套一层临时表
```sql
delete from Person where id not in (
    select id from (
        select min(id) as id from Person group by email
    )a
)
```


197. Rising Temperature
找出温度增加的日期的id

###     DATE_ADD(a.recordDate , interval 1 day) 重要

Weather table:
+----+------------+-------------+
| id | recordDate | temperature |
+----+------------+-------------+
| 1  | 2015-01-01 | 10          |
| 2  | 2015-01-02 | 25          |
| 3  | 2015-01-03 | 20          |
| 4  | 2015-01-04 | 30          |
+----+------------+-------------+
Output: 
+----+
| id |
+----+
| 2  |
| 4  |
+----+



```sql
select b.id from Weather a inner join Weather b on DATE_ADD(a.recordDate , interval 1 day) = b.recordDate 
and a.temperature < b.temperature
```



忘了可以用lag 或者lead

```py
select id 
from (  select id,
    recordDate,
    temperature,
    lag(recordDate,1,'None') over() as prev_day,
    lag(temperature,1,'None') over() as prev_temp
    from weather
    order by recordDate
    ) a
where recordDate = date_add(prev_day,interval 1 day) and
temperature > prev_temp
```



262. Trips and Users


Trips table:
+----+-----------+-----------+---------+---------------------+------------+
| id | client_id | driver_id | city_id | status              | request_at |
+----+-----------+-----------+---------+---------------------+------------+
| 1  | 1         | 10        | 1       | completed           | 2013-10-01 |
| 2  | 2         | 11        | 1       | cancelled_by_driver | 2013-10-01 |
| 3  | 3         | 12        | 6       | completed           | 2013-10-01 |
| 4  | 4         | 13        | 6       | cancelled_by_client | 2013-10-01 |
| 5  | 1         | 10        | 1       | completed           | 2013-10-02 |
| 6  | 2         | 11        | 6       | completed           | 2013-10-02 |
| 7  | 3         | 12        | 6       | completed           | 2013-10-02 |
| 8  | 2         | 12        | 12      | completed           | 2013-10-03 |
| 9  | 3         | 10        | 12      | completed           | 2013-10-03 |
| 10 | 4         | 13        | 12      | cancelled_by_driver | 2013-10-03 |
+----+-----------+-----------+---------+---------------------+------------+
Users table:
+----------+--------+--------+
| users_id | banned | role   |
+----------+--------+--------+
| 1        | No     | client |
| 2        | Yes    | client |
| 3        | No     | client |
| 4        | No     | client |
| 10       | No     | driver |
| 11       | No     | driver |
| 12       | No     | driver |
| 13       | No     | driver |
+----------+--------+--------+
Output: 
+------------+-------------------+
| Day        | Cancellation Rate |
+------------+-------------------+
| 2013-10-01 | 0.33              |
| 2013-10-02 | 0.00              |
| 2013-10-03 | 0.50              |
+------------+-------------------+



两个表 关于打车的,打车标有的订单是被取消的. 算出取消的订单是这一天总订单的比率 用户表被禁的用户不能计算 且按日按日分组

group by 就可以


count/sum 带条件
补充其他表的信息不要想着用join
```sql
select  request_at as Day,round(count(if(status!='completed',id,NULL)) / count(1),2) as "Cancellation Rate"
    from Trips 
    where  client_id not in (select users_id from users where banned = 'Yes')
    and driver_id not in (select users_id from users where banned = 'Yes')
    and request_at between '2013-10-01' and '2013-10-03'
    group by request_at
```



512. Game Play Analysis II
直接做出来的 不用复习

```sql
select a.player_id , a.device_id from Activity a 
    inner join 
    (select player_id , min(event_date) event_date  from Activity group by player_id) b 
on a.player_id = b.player_id and a.event_date =b.event_date
```


534. Game Play Analysis III

Activity table:
+-----------+-----------+------------+--------------+
| player_id | device_id | event_date | games_played |
+-----------+-----------+------------+--------------+
| 1         | 2         | 2016-03-01 | 5            |
| 1         | 2         | 2016-05-02 | 6            |
| 1         | 3         | 2017-06-25 | 1            |
| 3         | 1         | 2016-03-02 | 0            |
| 3         | 4         | 2018-07-03 | 5            |
+-----------+-----------+------------+--------------+
Output: 
+-----------+------------+---------------------+
| player_id | event_date | games_played_so_far |
+-----------+------------+---------------------+
| 1         | 2016-03-01 | 5                   |
| 1         | 2016-05-02 | 11                  |
| 1         | 2017-06-25 | 12                  |
| 3         | 2016-03-02 | 0                   |
| 3         | 2018-07-03 | 5                   |
+-----------+------------+---------------------+

输出每个player 截止到某一天共玩了多少句游戏

自连接自己做出来的:
```sql
select a.player_id, a.event_date,sum(b.games_played) as games_played_so_far  
    from  
    Activity a inner join Activity b 
        on a.player_id = b.player_id  and a.event_date >= b.event_date 
            group by a.player_id,a.event_date 
            order  by a.event_date ,a.player_id
```



sum over 又没想到
```sql
SELECT player_id, event_date,
SUM(games_played) OVER(PARTITION BY player_id ORDER BY event_date) AS games_played_so_far
FROM Activity
```


注意 over (order by) 如果不加order by  就是全部的 否则就是递增到当前这一行




550. Game Play Analysis IV

所有连续两天登陆的用户占总用户的比值

Activity table:
+-----------+-----------+------------+--------------+
| player_id | device_id | event_date | games_played |
+-----------+-----------+------------+--------------+
| 1         | 2         | 2016-03-01 | 5            |
| 1         | 2         | 2016-03-02 | 6            |
| 2         | 3         | 2017-06-25 | 1            |
| 3         | 1         | 2016-03-02 | 0            |
| 3         | 4         | 2018-07-03 | 5            |
+-----------+-----------+------------+--------------+
Output: 
+-----------+
| fraction  |
+-----------+
| 0.33      |
+-----------+



1. 有的用户只有一条数据, 只用lag 不行, 还要标号
2. 用with 做视图简化写法


```sql
# Write your MySQL query statement below
with temp as(
select *,lag(event_date) over(order by player_id, event_date) as p ,
     row_number() over(partition by player_id order by player_id, event_date) as 'rn' 
    from Activity 
)
select round( 
    (select count(distinct player_id) from temp where datediff(event_date,p)=1  and rn=2  )/
    (select count(distinct player_id) from Activity) 
,2) 
as  fraction
```



569. Median Employee Salary
求每个公司薪资的中位数
中位数 奇数和偶数不一样
*2 然后-  , 再0~2内 就是中位数

with temp as(
select * , row_number() over(partition by company order by salary) as no, count(1) over(partition by company)  as total from Employee 
)

select id ,company ,salary from temp where (cast(no*2 as signed)- cast(total as signed) ) between 0 and 2

不用窗口函数就是自连接


571. Find Median Given Frequency of Numbers


注意:

SELECT a,b,c,d , sum(frequency) FROM Numbers
SELECT a,b,c,d,  sum(frequency) over() FROM Numbers
这俩的区别

如果不加 over 只返回一条数据

```sql
select avg(num) median from (
        select *, sum(frequency) over (order by num) sum_ ,sum(frequency) over() as total from Numbers
    ) a 
where sum_ - frequency <= total/2 and sum_>= total /2 -- 前一堆小于 后一堆大于
```



574. Winning Candidate
直接做出来不用看

打印收获票数最多的候选者名字

+----+------+
| id | name |
+----+------+
| 1  | A    |
| 2  | B    |
| 3  | C    |
| 4  | D    |
| 5  | E    |
+----+------+
Vote table:
+----+-------------+
| id | candidateId |
+----+-------------+
| 1  | 2           |
| 2  | 4           |
| 3  | 3           |
| 4  | 2           |
| 5  | 5           |
+----+-------------+

```sql
# Write your MySQL query statement below
select name from Candidate where id = (
        select candidateId from  
            (select candidateId, count(1)as cnt from Vote group by candidateId order by cnt desc limit 1)a
    )
```


577. Employee Bonus
left join 太简单
奖金小于1000的 或者为空的

Input: 
Employee table:
+-------+--------+------------+--------+
| empId | name   | supervisor | salary |
+-------+--------+------------+--------+
| 3     | Brad   | null       | 4000   |
| 1     | John   | 3          | 1000   |
| 2     | Dan    | 3          | 2000   |
| 4     | Thomas | 3          | 4000   |
+-------+--------+------------+--------+
Bonus table:
+-------+-------+
| empId | bonus |
+-------+-------+
| 2     | 500   |
| 4     | 2000  |
+-------+-------+
```sql
select a.name ,b.bonus from Employee a left join Bonus b on a.empId = b.empId where b.bonus <1000 or b.bonus is NULL
```



578. Get Highest Answer Rate Question


SurveyLog table:
+----+--------+-------------+-----------+-------+-----------+
| id | action | question_id | answer_id | q_num | timestamp |
+----+--------+-------------+-----------+-------+-----------+
| 5  | show   | 285         | null      | 1     | 123       |
| 5  | answer | 285         | 124124    | 1     | 124       |
| 5  | show   | 369         | null      | 2     | 125       |
| 5  | skip   | 369         | null      | 2     | 126       |
+----+--------+-------------+-----------+-------+-----------+
Output: 
+------------+
| survey_log |
+------------+
| 285        |
+------------+

每个问题会被 show  会被answer 
求count(answer) /count(show)  比率最大值的问题的问题号

先查出来各个问题 answer 和 show各多少个, 然后自连接
然后rownumber  = 1

```sql
with temp as(
    select action,question_id, count(1) cnt from SurveyLog where action !='skip'  group by action,question_id 
)
select question_id  as survey_log from (
    select row_number() over(order by a.cnt/b.cnt desc) as no, a.question_id  from temp a inner join temp b on a.question_id = b.question_id and a.action ='answer' and b.action ='show' 
) c where no =1
```



579. Find Cumulative Salary of an Employee
Input: 
Employee table:
+----+-------+--------+
| id | month | salary |
+----+-------+--------+
| 1  | 1     | 20     |
| 2  | 1     | 20     |
| 1  | 2     | 30     |
| 2  | 2     | 30     |
| 3  | 2     | 40     |
| 1  | 3     | 40     |
| 3  | 3     | 60     |
| 1  | 4     | 60     |
| 3  | 4     | 70     |
| 1  | 7     | 90     |
| 1  | 8     | 90     |
+----+-------+--------+
Output: 
+----+-------+--------+
| id | month | Salary |
+----+-------+--------+
| 1  | 7     | 90     |
| 1  | 4     | 130    |
| 1  | 3     | 90     |
| 1  | 2     | 50     |
| 1  | 1     | 20     |
| 2  | 1     | 20     |
| 3  | 3     | 100    |
| 3  | 2     | 40     |
+----+-------+--------+



每个员工每个月往前追溯共三个月的工资总和, 不能算最近的一个月 完了
如果前一个月没工作, 就算0



每一行把之前的两个月列出来, 然后判断等不等于当月n-1 ,和当月n-2, 
```sql
with temp as(
 select id,
    
    month, 
    lag(month,1,0) over(partition by id order by month) as  pre, 
    lag(month,2,0) over(partition by id order by month) as  prepre ,
    
    salary,
    lag(salary,1,0) over(partition by id order by month) as pre_salary,
    lag(salary,2,0) over(partition by id order by month) as prepre_salary,
    
    
    row_number() over(partition by id order by month) as no,
    count(1) over(partition by id) as max_no
    
    from 
    Employee
    )
    
    select * from (
        select  id ,month , 
        case when month-1 = pre then pre_salary else 0 end + 
        case when month-2 = prepre then prepre_salary else 0 end + salary as "salary"
        from temp where no != max_no
    ) a order by id , month desc
```


580. Count Student Number in Departments
分组统计 , 外表补充信息,左链接 coalesce

```sql
select a.dept_name,  coalesce(b.cnt,0) as  student_number from 
    Department a left join 
    (
    select dept_id, count(1) as cnt from Student group by dept_id
    ) b  
    on a.dept_id = b.dept_id order by student_number desc ,dept_name
```



597. Friend Requests I: Overall Acceptance Rate
多字段去重 distinct a,b


with test as (
    select id, cumil1, cumil2 from 
    (
        select *,
            lead(id) over(order by id) as cumil1,
            lead(id,2) over(order by id) as cumil2
            from stadium
            where people>=100
    ) t
        where id +1 = cumil1 and cumil1 + 1 = cumil2
)




601. Human Traffic of Stadium

hard 题 直接摆出来就完了  不难
连续id 都大于100

Input: 
Stadium table:
+------+------------+-----------+
| id   | visit_date | people    |
+------+------------+-----------+
| 1    | 2017-01-01 | 10        |
| 2    | 2017-01-02 | 109       |
| 3    | 2017-01-03 | 150       |
| 4    | 2017-01-04 | 99        |
| 5    | 2017-01-05 | 145       |
| 6    | 2017-01-06 | 1455      |
| 7    | 2017-01-07 | 199       |
| 8    | 2017-01-09 | 188       |
+------+------------+-----------+
Output: 
+------+------------+-----------+
| id   | visit_date | people    |
+------+------------+-----------+
| 5    | 2017-01-05 | 145       |
| 6    | 2017-01-06 | 1455      |
| 7    | 2017-01-07 | 199       |
| 8    | 2017-01-09 | 188       |
+------+------------+-----------+



```sql
with temp as (
    select * from(
        select 
        id, 
        people,
        lead(id,1) over(order by id) id1,
        lead(id,2) over(order by id) id2,
        lead(people,1) over(order by id) people1,
        lead(people,2) over(order by id) people2

        from Stadium
    )a
    where id +1 = id1 and id +2 = id2 and people >=100 and people1>=100 and people2>=100
)

select id, visit_date,people from Stadium where id in (select  id from temp) or id in (select  id1 from temp) or id in (select  id2 from temp)
```

602. Friend Requests II: Who Has the Most Friends
+--------------+-------------+-------------+
| requester_id | accepter_id | accept_date |
+--------------+-------------+-------------+
| 1            | 2           | 2016/06/03  |
| 1            | 3           | 2016/06/08  |
| 2            | 3           | 2016/06/08  |
| 3            | 4           | 2016/06/09  |
+--------------+-------------+-------------+

求拥有最多好友的id  和他拥有的好友数


方法一:
把所有id 撸出来放到一起, 然后count
```py
with temp as(
select no,count(1) as cnt from (
(select requester_id as no from RequestAccepted)
    union all
(select accepter_id as no from RequestAccepted)
)a group by no
) 
select no as id, cnt as num from temp where cnt = (select max(cnt) from temp)
```

603. Consecutive Available Seats


```sql
with temp as(
    select * from(
    select seat_id ,free, lead(seat_id,1) over(order by seat_id) as id1 ,
        lead(free,1) over(order by seat_id) as free1 
        
        from Cinema
        )a
    where seat_id +1 = id1 and free =1 and free1=1
)
select seat_id  from temp 
union
select id1 as seat_id from temp 
order by  seat_id
```

608. Tree Node
Input: 
Tree table:
+----+------+
| id | p_id |
+----+------+
| 1  | null |
| 2  | 1    |
| 3  | 1    |
| 4  | 2    |
| 5  | 2    |
+----+------+
Output: 
+----+-------+
| id | type  |
+----+-------+
| 1  | Root  |
| 2  | Inner |
| 3  | Leaf  |
| 4  | Leaf  |
| 5  | Leaf  |
+----+-------+

打印出叶子节点 根节点内部节点


1. 第一次在case 条件里写 select
2. when id not in (select p_id from tree) then "Leaf"  =========>>> 不知道为啥 not in 不好使 



```sql
select 
id, 
case 
    when p_id is null then "Root" 
    -- when id not in (select p_id from tree) then "Leaf"  =========>>> 不知道为啥 not in 不好使
    when id  in (select p_id from tree) then "Inner" 
    else  "Leaf"
    end as "type"
from tree
```


610. Triangle Judgement

数值计算 判断三角形
```sql
select x,y,z,
CASE when (x+y<=z or y+z<=x or x+z<=y ) then 'No' else 'Yes' end as triangle
from Triangle
```

612. Shortest Distance in a Plane


Point2D table:
+----+----+
| x  | y  |
+----+----+
| -1 | -1 |
| 0  | 0  |
| -1 | -2 |
+----+----+
Output: 
+----------+
| shortest |
+----------+
| 1.00     |
+----------+

给一对坐标求两两的最短距离



1. sqrt
2. 条件判断 两个都相等求排除掉


```sql
select min(round( sqrt((a.x-b.x)*(a.x-b.x) + (a.y-b.y)*(a.y-b.y) ) ,2)) as shortest   from Point2D a inner join Point2D b where not(a.x =b.x and a.y=b.y)
```

614. Second Degree Follower

关注者和被关注者有一棵树
求非根节点非叶子节点
只要第一列的也在第二列 就行了 
```sql
select distinct followee as follower , count(follower) as num
    from follow
        where followee in (select follower from follow)  group by followee order by followee
```

613. Shortest Distance in a Line
x一维 坐标,  求坐标轴的最短距离

+----+
| x  |
+----+
| -1 |
| 0  |
| 2  |
+----+


遇到的坑:
 1. lead(x,1) 写成了  lead(x,1,0) 到了边界就不能再算,边界算成0 铁定不对
 2. (order by x)) 得排序

```sql
select min(a) as shortest from ( select abs(x- lead(x,1) over(order by x)) as a from Point)b
```





615. Average Salary: Departments VS Company
每个部门每个月的平均值 和 整个公司每个月的平均值 相比较 ,输出结果

#  hard 题 写了好长时间
##  date_format(pay_date, "%Y-%m") 日期格式化

每个部门每个月员工薪资的总和/员工数    和 整个公司每个月的业绩总和除以员工数

1. 先把公司员工所属的部门信息补充进薪资表
2. 然后group by 日期, 部门, 这张表作为base
3. 在前面的基础上可以方便的算出部门的平均值 作为表a 
4. 再base表的基础上 group by 部门 用来求公司总的平均值 作为表b
5. 两表用日期月份连接 然后 比较平均值






Salary table:
+----+-------------+--------+------------+
| id | employee_id | amount | pay_date   |
+----+-------------+--------+------------+
| 1  | 1           | 9000   | 2017/03/31 |
| 2  | 2           | 6000   | 2017/03/31 |
| 3  | 3           | 10000  | 2017/03/31 |
| 4  | 1           | 7000   | 2017/02/28 |
| 5  | 2           | 6000   | 2017/02/28 |
| 6  | 3           | 8000   | 2017/02/28 |
+----+-------------+--------+------------+
Employee table:
+-------------+---------------+
| employee_id | department_id |
+-------------+---------------+
| 1           | 1             |
| 2           | 2             |
| 3           | 2             |
+-------------+---------------+
Output: 
+-----------+---------------+------------+
| pay_month | department_id | comparison |
+-----------+---------------+------------+
| 2017-02   | 1             | same       |
| 2017-03   | 1             | higher     |
| 2017-02   | 2             | same       |
| 2017-03   | 2             | lower      |
+-----------+---------------+------------+


```sql
with temp as(
select sum(amount) as amount,count(1) as cnt,date_format(pay_date, "%Y-%m") as pay_month,department_id  from Salary a inner join Employee b on a.employee_id = b.employee_id group by pay_month,department_id
)

select a.pay_month,a.department_id , 
case when round(a.amount/a.cnt,2) > b.avg_ then "higher" when  round(a.amount/a.cnt,2) < b.avg_ then "lower" else "same" end as comparison from temp a 
inner join 
(select pay_month, round(sum(amount)/sum(cnt),2) as avg_ from temp group by pay_month) b  
    on a.pay_month = b.pay_month
    
order by department_id
```




对字段做coalesce 返回的结果为[]
如果要返回null  要直接在套一层 对查询结果做coalesce
Output:
{"headers": ["num"], "values": []}
Expected:
{"headers": ["num"], "values": [[null]]}


```sql
select coalesce((
select num from (
select num,row_number() over(order by num desc) as no ,cnt from (
    select count(1) as cnt,num from MyNumbers group by num
) a where cnt =1
    ) b where no =1)
    ,null) as num
```


626. Exchange Seats
两两交换,如果最后一个是奇数, 就不换


Input: 
Seat table:
+----+---------+
| id | student |
+----+---------+
| 1  | Abbot   |
| 2  | Doris   |
| 3  | Emerson |
| 4  | Green   |
| 5  | Jeames  |
+----+---------+
Output: 
+----+---------+
| id | student |
+----+---------+
| 1  | Doris   |
| 2  | Abbot   |
| 3  | Green   |
| 4  | Emerson |
| 5  | Jeames  |
+----+---------+


把上下两行全打出来, 然后根据条件判断打哪一个

```sql
select  id,  case when s1 is null and id %2=1 then student  when id %2 =1 then s1 when id %2=0 then s2 end as "student" from(
select 
id,
student,
lead(student,1) over() as s1,
lag(student,1) over() as s2
from Seat
)a
```


627. Swap Salary
case 的两种写法:

case sex when 'f' then 'm' 

case when sex ='m' then 'm'

```sql
update Salary set sex =case sex  when 'f' then 'm' else 'f' end
update Salary set sex =case when sex= 'f' then 'm' else 'f' end
```



618. Students Report By Geography
转置, 对齐
Student table:
+--------+-----------+
| name   | continent |
+--------+-----------+
| Jane   | America   |
| Pascal | Europe    |
| Xi     | Asia      |
| Jack   | America   |
+--------+-----------+
Output: 
+---------+------+--------+
| America | Asia | Europe |
+---------+------+--------+
| Jack    | Xi   | Pascal |
| Jane    | null | null   |
+---------+------+--------+


先分组编号 ,然后用max 把某一个编号有数据的都提到最上面

max /count / sum这些都能带条件
```sql
select   
max(if(continent='America',name,null)) as America,
max(if(continent='Asia',name,null)) as Asia,
max(if(continent='Europe',name,null)) as Europe

from (select *, row_number() over(partition by continent order by name) as no from  Student)a group by no
```


1045. Customers Who Bought All Products

1. 一次select 两个表
2. having count

```sql
select customer_id 
    from Customer a, Product b 
        group by customer_id 
            having count(distinct a.product_key) = (select count(1) from Product)
```


1083. Sales Analysis II
easy 题
##### having sum

```sql
select buyer_id from Sales a inner join Product b on a.product_id = b.product_id 
group by buyer_id
having sum(b.product_name='S8') >0 and sum(b.product_name='iPhone') =0
```



1098. Unpopular Books
一个 书的表 一个卖书的订单表
求卖出少于10本的书的id和名字
书的发行日要过滤掉最近一个月的
订单要过滤掉一年之前的

# 要注意 订单表中不存在的也要算作卖出0本
#  date_sub("2019-06-23", interval 1 month)


left join + having sum(coalesce(b.qty,0))

Input: 
Books table:
+---------+--------------------+----------------+
| book_id | name               | available_from |
+---------+--------------------+----------------+
| 1       | "Kalila And Demna" | 2010-01-01     |
| 2       | "28 Letters"       | 2012-05-12     |
| 3       | "The Hobbit"       | 2019-06-10     |
| 4       | "13 Reasons Why"   | 2019-06-01     |
| 5       | "The Hunger Games" | 2008-09-21     |
+---------+--------------------+----------------+
Orders table:
+----------+---------+----------+---------------+
| order_id | book_id | quantity | dispatch_date |
+----------+---------+----------+---------------+
| 1        | 1       | 2        | 2018-07-26    |
| 2        | 1       | 1        | 2018-11-05    |
| 3        | 3       | 8        | 2019-06-11    |
| 4        | 4       | 6        | 2019-06-05    |
| 5        | 4       | 5        | 2019-06-20    |
| 6        | 5       | 9        | 2009-02-02    |
| 7        | 5       | 8        | 2010-04-13    |
+----------+---------+----------+---------------+
Output: 
+-----------+--------------------+
| book_id   | name               |
+-----------+--------------------+
| 1         | "Kalila And Demna" |
| 2         | "28 Letters"       |
| 5         | "The Hunger Games" |
+-----------+--------------------+



```sql
select a.book_id, max(a.name) as name  from Books a left join 
(
    select  book_id, sum(quantity) as qty from Orders where dispatch_date >date_sub("2019-06-23", interval 1 year) group by book_id
) b
on a.book_id = b.book_id
where  a.available_from < date_sub("2019-06-23", interval 1 month)
group by a.book_id having sum(coalesce(b.qty,0)) < 10
order by book_id
```



1107. New Users Daily Count
用户第一次登陆在三个月内的 每天的登录用户个数

先拿login 筛选
然后打标号
然后拿标号和时间卡

```sql
select activity_date as login_date, count(1) as user_count from (
    select user_id,activity_date,row_number() over(partition by user_id order by activity_date) as no  from (
        select user_id,activity_date from Traffic where activity='login'
    ) a 
)b where no =1 and  activity_date>=date_sub('2019-06-30',interval 90 day) group by activity_date
order by activity_date
```




1126. Active Businesses
某个id有至少两个类型大于他的均值
Events table:
+-------------+------------+------------+
| business_id | event_type | occurences |
+-------------+------------+------------+
| 1           | reviews    | 7          |
| 3           | reviews    | 3          |
| 1           | ads        | 11         |
| 2           | ads        | 7          |
| 3           | ads        | 6          |
| 1           | page views | 3          |
| 2           | page views | 12         |
+-------------+------------+------------+
Output: 
+-------------+
| business_id |
+-------------+
| 1           |
+-------------+


```sql
select a.business_id from 
    Events a inner join (
        select event_type, avg(occurences) as avg_ from Events group by event_type
    )b
    on a.event_type = b.event_type
        where a.occurences > b.avg_
        group by a.business_id having count(1) >1

```


1142. User Activity for the Past 30 Days II
easy 题直接过 记录一下

```sql
select  coalesce(round((
select count(distinct session_id)   from Activity where activity_date > date_sub('2019-07-27', interval 30 day)
)/
(
select count(distinct user_id)   from Activity where activity_date > date_sub('2019-07-27', interval 30 day) 
),2) ,0) as average_sessions_per_user
```


1158. Market Analysis I

用户在1019年买的书
没买也算0

Input: 
Users table:
+---------+------------+----------------+
| user_id | join_date  | favorite_brand |
+---------+------------+----------------+
| 1       | 2018-01-01 | Lenovo         |
| 2       | 2018-02-09 | Samsung        |
| 3       | 2018-01-19 | LG             |
| 4       | 2018-05-21 | HP             |
+---------+------------+----------------+
Orders table:
+----------+------------+---------+----------+-----------+
| order_id | order_date | item_id | buyer_id | seller_id |
+----------+------------+---------+----------+-----------+
| 1        | 2019-08-01 | 4       | 1        | 2         |
| 2        | 2018-08-02 | 2       | 1        | 3         |
| 3        | 2019-08-03 | 3       | 2        | 3         |
| 4        | 2018-08-04 | 1       | 4        | 2         |
| 5        | 2018-08-04 | 1       | 3        | 4         |
| 6        | 2019-08-05 | 2       | 2        | 4         |
+----------+------------+---------+----------+-----------+
Items table:
+---------+------------+
| item_id | item_brand |
+---------+------------+
| 1       | Samsung    |
| 2       | Lenovo     |
| 3       | LG         |
| 4       | HP         |
+---------+------------+

```sql
select buyer_id, max(join_date) as join_date,  coalesce(count(1),0) as orders_in_2019 from Users b left join 
Orders a
on a.buyer_id = b.user_id where order_date between '2019-01-01' and '2019-12-31' group by buyer_id
union 
    select user_id,join_date, 0 as orders_in_2019  from Users where user_id not in(
     select buyer_id from Orders where  order_date between '2019-01-01' and '2019-12-31') 

```

1127. User Purchase Platform
单独 desktop的 单独是 mobile的 两个都买了的 都各自统计出来
如果以上三个类型有一个订单中没有也应该输出,  total_amount | total_users 这两咧都是0
Input: 
Spending table:
+---------+------------+----------+--------+
| user_id | spend_date | platform | amount |
+---------+------------+----------+--------+
| 1       | 2019-07-01 | mobile   | 100    |
| 1       | 2019-07-01 | desktop  | 100    |
| 2       | 2019-07-01 | mobile   | 100    |
| 2       | 2019-07-02 | mobile   | 100    |
| 3       | 2019-07-01 | desktop  | 100    |
| 3       | 2019-07-02 | desktop  | 100    |
+---------+------------+----------+--------+
Output: 
+------------+----------+--------------+-------------+
| spend_date | platform | total_amount | total_users |
+------------+----------+--------------+-------------+
| 2019-07-01 | desktop  | 100          | 1           |
| 2019-07-01 | mobile   | 100          | 1           |
| 2019-07-01 | both     | 200          | 1           |
| 2019-07-02 | desktop  | 100          | 1           |
| 2019-07-02 | mobile   | 100          | 1           |
| 2019-07-02 | both     | 0            | 0           |
+------------+----------+--------------+-------------+

卧槽写的这一屏幕

实际上就是先给每个user 打上标号 统计count  用count 来区分是上面三种的那一种

1. count(distinct user_id)


```sql
with temp as(
    select a.user_id, a.amount,a.spend_date,a.platform,b.cnt as cnt from Spending a inner join 
(
    select user_id, spend_date,count(1)as cnt from Spending
 group by user_id, spend_date
) b
on a.user_id = b.user_id and a.spend_date = b.spend_date
)

select spend_date, "both" as platform, sum(amount)as total_amount ,count(distinct user_id) as total_users from temp where cnt =2 group by spend_date
union
select spend_date, "desktop" as platform, sum(amount)as total_amount ,count(distinct user_id) as total_users from temp where cnt =1 and platform = "desktop" group by spend_date
union
select spend_date, "mobile" as platform, sum(amount)as total_amount ,count(distinct user_id) as total_users from temp where cnt =1 and platform = "mobile" group by spend_date
union
select distinct  spend_date, platform, total_amount,total_users from (
    select spend_date, "both" as platform ,0 as total_amount, 0 as total_users from Spending where spend_date not in(
        select spend_date from temp where cnt =2
    )
    )a
    union
select distinct  spend_date, platform, total_amount,total_users from (
    select spend_date, "desktop" as platform ,0 as total_amount, 0 as total_users from Spending where spend_date not in(
        select spend_date from temp where cnt =1 and  platform = "desktop" 
    )
    )b
union
select distinct  spend_date, platform, total_amount,total_users from (
    select spend_date, "mobile" as platform ,0 as total_amount, 0 as total_users from Spending where spend_date not in(
        select spend_date from temp where cnt = 1 and platform = "mobile" 
    )
    )c


order by field(platform,"desktop","mobile","both")  ,spend_date
```


1132. Reported Posts II

垃圾邮件删除率
原表需要去重
先根据删除表给每个Action条目打上 removed 1 or 0
然后
sum(removed)/count(1) 
然后全部加起来求平均值


Actions table:
+---------+---------+-------------+--------+--------+
| user_id | post_id | action_date | action | extra  |
+---------+---------+-------------+--------+--------+
| 1       | 1       | 2019-07-01  | view   | null   |
| 1       | 1       | 2019-07-01  | like   | null   |
| 1       | 1       | 2019-07-01  | share  | null   |
| 2       | 2       | 2019-07-04  | view   | null   |
| 2       | 2       | 2019-07-04  | report | spam   |
| 3       | 4       | 2019-07-04  | view   | null   |
| 3       | 4       | 2019-07-04  | report | spam   |
| 4       | 3       | 2019-07-02  | view   | null   |
| 4       | 3       | 2019-07-02  | report | spam   |
| 5       | 2       | 2019-07-03  | view   | null   |
| 5       | 2       | 2019-07-03  | report | racism |
| 5       | 5       | 2019-07-03  | view   | null   |
| 5       | 5       | 2019-07-03  | report | racism |
+---------+---------+-------------+--------+--------+
Removals table:
+---------+-------------+
| post_id | remove_date |
+---------+-------------+
| 2       | 2019-07-20  |
| 3       | 2019-07-18  |
+---------+-------------+
Output: 
+-----------------------+
| average_daily_percent |
+-----------------------+
| 75.00                 |
+-----------------------+


```sql
with temp as(
    select  sum(removed)/count(1) as rate, action_date from (
    select a.*,case when b.remove_date is null then 0 else 1 end as removed from (
      select distinct post_id, action_date,extra from Actions
    ) a left join Removals b on a.post_id = b.post_id where a.extra='spam'
    )a group by action_date
)

select round(avg(rate)*100,2) as average_daily_percent from temp
```



1159. Market Analysis II
三个表 用户表, 订单表, 产品表
每个用户以时间排名第二名买的商品是否是这个用户最喜欢的品牌

Input: 
Users table:
+---------+------------+----------------+
| user_id | join_date  | favorite_brand |
+---------+------------+----------------+
| 1       | 2019-01-01 | Lenovo         |
| 2       | 2019-02-09 | Samsung        |
| 3       | 2019-01-19 | LG             |
| 4       | 2019-05-21 | HP             |
+---------+------------+----------------+
Orders table:
+----------+------------+---------+----------+-----------+
| order_id | order_date | item_id | buyer_id | seller_id |
+----------+------------+---------+----------+-----------+
| 1        | 2019-08-01 | 4       | 1        | 2         |
| 2        | 2019-08-02 | 2       | 1        | 3         |
| 3        | 2019-08-03 | 3       | 2        | 3         |
| 4        | 2019-08-04 | 1       | 4        | 2         |
| 5        | 2019-08-04 | 1       | 3        | 4         |
| 6        | 2019-08-05 | 2       | 2        | 4         |
+----------+------------+---------+----------+-----------+
Items table:
+---------+------------+
| item_id | item_brand |
+---------+------------+
| 1       | Samsung    |
| 2       | Lenovo     |
| 3       | LG         |
| 4       | HP         |
+---------+------------+
Output: 
+-----------+--------------------+
| seller_id | 2nd_item_fav_brand |
+-----------+--------------------+
| 1         | no                 |
| 2         | yes                |
| 3         | yes                |
| 4         | no                 |
+-----------+--------------------+

先三表连接, 订单表打上分组的rownumber 判断是不是
然后补上 没下订单的用户和下订单不够的用户
```sql
select seller_id , 
case when  favorite_brand = item_brand then "yes" else "no" end as 2nd_item_fav_brand
from (
    select a.*,b.favorite_brand,c.item_brand, row_number()  over(partition by user_id order by order_date) as no from 
        Orders a  
        inner join Users b 
        inner join Items c 
            on a.item_id = c.item_id
            and a.seller_id =b.user_id  
) a where no =2

union 
select user_id,"no" as 2nd_item_fav_brand from Users where user_id not in (select seller_id from Orders group by seller_id having count(1) >=2)
```


1164. Product Price at a Given Date
Products table:
+------------+-----------+-------------+
| product_id | new_price | change_date |
+------------+-----------+-------------+
| 1          | 20        | 2019-08-14  |
| 2          | 50        | 2019-08-14  |
| 1          | 30        | 2019-08-15  |
| 1          | 35        | 2019-08-16  |
| 2          | 65        | 2019-08-17  |
| 3          | 20        | 2019-08-18  |
+------------+-----------+-------------+
Output: 
+------------+-------+
| product_id | price |
+------------+-------+
| 2          | 50    |
| 1          | 35    |
| 3          | 10    |
+------------+-------+
2019-08-16 时所有商品的价格
就是截止到2019-08-16, 每个商品的最新一条的价格更改到的价格, 每个商品最初的价格是10
先把表打上标号,然后挑出2019-08-16 最新一条价格 然后把上面没有数据的但是又出现在product里面的 都标记成10


```sql
with temp as(
       select * from (
           select *, row_number() over(partition by product_id order by change_date desc) as no from  Products where change_date <= '2019-08-16'
    ) a where no =1
) 

select distinct product_id , 10 as price from Products where product_id not in (select product_id from temp)
union
select product_id, new_price as price from temp
```

1193. Monthly Transactions I
#  count if sum if date_format %Y-%m
Input: 
Transactions table:
+------+---------+----------+--------+------------+
| id   | country | state    | amount | trans_date |
+------+---------+----------+--------+------------+
| 121  | US      | approved | 1000   | 2018-12-18 |
| 122  | US      | declined | 2000   | 2018-12-19 |
| 123  | US      | approved | 2000   | 2019-01-01 |
| 124  | DE      | approved | 2000   | 2019-01-07 |
+------+---------+----------+--------+------------+
Output: 
+----------+---------+-------------+----------------+--------------------+-----------------------+
| month    | country | trans_count | approved_count | trans_total_amount | approved_total_amount |
+----------+---------+-------------+----------------+--------------------+-----------------------+
| 2018-12  | US      | 2           | 1              | 3000               | 1000                  |
| 2019-01  | US      | 1           | 1              | 2000               | 2000                  |
| 2019-01  | DE      | 1           | 1              | 2000               | 2000                  |
+----------+---------+-------------+----------------+--------------------+-----------------------+

```sql
select  date_format(trans_date,"%Y-%m") as month , 
        country, 
        count(1) as trans_count, 
        count(if(state='approved',1,null)) as approved_count, 
        sum(amount) as trans_total_amount,
        sum(if(state='approved',amount,0))as approved_total_amount 
            from Transactions 
                group by date_format(trans_date,"%Y-%m"),country
```


1194. Tournament Winners
matches 表两个选手对决 和各自的分数
先各自 sum , 然后union 起来sum,把两排归并成一排 , 成为(player_id, score) ,然后join player表 标上组号
然后分组打标号选第一名
能做出来 就是步骤复杂点 每一步都不难
Input: 
Players table:
+-----------+------------+
| player_id | group_id   |
+-----------+------------+
| 15        | 1          |
| 25        | 1          |
| 30        | 1          |
| 45        | 1          |
| 10        | 2          |
| 35        | 2          |
| 50        | 2          |
| 20        | 3          |
| 40        | 3          |
+-----------+------------+
Matches table:
+------------+--------------+---------------+-------------+--------------+
| match_id   | first_player | second_player | first_score | second_score |
+------------+--------------+---------------+-------------+--------------+
| 1          | 15           | 45            | 3           | 0            |
| 2          | 30           | 25            | 1           | 2            |
| 3          | 30           | 15            | 2           | 0            |
| 4          | 40           | 20            | 5           | 2            |
| 5          | 35           | 50            | 1           | 1            |
+------------+--------------+---------------+-------------+--------------+
```sql
with temp as(
    select c.player_id,score,group_id from 
    (
            select player_id, sum(score) as score from 
            (
                    (select first_player as player_id, sum(first_score)as score  from Matches group by first_player)
                    union all 
                    (select second_player as player_id,sum(second_score)as score from Matches group by second_player)
            )a  group by player_id
    ) c inner join Players d on c.player_id = d.player_id
) 


select group_id, player_id from(
    select *,row_number() over(partition by group_id order by score desc, player_id) as no from temp
    )a  where no =1
```


1204. Last Person to Fit in the Bus

直接做出来不难 还是标号的问题
巴士总成重1000, 按turn的顺序上车 直到装不下, 求最后一个人

第一步 先sum(weight) over 求 按turn 累加
第二部 把每个人的下一步的累加和放到当前行
第三部 当前行和下一行的条件 满足 t<=1000 and t1>1000 为答案

Queue table:
+-----------+-------------+--------+------+
| person_id | person_name | weight | turn |
+-----------+-------------+--------+------+
| 5         | Alice       | 250    | 1    |
| 4         | Bob         | 175    | 5    |
| 3         | Alex        | 350    | 2    |
| 6         | John Cena   | 400    | 3    |
| 1         | Winston     | 500    | 6    |
| 2         | Marie       | 200    | 4    |
+-----------+-------------+--------+------+
Output: 
+-------------+
| person_name |
+-------------+
| John Cena   |
```sql
select person_name from (
        select *, lead(t,1,10000000) over(order by turn) as t1 from (
            select *, sum(Weight) over(order by turn) as t from Queue 
        )a 
)b where t<=1000 and t1>1000
```



1205. Monthly Transactions II
直接做出来的
Transactions table:
+-----+---------+----------+--------+------------+
| id  | country | state    | amount | trans_date |
+-----+---------+----------+--------+------------+
| 101 | US      | approved | 1000   | 2019-05-18 |
| 102 | US      | declined | 2000   | 2019-05-19 |
| 103 | US      | approved | 3000   | 2019-06-10 |
| 104 | US      | declined | 4000   | 2019-06-13 |
| 105 | US      | approved | 5000   | 2019-06-15 |
+-----+---------+----------+--------+------------+
Chargebacks table:
+----------+------------+
| trans_id | trans_date |
+----------+------------+
| 102      | 2019-05-29 |
| 101      | 2019-06-30 |
| 105      | 2019-09-18 |
+----------+------------+
Output: 
+---------+---------+----------------+-----------------+------------------+-------------------+
| month   | country | approved_count | approved_amount | chargeback_count | chargeback_amount |
+---------+---------+----------------+-----------------+------------------+-------------------+
| 2019-05 | US      | 1              | 1000            | 1                | 2000              |
| 2019-06 | US      | 2              | 8000            | 1                | 1000              |
| 2019-09 | US      | 0              | 0               | 1                | 5000              |
+---------+---------+----------------+-----------------+------------------+-------------------+

Chargebacks是退款的意思
统计每个国家每个月approved ,Chargebacks的总钱数和笔数
把 Chargebacks这张表fake成Transactions 的条目 union 在一起

然后统计


```sql
select * from (
select  date_format(trans_date,"%Y-%m") as month , 
        country, 
        count(if(state='approved',1,null)) as approved_count, 
        sum(if(state='approved',amount,0))as approved_amount,
        count(if(state='chargeback',1,null)) as chargeback_count, 
        sum(if(state='chargeback',amount,0))as chargeback_amount
        from (
            (select * from Transactions )
            union all
            (select a.id , a.country ,'chargeback' as state,a.amount,b.trans_date as trans_date from  Transactions a inner join Chargebacks b on a.id = b.trans_id)
        )c    group by date_format(trans_date,"%Y-%m"),country
)d where not (approved_count =0 and approved_amount =0 and chargeback_count =0 and chargeback_amount=0)
```


1212. Team Scores in Football Tournament
踢足球 , matches 两边的分数
赢得的3分 
平的得1分
输的的0分
求所有队伍总分

把所有队伍的各种情况的分数查出来, 然后整理成一个表 group by

每一条数据赢家的分数对应输家的ID 是0分  也要加进去


Input: 
Teams table:
+-----------+--------------+
| team_id   | team_name    |
+-----------+--------------+
| 10        | Leetcode FC  |
| 20        | NewYork FC   |
| 30        | Atlanta FC   |
| 40        | Chicago FC   |
| 50        | Toronto FC   |
+-----------+--------------+
Matches table:
+------------+--------------+---------------+-------------+--------------+
| match_id   | host_team    | guest_team    | host_goals  | guest_goals  |
+------------+--------------+---------------+-------------+--------------+
| 1          | 10           | 20            | 3           | 0            |
| 2          | 30           | 10            | 2           | 2            |
| 3          | 10           | 50            | 5           | 1            |
| 4          | 20           | 30            | 1           | 0            |
| 5          | 50           | 30            | 1           | 0            |
+------------+--------------+---------------+-------------+--------------+
Output: 
+------------+--------------+---------------+
| team_id    | team_name    | num_points    |
+------------+--------------+---------------+
| 10         | Leetcode FC  | 7             |
| 20         | NewYork FC   | 3             |
| 50         | Toronto FC   | 3             |
| 30         | Atlanta FC   | 1             |
| 40         | Chicago FC   | 0             |
+------------+--------------+---------------+


```sql
select b.team_id, b.team_name, sum(num_points) as num_points 
from (
    select host_team as team_id , 3 as num_points  from Matches where host_goals > guest_goals  -- 每一条数据主场赢的分数和赢家的id
    union all 
    select guest_team as team_id , 0 as num_points  from Matches where host_goals > guest_goals -- 每一条数据赢家的分数对应输家的ID
    union all 
    select guest_team as team_id , 3 as num_points  from Matches where host_goals < guest_goals  --每一条数据赢家的分数对应输家的ID 
    union all 
    select host_team as team_id , 0 as num_points  from Matches where host_goals < guest_goals 
    union all 
    select host_team as team_id , 1 as num_points  from Matches where host_goals = guest_goals 
    union all 
    select guest_team as team_id , 1 as num_points  from Matches where host_goals = guest_goals 
    union all 
    select team_id , 0 as num_points  from Teams where team_id not in( -- 注意 没参加比赛的d 也要作为0分干进去
      (select guest_team from Matches)
        union 
      (select host_team from Matches)
    )
) a inner join Teams b 
    on a.team_id = b.team_id
        group by team_id
    order by sum(num_points)  desc, team_id

```

1225. Report Contiguous Dates
成功表和失败表的连续时间段, 如果只有一天, 那么时间段开始和结束的时间是一天
先把每条数据打上 上一天和下一天的字段 用lead 和lag
然后 筛选出 当前日期不等于上一条的日期 和 当前日期不等于下一条的数据 这些数据就是开始和截止时间
其中当前日期不等于上一条 是开始点, 当前日期不等于下一条  是 某一段范围的结束时间
这两个时间肯定是配对的 如果开始和结束是一天 , 那么这些时间即不等于上一天 也不等于下一天
所以 开始时间和结束时间单独选出来打上标号 然后拿标号join 就可以同时取出 同一时间段的开始和结束时间
最后union  order by




Failed table:
+-------------------+
| fail_date         |
+-------------------+
| 2018-12-28        |
| 2018-12-29        |
| 2019-01-04        |
| 2019-01-05        |
+-------------------+
Succeeded table:
+-------------------+
| success_date      |
+-------------------+
| 2018-12-30        |
| 2018-12-31        |
| 2019-01-01        |
| 2019-01-02        |
| 2019-01-03        |
| 2019-01-06        |
+-------------------+
Output: 
+--------------+--------------+--------------+
| period_state | start_date   | end_date     |
+--------------+--------------+--------------+
| succeeded    | 2019-01-01   | 2019-01-03   |
| failed       | 2019-01-04   | 2019-01-05   |
| succeeded    | 2019-01-06   | 2019-01-06   |
+--------------+--------------+--------------+


```sql
# Write your MySQL query statement below
with t1 as(
    select * from (
        select 
            fail_date, 
            lag(fail_date, 1,'1970-01-01')over() as prev_date,
            lead(fail_date, 1,'1970-01-01')over() as next_date 
                from Failed where fail_date between '2019-01-01' and '2019-12-31'
    )a 
    where datediff(fail_date,prev_date) != 1 or  datediff(next_date,fail_date) != 1
),
t2 as(
    select * from (
        select 
            success_date,  
            lag(success_date, 1,'1970-01-01')over() as prev_date,
            lead(success_date, 1,'1970-01-01')over() as next_date 
                from Succeeded where success_date between '2019-01-01' and '2019-12-31'
    )a 
    where datediff(prev_date, success_date) != 1 or  datediff(success_date,next_date) != 1
)

(
select "failed" as period_state, start.fail_date as start_date, end.fail_date as end_date from 
(select fail_date , row_number() over(order by fail_date) as no from t1 where datediff(fail_date,prev_date) != 1) as start
inner join
(select fail_date , row_number() over(order by fail_date) as no from t1  where datediff(next_date,fail_date) != 1) as end
on start.no = end.no
)
union


select "succeeded" as period_state, start.success_date as start_date, end.success_date as end_date from 
(select success_date , row_number() over(order by success_date) as no from t2 where datediff(success_date,prev_date) != 1) as start
inner join
(select success_date , row_number() over(order by success_date) as no from t2  where datediff(next_date,success_date) != 1) as end
on start.no = end.no
    
order by start_date
```

```sql
    select a.sub_id as post_id, 
        count(distinct b.sub_id) as number_of_comments

    from submissions a left join submissions b on a.sub_id = b.parent_id
    where a.parent_id is null 
    group by 1 order by 1
```

1270. All People Report to the Given Manager




表组成的一棵树


Employees table:
+-------------+---------------+------------+
| employee_id | employee_name | manager_id |
+-------------+---------------+------------+
| 1           | Boss          | 1          |
| 3           | Alice         | 3          |
| 2           | Bob           | 1          |
| 4           | Daniel        | 2          |
| 7           | Luis          | 4          |
| 8           | Jhon          | 3          |
| 9           | Angela        | 8          |
| 77          | Robert        | 1          |
+-------------+---------------+------------+
Output: 
+-------------+
| employee_id |
+-------------+
| 2           |
| 77          |
| 4           |
| 7           |
+-------------+
挑出深度不超过3的

```sql
select a.employee_id
from   employees a
       inner join employees b
               on a.manager_id = b.employee_id
       inner join employees c
               on c.employee_id = b.manager_id
where c.manager_id = 1
       and a.employee_id != 1
```


1280. Students and Examinations
学生表 课程表 考试表
统计每个学生每门课程各考了多少次
有的可能一次都没考过 考试表里没有


先 学生表 课程表得出全量 , 然后left join  考试表
然后count(d 的考试字段) 

Students table:
+------------+--------------+
| student_id | student_name |
+------------+--------------+
| 1          | Alice        |
| 2          | Bob          |
| 13         | John         |
| 6          | Alex         |
+------------+--------------+
Subjects table:
+--------------+
| subject_name |
+--------------+
| Math         |
| Physics      |
| Programming  |
+--------------+

Examinations table:
+------------+--------------+
| student_id | subject_name |
+------------+--------------+
| 1          | Math         |
| 1          | Physics      |
| 1          | Programming  |
| 2          | Programming  |
| 1          | Physics      |
| 1          | Math         |
| 13         | Math         |
| 13         | Programming  |
| 13         | Physics      |
| 2          | Math         |
| 1          | Math         |
+------------+--------------+


```sql
select c.student_id, c.student_name,c.subject_name,
     count(d.subject_name) as attended_exams -- 主要是这一条
from (
select a.*,b.* from  Students a inner join Subjects b
) c 
left join Examinations d
on c.student_id = d.student_id and c.subject_name =d.subject_name
group by c.student_id, c.subject_name
order by c.student_id,c.subject_name
```

1285. Find the Start and End Number of Continuous Ranges

和上面那个时间连续分段的题目是一样的
一定会配对

Logs table:
+------------+
| log_id     |
+------------+
| 1          |
| 2          |
| 3          |
| 7          |
| 8          |
| 10         |
+------------+
Output: 
+------------+--------------+
| start_id   | end_id       |
+------------+--------------+
| 1          | 3            |
| 7          | 8            |
| 10         | 10           |
+------------+--------------+
```sql
with temp as(
    select * from (
        select log_id , lead(log_id,1, 4294967296)  over(order by log_id) as next_id, lag(log_id,1, 4294967296) over(order by log_id) as prev_id
        from Logs
    )a where log_id +1 !=  next_id or log_id -1 != prev_id
)

select s.log_id as start_id , e.log_id as end_id 
from 
    (select log_id, row_number()  over(order by log_id) as no from temp where log_id -1 != prev_id) s
    inner join
    (select log_id, row_number()  over(order by log_id) as no from temp where  log_id +1 !=  next_id) e
    on s.no = e.no
```

1294. Weather Type in Each Country
所有国家11月的天气类型

Countries table:
+------------+--------------+
| country_id | country_name |
+------------+--------------+
| 2          | USA          |
| 3          | Australia    |
| 7          | Peru         |
| 5          | China        |
| 8          | Morocco      |
| 9          | Spain        |
+------------+--------------+
Weather table:
+------------+---------------+------------+
| country_id | weather_state | day        |
+------------+---------------+------------+
| 2          | 15            | 2019-11-01 |
| 2          | 12            | 2019-10-28 |
| 2          | 12            | 2019-10-27 |
| 3          | -2            | 2019-11-10 |
| 3          | 0             | 2019-11-11 |
| 3          | 3             | 2019-11-12 |
| 5          | 16            | 2019-11-07 |
| 5          | 18            | 2019-11-09 |
| 5          | 21            | 2019-11-23 |
| 7          | 25            | 2019-11-28 |
| 7          | 22            | 2019-12-01 |
| 7          | 20            | 2019-12-02 |
| 8          | 25            | 2019-11-05 |
| 8          | 27            | 2019-11-15 |
| 8          | 31            | 2019-11-25 |
| 9          | 7             | 2019-10-23 |
| 9          | 3             | 2019-12-23 |
+------------+---------------+------------+
Output: 
+--------------+--------------+
| country_name | weather_type |
+--------------+--------------+
| USA          | Cold         |
| Australia    | Cold         |
| Peru         | Hot          |
| Morocco      | Hot          |
| China        | Warm         |
+--------------+--------------+

<=15 cold
>=25 hot
else  warm
```sql
select 
    country_name,
    case when avg(weather_state) <=15 then "Cold" when avg(weather_state) >=25 then "Hot" else "Warm" end as weather_type
    from Weather a inner join Countries b on a.country_id = b.country_id
    where a.day >='2019-11-01' and a.day <='2019-11-30' 
group by a.country_id
```



1321. Restaurant Growth

以7天为时间窗口的平均值
不足7天的不显示
时间都是连续的


先按日期 group by 一下
拿一个字段记录到目前为止的累计总和
用lag 拿到7天前的累积和
当前累积和- 7 天前的累积和
cut 掉前7条数据


Customer table:
+-------------+--------------+--------------+-------------+
| customer_id | name         | visited_on   | amount      |
+-------------+--------------+--------------+-------------+
| 1           | Jhon         | 2019-01-01   | 100         |
| 2           | Daniel       | 2019-01-02   | 110         |
| 3           | Jade         | 2019-01-03   | 120         |
| 4           | Khaled       | 2019-01-04   | 130         |
| 5           | Winston      | 2019-01-05   | 110         | 
| 6           | Elvis        | 2019-01-06   | 140         | 
| 7           | Anna         | 2019-01-07   | 150         |
| 8           | Maria        | 2019-01-08   | 80          |
| 9           | Jaze         | 2019-01-09   | 110         | 
| 1           | Jhon         | 2019-01-10   | 130         | 
| 3           | Jade         | 2019-01-10   | 150         | 
+-------------+--------------+--------------+-------------+
Output: 
+--------------+--------------+----------------+
| visited_on   | amount       | average_amount |
+--------------+--------------+----------------+
| 2019-01-07   | 860          | 122.86         |
| 2019-01-08   | 840          | 120            |
| 2019-01-09   | 840          | 120            |
| 2019-01-10   | 1000         | 142.86         |
+--------------+--------------+----------------+

```sql
select visited_on,(s - prev_sum) as amount, round((s- prev_sum)/7,2) as average_amount from (
        select *,lag(s,7,0) over() as prev_sum,row_number() over() as no from(
            select *, sum(amount) over(order by visited_on) as s from (
                select visited_on, sum(amount) as amount from  Customer group by visited_on
            ) d group by visited_on
        ) a
)b where no >=7
``` 





1322. Ads Performance


Ads table:
+-------+---------+---------+
| ad_id | user_id | action  |
+-------+---------+---------+
| 1     | 1       | Clicked |
| 2     | 2       | Clicked |
| 3     | 3       | Viewed  |
| 5     | 5       | Ignored |
| 1     | 7       | Ignored |
| 2     | 7       | Viewed  |
| 3     | 5       | Clicked |
| 1     | 4       | Viewed  |
| 2     | 11      | Viewed  |
| 1     | 2       | Clicked |
+-------+---------+---------+


Output: 
+-------+-------+
| ad_id | ctr   |
+-------+-------+
| 1     | 66.67 |
| 3     | 50.00 |
| 2     | 33.33 |
| 5     | 0.00  |
+-------+-------+

统计每个广告 click 占 (click+viewd)的比例
count countif 没啥好说的
没有比例的 单独union进去

```sql
with temp as(
    select ad_id, case when cnt =0 then 0 else round(click_cnt/cnt*100,2) end as ctr from(
        select ad_id,count(if(action='Clicked',1,null))as click_cnt , count(1) as cnt from Ads where action !='Ignored' group by ad_id
    ) a  
)

(select * from temp )
union
(select ad_id, 0 as ctr from Ads where  ad_id not in (select ad_id from temp))
order by ctr desc, ad_id
```



1327. List the Products Ordered in a Period
esay 题
没啥好说的
2020-09 下单的产品总数大于100 的名字和产品总数

Input: 
Products table:
+-------------+-----------------------+------------------+
| product_id  | product_name          | product_category |
+-------------+-----------------------+------------------+
| 1           | Leetcode Solutions    | Book             |
| 2           | Jewels of Stringology | Book             |
| 3           | HP                    | Laptop           |
| 4           | Lenovo                | Laptop           |
| 5           | Leetcode Kit          | T-shirt          |
+-------------+-----------------------+------------------+
Orders table:
+--------------+--------------+----------+
| product_id   | order_date   | unit     |
+--------------+--------------+----------+
| 1            | 2020-02-05   | 60       |
| 1            | 2020-02-10   | 70       |
| 2            | 2020-01-18   | 30       |
| 2            | 2020-02-11   | 80       |
| 3            | 2020-02-17   | 2        |
| 3            | 2020-02-24   | 3        |
| 4            | 2020-03-01   | 20       |
| 4            | 2020-03-04   | 30       |
| 4            | 2020-03-04   | 60       |
| 5            | 2020-02-25   | 50       |
| 5            | 2020-02-27   | 50       |
| 5            | 2020-03-01   | 50       |
+--------------+--------------+----------+
Output: 
+--------------------+---------+
| product_name       | unit    |
+--------------------+---------+
| Leetcode Solutions | 130     |
| Leetcode Kit       | 100     |
+--------------------+---------+

```sql
# Write your MySQL query statement below
select d.product_name, c.unit  from (
    select product_id,unit from (
    select product_id, sum(unit)as unit   from Orders where date_format(order_date ,"%Y-%m") = "2020-02"  group by product_id
        ) a where unit >= 100
) c inner join Products d on c.product_id=d.product_id
```


1341. Movie Rating
直接做出来 不用看
查两个最大值 合起来 
一个是投票条目最多的用户的名字
一个是rating 平均最高的电影的名字

Movies table:
+-------------+--------------+
| movie_id    |  title       |
+-------------+--------------+
| 1           | Avengers     |
| 2           | Frozen 2     |
| 3           | Joker        |
+-------------+--------------+
Users table:
+-------------+--------------+
| user_id     |  name        |
+-------------+--------------+
| 1           | Daniel       |
| 2           | Monica       |
| 3           | Maria        |
| 4           | James        |
+-------------+--------------+
MovieRating table:
+-------------+--------------+--------------+-------------+
| movie_id    | user_id      | rating       | created_at  |
+-------------+--------------+--------------+-------------+
| 1           | 1            | 3            | 2020-01-12  |
| 1           | 2            | 4            | 2020-02-11  |
| 1           | 3            | 2            | 2020-02-12  |
| 1           | 4            | 1            | 2020-01-01  |
| 2           | 1            | 5            | 2020-02-17  | 
| 2           | 2            | 2            | 2020-02-01  | 
| 2           | 3            | 2            | 2020-03-01  |
| 3           | 1            | 3            | 2020-02-22  | 
| 3           | 2            | 4            | 2020-02-25  | 
+-------------+--------------+--------------+-------------+
```sql
select name as results from (
select * ,row_number() over(order by cnt desc , name) as no from(
     select a.user_id,  count(distinct movie_id) as cnt ,b.name from MovieRating a inner join Users b on a.user_id = b.user_id  group by a.user_id
) a
    ) c where no =1
    
    
union 

select title as results from (
select *,row_number() over(order by avg_ desc ,title) as no from(
      select a.movie_id,b.title,avg(rating) as avg_  from MovieRating a inner join Movies b on a.movie_id = b.movie_id  where date_format(created_at , "%Y-%m") = '2020-02'  group by a.movie_id
    ) a
) c where no =1
```


1364. Number of Trusted Contacts of a Customer

直接做出来的 就是业务比较复杂

Contacts表 是user_id 联系的contact_name 名字 ,但是只有名字再customer 中的才会被信任
题名要找出每张发票对应的user_id 联系人数量和这些联系人中信任的数量


不能用 发送票 各自join customer 和 contacts

1. 先筛选出 contacts 中存在于custormer中的name ,过滤掉没有的
2. 左连接到 contacts 中 ,  每个user_id 都冗余一个 customer count  作为 trust_customer_cnt 后面用max 来提取出来
3. 连接得到的表 用group by 求出userid 的数量 作为contacts_cnt, 上面的trust_customer_cnt 用max 提取
4. 得到的结果left join 到发票 表 得到结果, 然后在join一下constoer表 得到名字



Input: 
Customers table:
+-------------+---------------+--------------------+
| customer_id | customer_name | email              |
+-------------+---------------+--------------------+
| 1           | Alice         | alice@leetcode.com |
| 2           | Bob           | bob@leetcode.com   |
| 13          | John          | john@leetcode.com  |
| 6           | Alex          | alex@leetcode.com  |
+-------------+---------------+--------------------+
Contacts table:
+-------------+--------------+--------------------+
| user_id     | contact_name | contact_email      |
+-------------+--------------+--------------------+
| 1           | Bob          | bob@leetcode.com   |
| 1           | John         | john@leetcode.com  |
| 1           | Jal          | jal@leetcode.com   |
| 2           | Omar         | omar@leetcode.com  |
| 2           | Meir         | meir@leetcode.com  |
| 6           | Alice        | alice@leetcode.com |
+-------------+--------------+--------------------+
Invoices table:
+------------+-------+---------+
| invoice_id | price | user_id |
+------------+-------+---------+
| 77         | 100   | 1       |
| 88         | 200   | 1       |
| 99         | 300   | 2       |
| 66         | 400   | 2       |
| 55         | 500   | 13      |
| 44         | 60    | 6       |
+------------+-------+---------+
Output: 
+------------+---------------+-------+--------------+----------------------+
| invoice_id | customer_name | price | contacts_cnt | trusted_contacts_cnt |
+------------+---------------+-------+--------------+----------------------+
| 44         | Alex          | 60    | 1            | 1                    |
| 55         | John          | 500   | 0            | 0                    |
| 66         | Bob           | 400   | 2            | 0                    |
| 77         | Alice         | 100   | 3            | 2                    |
| 88         | Alice         | 200   | 3            | 2                    |
| 99         | Bob           | 300   | 2            | 0                    |
+------------+---------------+-------+--------------+----------------------+


```sql
with temp as(
        select user_id,count(1) as contacts_cnt ,coalesce(max(customers_cnt),0) as  trusted_contacts_cnt from (
            select a.user_id,b.cnt  as customers_cnt from Contacts a left join (
                select user_id , count(1) as cnt from Contacts where contact_name in (select customer_name from Customers) group by user_id
            )b on a.user_id =b.user_id
    ) c  group by user_id
) 


select c.invoice_id  ,d.customer_name , c.price,  c.contacts_cnt , c.trusted_contacts_cnt from (
    select a.user_id, invoice_id, "" as customer_name, price, coalesce(contacts_cnt,0) as contacts_cnt, coalesce(trusted_contacts_cnt,0) as trusted_contacts_cnt  from Invoices a left join temp b on a.user_id = b.user_id
) c inner join Customers d on c.user_id = d.customer_id
order by invoice_id
```



1369. Get the Second Most Recent Activity
每个用户时间上第二近的出行计划
如果只有一个出行计划  , 就显示那一个

其实很简单 半天没混沌过来
犯的错误:
先是 要查出来endDate 然后 直接 in  ,忽略了username
然后是 查出no=1  然后用的no的starttime

分组, 编号 然后 用lead 查出来每一条的下一组 然后no=1的条目 的lead 如果为null 就用 当前的enddate



```sql
select c.* from UserActivity c inner join (
    select *,case when next_date is not null then next_date else endDate end as endDate1 from (
        select *,lead(endDate, 1,null) over(partition by username order by endDate desc) as next_date from (
            select *, row_number() over(partition by username order by endDate desc) as no from UserActivity 
        )a 
    )b where no =1
) d on c.username = d.username and c.endDate = d.endDate1
```


1384. Total Sales Amount by Year

根据时间范围把不同的时间算到不同的年里面去, 然后对每年进行统计
注意年显示2018 -2020  是固定的不用自己解
拿时间窗口去抠 就行, 抠出来每年的时间段

Product table:
+------------+--------------+
| product_id | product_name |
+------------+--------------+
| 1          | LC Phone     |
| 2          | LC T-Shirt   |
| 3          | LC Keychain  |
+------------+--------------+
Sales table:
+------------+--------------+-------------+---------------------+
| product_id | period_start | period_end  | average_daily_sales |
+------------+--------------+-------------+---------------------+
| 1          | 2019-01-25   | 2019-02-28  | 100                 |
| 2          | 2018-12-01   | 2020-01-01  | 10                  |
| 3          | 2019-12-01   | 2020-01-31  | 1                   |
+------------+--------------+-------------+---------------------+
Output: 
+------------+--------------+-------------+--------------+
| product_id | product_name | report_year | total_amount |
+------------+--------------+-------------+--------------+
| 1          | LC Phone     |    2019     | 3500         |
| 2          | LC T-Shirt   |    2018     | 310          |
| 2          | LC T-Shirt   |    2019     | 3650         |
| 2          | LC T-Shirt   |    2020     | 10           |
| 3          | LC Keychain  |    2019     | 31           |
| 3          | LC Keychain  |    2020     | 31           |
+------------+--------------+-------------+--------------+



```sql
with temp as(
    select * from(
        (select product_id,average_daily_sales,greatest('2018-01-01',period_start) as period_start, least('2018-12-31',period_end)as period_end from Sales)
        union
        (select product_id,average_daily_sales,greatest('2019-01-01',period_start) as period_start, least('2019-12-31',period_end)as period_end from Sales)
        union
        (select product_id,average_daily_sales,greatest('2020-01-01',period_start) as period_start, least('2020-12-31',period_end)as period_end from Sales)
    ) a
    where period_start <= period_end
)


select a.product_id , 
    b.product_name , 
    date_format(a.period_start,"%Y")as report_year , 
    (datediff(period_end,period_start)+1) *a.average_daily_sales  as total_amount 
        from temp a inner join Product  b on a.product_id = b.product_id
        
order by product_id ,report_year
```


1393. Capital Gain/Loss
股票买卖一定配对
算盈利
买卖各分组标号,  然后拿标号 join



```sql
select stock_name , sum(diff) as capital_gain_loss from (
    select a.* , (b.price - a.price) as diff from
    (select *, row_number() over(partition by stock_name order by operation_day) as no from Stocks where operation = 'Buy') a
    inner join
    (select *, row_number() over(partition by stock_name order by operation_day) as no from Stocks where operation = 'Sell')b
    on a.no = b.no and a.stock_name = b.stock_name
)c group by stock_name
```

如果不配对 怎么办

这个解法慢一些 但是如果不配对也能对
把buy 的price 改成 负的 , 然后分组 累加


```sql
    select stock_name , s as capital_gain_loss from (
    select stock_name,s, row_number() over(partition by stock_name order by operation_day desc) as no from (
        select stock_name,operation_day, sum(price) over(partition by stock_name order by operation_day) as s from 
        (
        select stock_name , operation , operation_day , 
            case when operation ='buy' then -1*price else price end as price 
        from Stocks
        ) a
    )b
        )c where no =1
```


1398. Customers Who Bought Products A and B but Not C
Input: 
Customers table:
+-------------+---------------+
| customer_id | customer_name |
+-------------+---------------+
| 1           | Daniel        |
| 2           | Diana         |
| 3           | Elizabeth     |
| 4           | Jhon          |
+-------------+---------------+
Orders table:
+------------+--------------+---------------+
| order_id   | customer_id  | product_name  |
+------------+--------------+---------------+
| 10         |     1        |     A         |
| 20         |     1        |     B         |
| 30         |     1        |     D         |
| 40         |     1        |     C         |
| 50         |     2        |     A         |
| 60         |     3        |     A         |
| 70         |     3        |     B         |
| 80         |     3        |     D         |
| 90         |     4        |     C         |
+------------+--------------+---------------+
Output: 
+-------------+---------------+
| customer_id | customer_name |
+-------------+---------------+
| 3           | Elizabeth     |
+-------------+---------------+


所有A B C 这几种里面 包含AB 但是不包含C的 customer

还有一种方法是 AB C 各给权重 
AB是1 C 是-1 其他是0  然后判断

```sql
    select * from Customers where customer_id in(
        select customer_id from (
                select customer_id, group_concat(distinct product_name) as grouname from Orders a where  product_name in('A','B','C') group by customer_id
            ) b where grouname in('A,B','B,A')
    )
```


1412. Find the Quiet Students in All Exams
找出所有的quit student  , 就是至少参加过一场考试,在任何考试中都既不得第一名 也不得倒数第一


Student table:
+-------------+---------------+
| student_id  | student_name  |
+-------------+---------------+
| 1           | Daniel        |
| 2           | Jade          |
| 3           | Stella        |
| 4           | Jonathan      |
| 5           | Will          |
+-------------+---------------+
Exam table:
+------------+--------------+-----------+
| exam_id    | student_id   | score     |
+------------+--------------+-----------+
| 10         |     1        |    70     |
| 10         |     2        |    80     |
| 10         |     3        |    90     |
| 20         |     1        |    80     |
| 30         |     1        |    70     |
| 30         |     3        |    80     |
| 30         |     4        |    90     |
| 40         |     1        |    60     |
| 40         |     2        |    70     |
| 40         |     4        |    80     |
+------------+--------------+-----------+
Output: 
+-------------+---------------+
| student_id  | student_name  |
+-------------+---------------+
| 2           | Jade          |
+-------------+---------------+

找出得过第一或倒数第一的, 把这些排除, 然后把没考过试的排除

```sql
select * from Student where student_id not in (
    select distinct student_id from (
        select student_id,
            rank() over(partition by exam_id  order by score) as no_asc,
            rank() over(partition by exam_id order by score desc) as no_desc
            from Exam
        )a  where no_asc =1 or no_desc =1
)  and student_id in (select distinct student_id from Exam)
order by student_id
```


1435. Create a Session Bar Chart
根据持续时间分类统计个数
不应该group  by
应该 各自查出来 然后union

Sessions table:
+-------------+---------------+
| session_id  | duration      |
+-------------+---------------+
| 1           | 30            |
| 2           | 199           |
| 3           | 299           |
| 4           | 580           |
| 5           | 1000          |
+-------------+---------------+
Output: 
+--------------+--------------+
| bin          | total        |
+--------------+--------------+
| [0-5>        | 3            |
| [5-10>       | 1            |
| [10-15>      | 0            |
| 15 or more   | 1            |
+--------------+--------------+

```sql
select '[0-5>' as bin, count(session_id ) as total from Sessions where duration/60 <5
union
select '[5-10>' as bin, count(session_id ) as total from Sessions where duration/60 between 5 and 10
union
select '[10-15>' as bin, count(session_id ) as total from Sessions where duration/60 between 10 and 15
union
select '15 or more' as bin, count(session_id ) as total from Sessions where duration/60 > 15
```

1440. Evaluate Boolean Expression
Variables table:
+------+-------+
| name | value |
+------+-------+
| x    | 66    |
| y    | 77    |
+------+-------+
Expressions table:
+--------------+----------+---------------+
| left_operand | operator | right_operand |
+--------------+----------+---------------+
| x            | >        | y             |
| x            | <        | y             |
| x            | =        | y             |
| y            | >        | x             |
| y            | <        | x             |
| x            | =        | x             |
+--------------+----------+---------------+
Output: 
+--------------+----------+---------------+-------+
| left_operand | operator | right_operand | value |
+--------------+----------+---------------+-------+
| x            | >        | y             | false |
| x            | <        | y             | true  |
| x            | =        | y             | false |
| y            | >        | x             | true  |
| y            | <        | x             | false |
| x            | =        | x             | true  |
+--------------+----------+---------------+-------+

直接做出来
连续join 把value  join进去 然后判断

```sql
    select left_operand , operator , right_operand, case when value =1 then "true" else "false" end as value from (
        select 
        left_operand , operator , right_operand,
        case when operator ='>' then lv > rv
            when operator ='<' then lv < rv
            when operator ='=' then lv = rv end as value
            
        from (
            select a.*,b.value as lv, c.value as rv from Expressions a inner join Variables b on a.left_operand =b.name inner join Variables c on a.right_operand =c.name
        ) d
    )e
```

1454. Active Users
选出连续5天登陆的用户

Accounts table:
+----+----------+
| id | name     |
+----+----------+
| 1  | Winston  |
| 7  | Jonathan |
+----+----------+
Logins table:
+----+------------+
| id | login_date |
+----+------------+
| 7  | 2020-05-30 |
| 1  | 2020-05-30 |
| 7  | 2020-05-31 |
| 7  | 2020-06-01 |
| 7  | 2020-06-02 |
| 7  | 2020-06-02 |
| 7  | 2020-06-03 |
| 1  | 2020-06-07 |
| 7  | 2020-06-10 |
+----+------------+

###   注意 5的话 要-4 , 低分也是4

```sql
select * from Accounts where id in(
    select distinct id from (
        select  *, lag(login_date ,4,'1970-01-01') over(partition by id  order by login_date) as prev from (
            select distinct id , login_date from Logins
        )e 
    ) a where datediff(login_date,prev) =4
)  order by id
```

1459. Rectangles Area

没啥好看的做记录
点两两形成矩阵的面积大小 

Points table:
+----------+-------------+-------------+
| id       | x_value     | y_value     |
+----------+-------------+-------------+
| 1        | 2           | 7           |
| 2        | 4           | 8           |
| 3        | 2           | 10          |
+----------+-------------+-------------+
Output: 
+----------+-------------+-------------+
| p1       | p2          | area        |
+----------+-------------+-------------+
| 2        | 3           | 4           |
| 1        | 2           | 2           |
+----------+-------------+-------------+

```sql
select distinct * from (
    select 
    case when a.id < b.id then a.id
    else b.id end as p1,

    case when a.id > b.id then a.id
    else b.id end as p2,
    
     abs(a.x_value - b.x_value) * abs(a.y_value - b.y_value)   as area  from Points a inner join Points b on a.id != b.id
) c where area !=0
order by area desc ,p1,p2
```
Example 1:
打电话时间大于总体平均值的 郭嘉 
substr

Person table:
+----+----------+--------------+
| id | name     | phone_number |
+----+----------+--------------+
| 3  | Jonathan | 051-1234567  |
| 12 | Elvis    | 051-7654321  |
| 1  | Moncef   | 212-1234567  |
| 2  | Maroua   | 212-6523651  |
| 7  | Meir     | 972-1234567  |
| 9  | Rachel   | 972-0011100  |
+----+----------+--------------+
Country table:
+----------+--------------+
| name     | country_code |
+----------+--------------+
| Peru     | 051          |
| Israel   | 972          |
| Morocco  | 212          |
| Germany  | 049          |
| Ethiopia | 251          |
+----------+--------------+
Calls table:
+-----------+-----------+----------+
| caller_id | callee_id | duration |
+-----------+-----------+----------+
| 1         | 9         | 33       |
| 2         | 9         | 4        |
| 1         | 2         | 59       |
| 3         | 12        | 102      |
| 3         | 12        | 330      |
| 12        | 3         | 5        |
| 7         | 9         | 13       |
| 7         | 1         | 3        |
| 9         | 7         | 1        |
| 1         | 7         | 7        |
+-----------+-----------+----------+

```sql
with temp as(
select id,a.name as person,b.name as country from Person a inner join Country b on  substr(a.phone_number,1,3) = b.country_code
)


select country from (
select avg(duration) as duration, country from(
    (select duration, country from  Calls a inner join temp b  on a.caller_id = b.id)
     union all
    (select duration, country from  Calls a inner join temp b  on a.callee_id = b.id)
) cte
group by country
) d where duration > (select avg(duration) from Calls)
```

1479. (hard) Sales by Day of the Week
Orders table:
+------------+--------------+-------------+--------------+-------------+
| order_id   | customer_id  | order_date  | item_id      | quantity    |
+------------+--------------+-------------+--------------+-------------+
| 1          | 1            | 2020-06-01  | 1            | 10          |
| 2          | 1            | 2020-06-08  | 2            | 10          |
| 3          | 2            | 2020-06-02  | 1            | 5           |
| 4          | 3            | 2020-06-03  | 3            | 5           |
| 5          | 4            | 2020-06-04  | 4            | 1           |
| 6          | 4            | 2020-06-05  | 5            | 5           |
| 7          | 5            | 2020-06-05  | 1            | 10          |
| 8          | 5            | 2020-06-14  | 4            | 5           |
| 9          | 5            | 2020-06-21  | 3            | 5           |
+------------+--------------+-------------+--------------+-------------+
Items table:
+------------+----------------+---------------+
| item_id    | item_name      | item_category |
+------------+----------------+---------------+
| 1          | LC Alg. Book   | Book          |
| 2          | LC DB. Book    | Book          |
| 3          | LC SmarthPhone | Phone         |
| 4          | LC Phone 2020  | Phone         |
| 5          | LC SmartGlass  | Glasses       |
| 6          | LC T-Shirt XL  | T-Shirt       |
+------------+----------------+---------------+
Output: 
+------------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
| Category   | Monday    | Tuesday   | Wednesday | Thursday  | Friday    | Saturday  | Sunday    |
+------------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+
| Book       | 20        | 5         | 0         | 0         | 10        | 0         | 0         |
| Glasses    | 0         | 0         | 0         | 0         | 5         | 0         | 0         |
| Phone      | 0         | 0         | 5         | 1         | 0         | 0         | 10        |
| T-Shirt    | 0         | 0         | 0         | 0         | 0         | 0         | 0         |
+------------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+


也是一种转置题
group by 和 sum(if) 
或者no  group by no max(if)
转置题都是条件汇总

```sql
with temp as(
    select 
    item_category as Category,
    sum(if(dayname_='Monday',qty, 0)) as 'Monday',
    sum(if(dayname_='Tuesday',qty, 0)) as 'Tuesday',
    sum(if(dayname_='Wednesday',qty, 0)) as 'Wednesday',
    sum(if(dayname_='Thursday',qty, 0)) as 'Thursday',
    sum(if(dayname_='Friday',qty, 0)) as 'Friday',
    sum(if(dayname_='Saturday',qty,0)) as 'Saturday',
    sum(if(dayname_='Sunday',qty, 0)) as 'Sunday'
    from 
    (
        select b.item_category,dayname(a.order_date) as dayname_, sum(quantity) as qty from  Orders a inner join Items b on a.item_id = b.item_id  group by  b.item_category,dayname(a.order_date)
    )c  group by  item_category
)


(select * from temp)
union 
(select item_category as Category,
    0 as 'Monday',
    0 as 'Tuesday',
    0 as 'Wednesday',
    0 as 'Thursday',
    0 as 'Friday',
    0 as 'Saturday',
    0 as 'Sunday'

from Items where  item_category not in (select Category from temp)
)
order by Category
```


1511. Customer Order Frequency
easy题纯记录
在2020年7,8 月
Customers table:
+--------------+-----------+-------------+
| customer_id  | name      | country     |
+--------------+-----------+-------------+
| 1            | Winston   | USA         |
| 2            | Jonathan  | Peru        |
| 3            | Moustafa  | Egypt       |
+--------------+-----------+-------------+
Product table:
+--------------+-------------+-------------+
| product_id   | description | priceG       |
+--------------+-------------+-------------+
| 10           | LC Phone    | 300         |
| 20           | LC T-Shirt  | 10          |
| 30           | LC Book     | 45          |
| 40           | LC Keychain | 2           |
+--------------+-------------+-------------+
Orders table:
+--------------+-------------+-------------+-------------+-----------+
| order_id     | customer_id | product_id  | order_date  | quantity  |
+--------------+-------------+-------------+-------------+-----------+
| 1            | 1           | 10          | 2020-06-10  | 1         |
| 2            | 1           | 20          | 2020-07-01  | 1         |
| 3            | 1           | 30          | 2020-07-08  | 2         |
| 4            | 2           | 10          | 2020-06-15  | 2         |
| 5            | 2           | 40          | 2020-07-01  | 10        |
| 6            | 3           | 20          | 2020-06-24  | 2         |
| 7            | 3           | 30          | 2020-06-25  | 2         |
| 9            | 3           | 30          | 2020-05-08  | 3         |
+--------------+-------------+-------------+-------------+-----------+


```sql
select customer_id, name from Customers where customer_id in (
select customer_id   from (
select  customer_id ,month,sum(quantity* price)as spend  from (
select sum(quantity) as quantity, date_format(order_date, "%Y-%m") as month , product_id,  customer_id from Orders where order_date >='2020-06-01' and order_date < '2020-08-01' group by date_format(order_date, "%Y-%m"),product_id,customer_id
    ) a inner join Product b on a.product_id = b.product_id group by customer_id ,month
) d where spend >= 100 group by customer_id having count(1) >1
)
```


这个我不会写
having  sum(if) and sum(if)
```sql
select c.customer_id, c.name from Customers c
join Orders o on c.customer_id = o.customer_id
join Product p on o.product_id = p.product_id
where year(o.order_date) = 2020
group by c.customer_id
having sum(if(month(o.order_date)=6, o.quantity, 0) * price) >= 100
and sum(if(month(o.order_date)=7, o.quantity, 0) * price) >= 100;
```

1517. Find Users With Valid E-Mails
正则表达式 匹配邮箱
Users table:
+---------+-----------+-------------------------+
| user_id | name      | mail                    |
+---------+-----------+-------------------------+
| 1       | Winston   | winston@leetcode.com    |
| 2       | Jonathan  | jonathanisgreat         |
| 3       | Annabelle | bella-@leetcode.com     |
| 4       | Sally     | sally.come@leetcode.com |
| 5       | Marwan    | quarz#2020@leetcode.com |
| 6       | David     | david69@gmail.com       |
| 7       | Shapiro   | .shapo@leetcode.com     |
+---------+-----------+-------------------------+
Output: 
+---------+-----------+-------------------------+
| user_id | name      | mail                    |
+---------+-----------+-------------------------+
| 1       | Winston   | winston@leetcode.com    |
| 3       | Annabelle | bella-@leetcode.com     |
| 4       | Sally     | sally.come@leetcode.com |
+---------+-----------+-------------------------+



```sql
where mail regexp '^[a-zA-Z][a-zA-Z0-9_.-]*@leetcode[.]com$'
```

1527. Patients With a Condition
like 匹配
```sql
select * 
from patients 
where conditions like "DIAB1%" 
or 
conditions like "% DIAB1%"
```



1543. Fix Product Name Format

lcase 
trim

```sql
select lcase(trim(product_name)) as product_name, date_format(sale_date,'%Y-%m') as sale_date
, count(*) as total
from Sales
group by lcase(trim(product_name)), date_format(sale_date, '%Y-%m')
order by product_name, sale_date
```


1555. Bank Account Summary
medium 常规题
没卡住 记录
Users table:
+------------+--------------+-------------+
| user_id    | user_name    | credit      |
+------------+--------------+-------------+
| 1          | Moustafa     | 100         |
| 2          | Jonathan     | 200         |
| 3          | Winston      | 10000       |
| 4          | Luis         | 800         | 
+------------+--------------+-------------+
Transactions table:
+------------+------------+------------+----------+---------------+
| trans_id   | paid_by    | paid_to    | amount   | transacted_on |
+------------+------------+------------+----------+---------------+
| 1          | 1          | 3          | 400      | 2020-08-01    |
| 2          | 3          | 2          | 500      | 2020-08-02    |
| 3          | 2          | 1          | 200      | 2020-08-03    |
+------------+------------+------------+----------+---------------+

一通信用卡转账之后 最后每个用户的钱 是否低于0 
注意没有交易的用户也要算

```sql
with temp as(
select a.user_id,b.user_name, a.credit + b.credit  as credit,
case when  a.credit + b.credit >0 then "No" else "Yes" end  as credit_limit_breached
from(
    select  user_id, sum(amount) as credit from(
        (select paid_by as user_id ,amount*-1 as amount from Transactions)
        union all
        (select paid_to as user_id ,amount as amount from Transactions)
    ) b group by user_id
)
a inner join Users b on a.user_id = b.user_id
)


(select * from temp)
union all
(
select user_id,user_name,  credit,"No" as credit_limit_breached from Users where user_id not in (select user_id from temp)
)
```



1613. Find the Missing IDs
递归range

```sql
# Write your MySQL query statement below
with recursive cte as (
select 1 as VALUE 
 UNION 
 select VALUE + 1 from cte where value < (select max(customer_id) from customers)
)

select value as ids from cte where value not in (select customer_id from Customers)
```


```sql
select contest_id, round(count(distinct user_id) / (select count(1)  from Users),2) as percentage  from Users) from Register  group by contest_id order by percentage desc  ,contest_id
```


1635. Hopper Company Queries I
hard
要老命了 一屏幕代码
2020年当前某个月份活动的司机总数
首先把前面2021年的筛选出来 然后把小于2020么,的标记成 2020年一月
然后把没有的月份 挑出来作为0 
然后逐渐累加



Drivers table:
+-----------+------------+
| driver_id | join_date  |
+-----------+------------+
| 10        | 2019-12-10 |
| 8         | 2020-1-13  |
| 5         | 2020-2-16  |
| 7         | 2020-3-8   |
| 4         | 2020-5-17  |
| 1         | 2020-10-24 |
| 6         | 2021-1-5   |
+-----------+------------+
Rides table:
+---------+---------+--------------+
| ride_id | user_id | requested_at |
+---------+---------+--------------+
| 6       | 75      | 2019-12-9    |
| 1       | 54      | 2020-2-9     |
| 10      | 63      | 2020-3-4     |
| 19      | 39      | 2020-4-6     |
| 3       | 41      | 2020-6-3     |
| 13      | 52      | 2020-6-22    |
| 7       | 69      | 2020-7-16    |
| 17      | 70      | 2020-8-25    |
| 20      | 81      | 2020-11-2    |
| 5       | 57      | 2020-11-9    |
| 2       | 42      | 2020-12-9    |
| 11      | 68      | 2021-1-11    |
| 15      | 32      | 2021-1-17    |
| 12      | 11      | 2021-1-19    |
| 14      | 18      | 2021-1-27    |
+---------+---------+--------------+
AcceptedRides table:
+---------+-----------+---------------+---------------+
| ride_id | driver_id | ride_distance | ride_duration |
+---------+-----------+---------------+---------------+
| 10      | 10        | 63            | 38            |
| 13      | 10        | 73            | 96            |
| 7       | 8         | 100           | 28            |
| 17      | 7         | 119           | 68            |
| 20      | 1         | 121           | 92            |
| 5       | 7         | 42            | 101           |
| 2       | 4         | 6             | 38            |
| 11      | 8         | 37            | 43            |
| 15      | 8         | 108           | 82            |
| 12      | 8         | 38            | 34            |
| 14      | 1         | 90            | 74            |
+---------+-----------+---------------+---------------+
Output: 
+-------+----------------+----------------+
| month | active_drivers | accepted_rides |
+-------+----------------+----------------+
| 1     | 2              | 0              |
| 2     | 3              | 0              |
| 3     | 4              | 1              |
| 4     | 4              | 0              |
| 5     | 5              | 0              |
| 6     | 5              | 1              |
| 7     | 5              | 1              |
| 8     | 5              | 1              |
| 9     | 5              | 0              |
| 10    | 6              | 0              |
| 11    | 6              | 2              |
| 12    | 6              | 1              |
+-------+----------------+----------------+
```sql
with recursive temp as(
    select 1 as value
    union 
    select value+1 from temp where value <12
),

drivers1 as (
select  driver_id , 
case when join_date < '2020-01-01' then  1 
else month(join_date) end as month
from Drivers where  join_date<='2020-12-31'
),

drivers2 as(
    select driver_id,month, cnt from (
        select driver_id,month,sum(c) over(order by  month) as cnt from (
            (select driver_id, month ,count(1) as c  from drivers1 group by month)
            union all
            (
                select 0 as driver_id,  value as month ,0 as c  from temp where value not in (select month  from drivers1)
            )
        ) a 
    ) b order by month
),

acrides as(
    select month(requested_at) as month, count(1) as accepted_rides from AcceptedRides a inner join Rides b on a.ride_id = b.ride_id where year(b.requested_at) = 2020 group by month(requested_at)
)




select a.month,a.cnt as active_drivers,coalesce(b.accepted_rides,0) as accepted_rides from drivers2 a left join acrides b on a.month =b.month order by a.month;

```

1645. Hopper Company Queries II
hard 前面那道题一样的背景
主要还是把driver的累计 起来 ,后面都是添头
```sql
# Write your MySQL query statement below
with recursive temp as(
    select 1 as value
    union 
    select value+1 from temp where value <12
),

drivers1 as (
select  driver_id , 
case when join_date < '2020-01-01' then  1 
else month(join_date) end as month
from Drivers where  join_date<='2020-12-31'
),

drivers2 as(
    select driver_id,month, cnt from (
        select driver_id,month,sum(c) over(order by  month) as cnt from (
            (select driver_id, month ,count(1) as c  from drivers1 group by month)
            union all
            (
                select 0 as driver_id,  value as month ,0 as c  from temp where value not in (select month  from drivers1)
            )
        ) a 
    ) b order by month
)
select a.month,coalesce(round(coalesce(b.cnt,0) /a.cnt*100,2),0) as working_percentage  from drivers2 a left join (
select count(distinct a.driver_id) as  cnt  ,month(requested_at) month from AcceptedRides a inner join Rides b on a.ride_id = b.ride_id where year(requested_at)=2020  group by month(requested_at)
    ) b on a.month= b.month order by a.month
```

1651. Hopper Company Queries III
和上面一样的表
每三个月求平均值
1. 先把各明细按月加起来
2. 然后补充缺失的月份
```sql
with recursive seq as(
    (select 1 as value )
    union
    select value+1 from seq where value <12
),
temp as(-- 先把各明细按月加起来 
    select 
    month(requested_at) as month,
    sum(ride_distance) as ride_distance,  
    sum(ride_duration) as ride_duration

    from AcceptedRides a inner join  Rides b on a.ride_id = b.ride_id where year(requested_at) =2020 group by month(requested_at)
),

temp1 as( -- 然后补充缺失的月份
(select month,ride_distance,ride_duration  from temp )
union all 
(select value as month ,0 as ride_distance, 0 as ride_duration from seq where value not in ((select month from temp )))
order by month
)


select month-2 as month , round((r1+r2+ride_distance)/3,2) as average_ride_distance, round((d1+d2+ride_duration)/3,2) as average_ride_duration from (
    select 
    month, 
    lag(ride_distance,2,0) over(order by month) as r2,
    lag(ride_distance,1,0) over(order by month) as r1,
    ride_distance, 


    lag(ride_duration,2,0) over(order by month) as d2,
    lag(ride_duration,1,0) over(order by month) as d1,
    ride_duration


    from temp1
)a where month >2
```

1667. Fix Names in a Table

字符串操作函数 
upper lower length

```sql
select user_id, concat(
    upper(
        substr(name,1,1)
        ),
    lower(
        substr(name,2,length(name))
    )
    )as name from Users order by user_id;
```


1709. Biggest Window Between Visits
UserVisits table:
+---------+------------+
| user_id | visit_date |
+---------+------------+
| 1       | 2020-11-28 |
| 1       | 2020-10-20 |
| 1       | 2020-12-3  |
| 2       | 2020-10-5  |
| 2       | 2020-12-9  |
| 3       | 2020-11-11 |
+---------+------------+
Output: 
+---------+---------------+
| user_id | biggest_window|
+---------+---------------+
| 1       | 39            |
| 2       | 65            |
| 3       | 51            |
+---------+---------------+

每个用户时间间隔的最大值
当前一天是2021-1-1
```sql
with temp as(
    select distinct user_id , visit_date from(
        (select * from UserVisits )
        union all
        (
         select distinct user_id  ,'2021-1-1' as visit_date  from UserVisits 
        )
    )a order by user_id , visit_date
)
select user_id, diff as biggest_window from (
select *, row_number()  over(partition by user_id order by diff desc)as no from(
    select *, datediff(visit_date, prev_date) as diff from(
        select *, lag(visit_date ,1, null) over(partition by user_id order by visit_date) as prev_date from temp 
    )a
) b
)c where no =1 order by user_id
```


1747. Leetflex Banned Accounts

查处使用不同的ip地址在不同的地方登陆的时间范围有交集的用户

Input: 
LogInfo table:
+------------+------------+---------------------+---------------------+
| account_id | ip_address | login               | logout              |
+------------+------------+---------------------+---------------------+
| 1          | 1          | 2021-02-01 09:00:00 | 2021-02-01 09:30:00 |
| 1          | 2          | 2021-02-01 08:00:00 | 2021-02-01 11:30:00 |
| 2          | 6          | 2021-02-01 20:30:00 | 2021-02-01 22:00:00 |
| 2          | 7          | 2021-02-02 20:30:00 | 2021-02-02 22:00:00 |
| 3          | 9          | 2021-02-01 16:00:00 | 2021-02-01 16:59:59 |
| 3          | 13         | 2021-02-01 17:00:00 | 2021-02-01 17:59:59 |
| 4          | 10         | 2021-02-01 16:00:00 | 2021-02-01 17:00:00 |
| 4          | 11         | 2021-02-01 17:00:00 | 2021-02-01 17:59:59 |
+------------+------------+---------------------+---------------------+
Output: 
+------------+
| account_id |
+------------+
| 1          |
| 4          |
+------------+


自连接 join的时候可以指定其中一个更小
```sql
select 
    distinct a.account_id
from LogInfo a inner join LogInfo b on a.account_id = b.account_id and a.ip_address !=b.ip_address and a.login < b.login
where b.login <= a.logout
```

1767. Find the Subtasks That Did Not Execute

hard题 但是很快做出来 不难


任务表 的子任务数字表示从1到n
execute表示执行成功的
找出没有执行的

先把表1展开 
变成
1 1
1 2
1 3
2 1 
2 2
...
先利用递归弄出连续数
然后利用笛卡尔积连乘
得到上面的表
然后和表2排重

Tasks table:
+---------+----------------+
| task_id | subtasks_count |
+---------+----------------+
| 1       | 3              |
| 2       | 2              |
| 3       | 4              |
+---------+----------------+
Executed table:
+---------+------------+
| task_id | subtask_id |
+---------+------------+
| 1       | 2          |
| 3       | 1          |
| 3       | 2          |
| 3       | 3          |
| 3       | 4          |
+---------+------------+
Output: 
+---------+------------+
| task_id | subtask_id |
+---------+------------+
| 1       | 1          |
| 1       | 3          |
| 2       | 1          |
| 2       | 2          |
+---------+------------+

```sql
    with recursive seq as(
    (select 1 as value)
        union
        (select value +1 from seq where value < (select max(subtasks_count) from Tasks))
    )

    select task_id, subtask_id from (
        select b.task_id, value as subtask_id, concat(b.task_id,",",value)as cat from seq a  join Tasks b where a.value <= b.subtasks_count
        order by b.task_id 
    ) a where cat not in (select concat(task_id,",",subtask_id)from  Executed)
```


1843. Suspicious Bank Accounts
meduim
Input: 
Accounts table:
+------------+------------+
| account_id | max_income |
+------------+------------+
| 3          | 21000      |
| 4          | 10400      |
+------------+------------+
Transactions table:
+----------------+------------+----------+--------+---------------------+
| transaction_id | account_id | type     | amount | day                 |
+----------------+------------+----------+--------+---------------------+
| 2              | 3          | Creditor | 107100 | 2021-06-02 11:38:14 |
| 4              | 4          | Creditor | 10400  | 2021-06-20 12:39:18 |
| 11             | 4          | Debtor   | 58800  | 2021-07-23 12:41:55 |
| 1              | 4          | Creditor | 49300  | 2021-05-03 16:11:04 |
| 15             | 3          | Debtor   | 75500  | 2021-05-23 14:40:20 |
| 10             | 3          | Creditor | 102100 | 2021-06-15 10:37:16 |
| 14             | 4          | Creditor | 56300  | 2021-07-21 12:12:25 |
| 19             | 4          | Debtor   | 101100 | 2021-05-09 15:21:49 |
| 8              | 3          | Creditor | 64900  | 2021-07-26 15:09:56 |
| 7              | 3          | Creditor | 90900  | 2021-06-14 11:23:07 |
+----------------+------------+----------+--------+---------------------+
Output: 
+------------+
| account_id |
+------------+
| 3          |
+------------+


常规的分组标记, 连续两个月超出最大收入的可疑账户
lag 然后比较
# 注意 '2021-07' 不能用date_sub  format的时候要加上01 :"%Y-%m-01"

```sql
select distinct account_id  from (
    select a.* , 
    lag(month, 1,null) over(partition by a.account_id order by month) as prev_month,
    lag(amount, 1,null) over(partition by a.account_id order by month) as prev_amount,
    b.max_income
    from (
        select account_id,sum(amount) as amount , date_format(day, "%Y-%m-01") as month from Transactions where type ="Creditor"  group by account_id, date_format(day, "%Y-%m-01")
    ) a inner join Accounts b on a.account_id = b.account_id
) c where amount > max_income and prev_amount > max_income and prev_month = date_sub(month, interval 1 month)
```

1853. Convert Date Format
dayname("2022-04-12")  tuesday
monthname() = April
%e = %m 不过 没有前缀0

```sql
select Date_Format(D.Day, "%W, %M %e, %Y") as day
from 
Days
```


1892. Page Recommendations II
没做完 迪卡耳机

朋友喜欢的 自己不喜欢的推荐给自己

Friendship table:
+----------+----------+
| user1_id | user2_id |
+----------+----------+
| 1        | 2        |
| 1        | 3        |
| 1        | 4        |
| 2        | 3        |
| 2        | 4        |
| 2        | 5        |
| 6        | 1        |
+----------+----------+
Likes table:
+---------+---------+
| user_id | page_id |
+---------+---------+
| 1       | 88      |
| 2       | 23      |
| 3       | 24      |
| 4       | 56      |
| 5       | 11      |
| 6       | 33      |
| 2       | 77      |
| 3       | 77      |
| 6       | 88      |
+---------+---------+
Output: 
+---------+---------+---------------+
| user_id | page_id | friends_likes |
+---------+---------+---------------+
| 1       | 77      | 2             |
| 1       | 23      | 1             |
| 1       | 24      | 1             |
| 1       | 56      | 1             |
| 1       | 33      | 1             |
| 2       | 24      | 1             |
| 2       | 56      | 1             |
| 2       | 11      | 1             |
| 2       | 88      | 1             |
| 3       | 88      | 1             |
| 3       | 23      | 1             |
| 4       | 88      | 1             |
| 4       | 77      | 1             |
| 4       | 23      | 1             |
| 5       | 77      | 1             |
| 5       | 23      | 1             |
+---------+---------+---------------+
Explanation: 
```
```



1951. All the Pairs With the Maximum Number of Common Followers
最多的共同关注者

Relations table:
+---------+-------------+
| user_id | follower_id |
+---------+-------------+
| 1       | 3           |
| 2       | 3           |
| 7       | 3           |
| 1       | 4           |
| 2       | 4           |
| 7       | 4           |
| 1       | 5           |
| 2       | 6           |
| 7       | 5           |
+---------+-------------+
Output: 
+----------+----------+
| user1_id | user2_id |
+----------+----------+
| 1        | 7        |
+----------+----------+

思路:
 #   on a.follower_id =b.follower_id  where a.user_id < b.user_id

```sql
select  user1_id ,  user2_id from (
    select  user1_id ,  user2_id, rank()over(order by cnt desc)as no from (
        select a.user_id as user1_id, b.user_id as user2_id,count(1) as cnt   from Relations a inner join Relations  b on a.follower_id =b.follower_id  where a.user_id < b.user_id group by user1_id,user2_id 
    ) c 
) d where no =1
```

1972. First and Last Call On the Same Day
找出每天打的第一通电话和最后一通电话是给同一个人的用户


Calls table:
+-----------+--------------+---------------------+
| caller_id | recipient_id | call_time           |
+-----------+--------------+---------------------+
| 8         | 4            | 2021-08-24 17:46:07 |
| 4         | 8            | 2021-08-24 19:57:13 |
| 5         | 1            | 2021-08-11 05:28:44 |
| 8         | 3            | 2021-08-17 04:04:15 |
| 11        | 3            | 2021-08-17 13:07:00 |
| 8         | 11           | 2021-08-17 22:22:22 |
+-----------+--------------+---------------------+
Output: 
+---------+
| user_id |
+---------+
| 1       |
| 4       |
| 5       |
| 8       |
+---------+


1. 找出每个人都是给谁打的
2. 把被个人打电话按顺序标号 , 找出第一通和最后一通
3. 然后比较每通电话的conterpart 是否是一个人
其实逻辑很简单
坑:如果一个人只打了一通 打电话的对手有可能不止打一通, 所以不能把对手的号加进去


```sql
with add_day as(
        select *, date_format(call_time,"%Y-%m-%d")as day from Calls 
),
expand_user as(
select  recipient_id as caller_id , caller_id as recipient_id ,call_time ,day  from  add_day
    union 
select  caller_id as caller_id , recipient_id as recipient_id ,call_time ,day  from  add_day
),
first_day as (
    select *, row_number() over(partition by caller_id , day  order by call_time) as first_no from expand_user 
),
last_day as(
    select *, row_number() over(partition by caller_id , day  order by call_time desc) as last_no from expand_user 
),
temp as (
    select a.caller_id, a.recipient_id as p1, b.recipient_id as p2 from first_day a inner join last_day b on a.day = b.day and a.first_no = 1 and b.last_no =1 and a.caller_id = b.caller_id
)

select distinct caller_id as user_id from temp where  p1 = p2
```


1988. Find Cutoff Score for Each School

学校招生 找小于当前配额的最大值
如果最大值 不唯一 找最大值里面的最小值
无满足配额的要写-1
左链接是为了返回-1

先相乘 然后 分组编号 ,然后用group max 去除较大值
Schools table:
+-----------+----------+
| school_id | capacity |
+-----------+----------+
| 11        | 151      |
| 5         | 48       |
| 9         | 9        |
| 10        | 99       |
+-----------+----------+
Exam table:
+-------+---------------+
| score | student_count |
+-------+---------------+
| 975   | 10            |
| 966   | 60            |
| 844   | 76            |
| 749   | 76            |
| 744   | 100           |
+-------+---------------+
Output:
+-----------+-------+
| school_id | score |
+-----------+-------+
| 5         | 975   |
| 9         | -1    |
| 10        | 749   |
| 11        | 744   |
+-----------+-------+
```sql
select school_id, min(score) as score from (
    select school_id,
    case when score is not null then score else -1 end as score
    from (
        select a.*,b.score, rank() over(partition by a.school_id order by student_count desc) as no from Schools a left join Exam b on a.capacity >= b.student_count
    ) c where no =1
) d group by school_id
```

```
select bus_id ,passenger_cnt-last_cnt as   passengers_cnt from (
    select bus_id,  passenger_cnt, lag(passenger_cnt) over(order by arrival_time)as last_cnt from (
        select bus_id, count(1) over(order by a.arrival_time) as passenger_cnt ,a.arrival_time from Buses a  inner join Passengers b on a.arrival_time > b.arrival_time
    ) a 
) b order by bus_id
```