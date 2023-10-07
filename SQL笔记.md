#### SQL必知必会

- 长型数据转宽型数据

  ```sql
  max(case when a.c_id = '01' then a.s_score else null end)
  ```

  其中max是统计函数，所以需要分组，也就是需要用到group by，类似的统计函数还有avg， count，sum

- 分组后的条件要写在having里面

- HAVING子句是用于对GROUP BY子句分组后的结果进行筛选的条件表达式。它在查询中与WHERE子句有些相似，但有一些关键的区别。

  通常情况下，WHERE子句在执行GROUP BY之前过滤行。它可以在FROM子句引用的表中应用谓词，并排除不满足条件的行。而HAVING子句则是在GROUP BY之后过滤结果集中的组。

  因此，当需要根据聚合函数（如SUM、COUNT、AVG等）的计算结果来筛选或筛除分组后的结果时，就需要使用HAVING子句。

  在你提供的SQL查询示例中，HAVING子句用于筛选平均成绩大于60的学生。GROUP BY a.s_id将结果按照学生ID分组，然后HAVING avg(a.s_score) > 60条件过滤出平均成绩大于60的组。

  总结起来，以下情况下需要使用HAVING子句进行条件查询：

  1. 当需要基于聚合函数的计算结果进行分组后的筛选时。
  2. 当需要对分组后的结果集中的组应用条件过滤时。

- 对于内连接（INNER JOIN），在使用WHERE子句指定连接条件时，可以省略关键词"INNER JOIN"。相反，可以直接使用逗号（,）将要连接的表列出。

- ```sql
  select
  	a.s_id,
  	b.s_name,
  	count(a.c_id) cnt_s,
  	sum(a.s_score) sum_s
  from
  	score a, student b
  where
  	a.s_id = b.s_id
  group by a.s_id;
  # 查询所有学生的学号，姓名，选课数，总分
  # 但是该查询选用的内连接，分数表只有01-07学号的学生选课分数，但是还有08学号的学习存在，可能该同学缺考，通过该查询后的最终数据会出现没有08学生的信息，如果需要该学生的记录的话，则只需要将内连接改成外连接，左连接是左边的表数据一定存在。但是还要注意将where改成on，并且最重要的是需要在 GROUP BY 子句中包括所有非聚合列，以便与聚合函数一起使用，也就是在group by后添加b.s_name
  select
  	b.s_id, # 如果这里选择a.s_id会出现为学号null的情况，所以改成所有从student表中取得数据，	
  	b.s_name,   	# 并且group by后的数据要和这里一样
  	count(a.c_id) cnt_s,
  	ifnull(sum(a.s_score),0) sum_s # ifnull 将null值转化为指定值
  from
  	score a
  right join
  	student b
  on
  	a.s_id = b.s_id
  group by b.s_id,b.s_name;
  ```

- 上面的平均值小于60也要注意这个问题，having比较的是ifnull(avg(a.s_score),0) < 60；

- 查询不存在条件的数据除了可以使用not in还可以使用not exits，并且后者的效率更高

- ```sql
  # 查询没学过张三老师授课的学生信息
  select
  	d.*
  from 
  	student d
  where
  	d.s_id not in
  (select
  	c.s_id
  from
  	teacher a, course b, score c
  where
  	a.t_id = b.t_id
  and
  	b.c_id = c.c_id
  and
  	a.t_name = '张三')
  	
  ####################################
  # 查询没学过张三老师授课的学生信息
  select
  	*
  from 
  	student
  where not exists(
  	select 1 from
  		(select
  			c.s_id
  		from
  			teacher a, course b, score c
  		where
  			a.t_id = b.t_id
  		and
  			b.c_id = c.c_id
  		and
  			a.t_name = '张三') t
  	where
  			t.s_id = student.s_id)
  在子查询中，使用SELECT 1是一种常见的做法，它实际上不会返回任何具体的数据。选择1是为了简化查询并提高性能。
  
  在这个上下文中，子查询的目的只是判断是否存在符合条件的记录，而不需要返回实际的数据。因此，选择1作为子查询的结果是一种惯用的做法，表示只需返回一个任意的非空值即可。
  
  相比于选择具体的列或使用*通配符来返回所有列，使用SELECT 1可以减少不必要的数据传输和处理，从而提高查询的性能。因为在这里我们只关心是否存在符合条件的记录，而不需要实际的数据内容。
  ```

- group by 1,2,3,4（1234代表列，也可以直接写列名）和 distinct都可以去重

- ```sql
  查询和"01"号的同学学习的课程完全相同的其他同学的信息 
  #首先用score表自连接也就是两个score 分别取别名s1 s2 连结条件是s1.c_id = s2.c_id 然后把s1表看做01号学生的成绩表也就是增加限制条件为 and s1.s_id = '01',把s2这张表看做除了01号学生的成绩表也就是and s2.s_id != '01',然后再进行分组汇总,把s2表的学生id进行分组后,加入having作为过滤条件,也就是想要达到s1 与s2课程id是一样的之后再加上 把学习课程的数量也进行计数相比较(因为01号学生学习了三门课程,having要过滤掉不等于三的),这样就获得了课程id一致,课程数量也一致的s2学生id了  ,最后和student表join 获得学生姓名
  select s2.s_id,student.s_name
  from score as s1 
  join score as s2 
  on s1.c_id = s2.c_id
  join student on student.s_id = s2.s_id
  and s1.s_id = '01'
  and s2.s_id != '01'
  group by s2.s_id,student.s_id
  having count(s2.c_id) = (select count(*) from score where s_id = '01')
  ```

- round（*，2）保留两位小数（四舍五入）

- ```sql
  # 按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩
  select
  	a1.*,a2.avg_s
  from
  	(select * from score) a1,
  	(select a.s_id,round(avg(a.s_score),2) avg_s from score a group by a.s_id) a2
  where
  	a1.s_id = a2.s_id
  order by
  	avg_s desc
  ############################################ （注意开窗函数在mysql8.0版本后才加入）
  # 开窗函数 开窗函数允许你在不破坏原来的表的字段的基础上进行聚合函数操作，如sum求和，count计数，avg求均值
  select
  	a.*,
  	round(avg(a.s_score) over(partition by a.s_id),2) avg_s
  from
  	score a
  order by
  	avg_s desc
  注意这里的round函数右括号必须包含在开窗函数外面，否则会报错
  ```

- sum(case when s_score >= 60 then 1 else 0 end) 通过该方法求及格个数

- ```sql
  # 按各科成绩进行排序，并显示排名 开窗函数运用 mysql8.0解法
  select
  	a.*,
  	rank() over(partition by c_id order by s_score desc) 排名
  from
  	score a
  # 按各科成绩进行排序，并显示排名 mysql5.7解法 子查询 原理见下方
  select
  	a.*,
  	(select count(s_score) from score b where b.c_id = a.c_id and a.s_score < b.s_score) + 1 排名
  from
  	score a
  order by
  	a.c_id, a.s_score desc
  ```

- 三种rank函数的区别

  1. rank() 1，1，3
  2. dense_rank() 1，1，2
  3. row_number 1，2，3，4

- ```sql
  # 查询学生的总成绩并进行排名
  select
  	t.*,rank() over(order by sum_s desc) rk
  from
  	(select
  		a.s_id,sum(s_score) sum_s
  	from
  		score a
  	group by
  		a.s_id) t
  ###################### 子查询详解
  select
  	a.*,
  	(select count(sum_s) from sum_s_temp b where a.sum_s < b.sum_s) + 1 rk
  from
  	sum_s_temp
  order by 
  	sum_s desc
  	
  原理就是子查询将副表b中的每一行数据与主表a的当前遍历的一行数据进行比较从而得到排名
  简单的理解就是查询表中比当前行大的数据有几行，这样就可以得到对应的排名，注意还要加1，因为排名是从1开始的
  ```

- 求每一门成绩都在70分以上的学生，思路：如果一个学生的最小成绩＞70，则每一门的成绩都在70分以上。

- having 条件后只能使用聚合函数的完整表达式，order by可以使用上面表达式的别名。

- 日期函数详解：

  1. now()
  2. year(),month(),day()
  3. dayofyear(),weekofyear()
  4. date_format(now(),'%m%d') 日期格式化
  5. str_to_date('190613','%y%m%d) 将不规范的日期字符串转为规范的日期

- concat函数 将两个字符串连接在一起

- 查询本周过生日的学生，思路，将其月份日期取出拼接当前年份，转为日期，过了多少周是否与当前时间过了多少周相等

- 注意！！！！！！重大知识点：在MySQL中，"Y"表示四位数的年份，而"y"表示两位数的年份。

- 对当前日期加上七天，now（） + interval ‘7’ day

- 注意上面解决的查询下一周过生日的学生，这里直接加一会出现大问题，所以不能简单的做加一操作就是下一周了。

- datediff 比较两个日期的差距时间 

  ```sql
  select
      a.id
  from
      Weather a, Weather b
  where
      datediff(a.recordDate, b.recordDate) = 1
  and
      a.temperature > b.temperature
  ```

- cross join 将两个表的每一行都连接在一起

  ```sql
  SELECT 
      *
  FROM
      Students s
  CROSS JOIN
      Subjects sub
  ```

- 

