1. 最晚入职员工的所有信息

   `select * from employees` **`order by hire_date desc limit 1`**

   limit 和 offset 关键字

2. 查找入职员工时间排名倒数第三的员工所有信息

   `select * from employees order by hire_date` **`desc limit 1 offset 2`**

   limit 和 offset 关键字

3. 各个join范围（记得StackOverflow有一个图挺好），join的写法

   - 连续两个left join

     [查找所有员工的last_name和first_name以及对应的dept_name](https://www.nowcoder.com/practice/5a7975fabe1146329cee4f670c27ad55?tpId=82&tags=&title=&diffculty=0&judgeStatus=0&rp=1&ru=/ta/sql&qru=/ta/sql/question-ranking)

   ``` sql
   SELECT em.last_name, em.first_name, dp.dept_name
   FROM (employees AS em LEFT JOIN dept_emp AS de ON em.emp_no = de.emp_no)
   LEFT JOIN departments AS dp ON de.dept_no = dp.dept_no
   ```

   - 先join和后join的区别，多个连续join
   - join的性能
   - join经常出现
   - join题重要的是一步一步慢慢的导出结果

4. 聚合函数、group by 和 having 的用法和具体例子,having 后写聚合函数

   和之前的select部分的顺序

   - [获取所有部门中当前员工当前薪水最高的相关信息](https://www.nowcoder.com/practice/4a052e3e1df5435880d4353eb18a91c6?tpId=82&&tqId=29764&rp=1&ru=/ta/sql&qru=/ta/sql/question-ranking)
- [从titles表获取按照title进行分组，注意对于重复的emp_no进行忽略](https://www.nowcoder.com/practice/c59b452f420c47f48d9c86d69efdff20?tpId=82&tags=&title=&diffculty=0&judgeStatus=0&rp=1&ru=/ta/sql&qru=/ta/sql/question-ranking)`count(distinct emp_no) t `
  
5. with写法和子查询 性能原因和索引命中问题

6. 获取当前薪水**第二多**的员工的emp_no以及其对应的薪水salary

   第一种写法：`SELECT column FROM table ORDER BY column DESC LIMIT 1,1;`

   第二种写法：

   ```sql
   select SAL from EMPLOYEE E1 where 
    (N - 1) = (select count(distinct(SAL)) 
               from EMPLOYEE E2 
               where E2.SAL > E1.SAL )
   -- 举例如第二大的则
    select SAL from EMPLOYEE E1 where 
        (2 - 1) = (select count(distinct(SAL)) 
                   from EMPLOYEE E2 
                   where E2.SAL > E1.SAL )
                   
   -- 也可以写成自连接
   select E1.SAL from EMPLOYEE E1 join EMPLOYEE E2 on E1.SAL <= E2.SAL
   group by E1.SAL
   having count(distinct E2.SAL)=2
   ```

7. [查找员工编号emp_now为10001其自入职以来的薪水salary涨幅值growth](https://www.nowcoder.com/practice/c727647886004942a89848e2b5130dc2?tpId=82&&tqId=29772&rp=1&ru=/ta/sql&qru=/ta/sql/question-ranking)

   （特点是要找两个时间点），这种就分别找出两个时间点就可了

   ```sql
   SELECT ( 
   (SELECT salary FROM salaries WHERE emp_no = 10001 ORDER BY to_date DESC LIMIT 1) -
   (SELECT salary FROM salaries WHERE emp_no = 10001 ORDER BY to_date ASC LIMIT 1)
   ) AS growth
   ```

8. [对所有员工的当前薪水按照salary进行按照1-N的排名](https://www.nowcoder.com/practice/b9068bfe5df74276bd015b9729eec4bf?tpId=82&&tqId=29775&rp=1&ru=/ta/sql&qru=/ta/sql/question-ranking)

   难点在于输出每个员工的名次，MySQL没有计数功能，在MySQL中可以选择这么做：

   ```sql
   SELECT t.*, 
          @rownum := @rownum + 1 AS rank
     FROM YOUR_TABLE t, 
          (SELECT @rownum := 0) r
   ```

   但是有人说这么做好像不太合适，可以用表的连接的写法

   ``` sql
   SELECT s1.emp_no, s1.salary, COUNT(DISTINCT s2.salary) AS rank
   FROM salaries AS s1, salaries AS s2
   WHERE s1.to_date = '9999-01-01'  AND s2.to_date = '9999-01-01' AND s1.salary <= s2.salary
   GROUP BY s1.emp_no
   ORDER BY s1.salary DESC, s1.emp_no ASC
   ```

   关键在于 `s1.salary <= s2.salary` 这句，计数有多少个大于等于它，第1名应该是1个以此类推，`GROUP BY s1.emp_no` 表明计算哪个人的
   
9. 按分数段（0-100）输出学生人数

   - 使用分段`[100-85],[85-70],[70-60],[‹60]`来统计各科成绩，分别统计：各分数段人数，课程号和课程名称

   ``` sql
   select a.课程号,b.课程名称,
   sum(case when 成绩 between 85 and 100 then 1 else 0 end) as '[100-85]',
   sum(case when 成绩 >=70 and 成绩<85 then 1 else 0 end) as '[85-70]',
   sum(case when 成绩 >=60 and 成绩<70 then 1 else 0 end) as '[70-60]',
   sum(case when 成绩 <60 then 1 else 0 end) as '[<60]'
   from score as a RIGHT JOIN course as b
   on a.课程号=b.课程号
   GROUP BY a.课程号,b.课程名称;
   ```

10. 找出班级人数大于30人的班级还有人数

    ``` sql
    select classno,count(*) num 
    from student group by classno having count(*)>30;
    ```

11. 员工表有两列id，email。不同的id有相同email，想删除相同的email只保留id最小的那一行

    [196. 删除重复的电子邮箱](https://leetcode-cn.com/problems/delete-duplicate-emails/)

    ``` sql
    DELETE p2 FROM Person p1 JOIN Person p2
    ON p2.Email = p1.Email WHERE p2.Id > p1.Id;
    ```

12. 当前员工用(salaries.to_date='9999-01-01')来表示，学习一波，避免了额外字段

13. 门门课程都及格的学生

    ``` sql
    Select sno frome sc group by sno having(min(grade)>=60)
    ```

14. 既有课程大于90分又有课程不及格的学生

    ``` sql
    Select sno from sc where grade >90 and sno in (select sno from sc where grade<60)
    ```

15. 求各门课程去掉一个最高分和最低分后的平均分

    ``` sql
    SELECT CNO,avg(GRADE) from SC 
    where GRADE not in (select top 1 GRADE from SC order by GRADE) 
    and   GRADE not in (select top 1 GRADE from SC order by GRADE desc) 
    group by CNO;
    ```

16. 查询两门以上不及格课程的同学的学号及其平均成绩

    ``` sql
    select 学号,avg(成绩) as 平均成绩
    from score
    where 成绩<60
    group by 学号
    having count(课程号)>=2
    ```

17. 统计每门课程的学生选修人数(超过2人的课程才统计)，要求输出课程号和选修人数，查询结果按人数降序排序，若人数相同，按课程号升序排序

    ```sql
    select 课程号,count(*)as 选修人数 
    from score 
    group by 课程号 
    having count(*)>2 
    ORDER BY 选修人数 desc,课程号 asc
    ```

18. 查询出每门课程的及格人数和不及格人数

    ```sql
    select 课程号,
    sum(case when 成绩>=60 then 1 else 0 end) as 及格人数,
    sum(case when 成绩<60 then 1 else 0 end) as 不及格人数
    from score
    group by 课程号
    ```

19. 

