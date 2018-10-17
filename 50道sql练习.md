题目地址：https://www.jianshu.com/p/476b52ee4f1b

（1）查询" 01 "课程比" 02 "课程成绩高的学生的信息及课程分数

select * from sc,student where sc.SId=student.SId and
student.SId in(select sc2.sid sid from (
select sid,cid,score from sc where cid=01) sc1,(select 
sid,cid,score from sc where cid=02) 
sc2 where sc1.sid=sc2.sid and sc1.score>sc2.score)

修改sql

select * from student,(select  sc1.SId,sc1.class1 class1,sc2.class2 class2 from (select SId,score as class1 from sc where CId=01) as sc1,
(select SId,score as class2 from sc where CId=02) as sc2 where sc1.SId=sc2.SId
and sc1.class1>sc2.class2)as sc3 where sc3.SId=student.SId



（2）查询同时存在" 01 "课程和" 02 "课程的情况
select * from 
​    (select * from sc where sc.CId = '01') as t1, 
​    (select * from sc where sc.CId = '02') as t2
where t1.SId = t2.SId;



（3）查询存在" 01 "课程但可能不存在" 02 "课程的情况(不存在时显示为 null )
select * from
(select SId,CId,score from sc where sc.CId =01) as sc1
LEFT JOIN
-- 左链接影响右表，右链接影响左表，所谓影响就是当被影响的表的某一个数据不存在的时候显示null
-- table name left/right table name on 直接写条件
(select  SId,CId,score from sc where sc.CId=02) as sc2 
on sc1.SId=sc2.SId



（4）查询不存在" 01 "课程但存在" 02 "课程的情况

select * from sc where cid=02 and sid not in (select sid from sc where CId=01);



（5）查询平均成绩大于等于 60 分的同学的学生编号和学生姓名和平均成绩
select student.Sname,student.SId,sc1.avgscore from student,(
select AVG(score) as avgscore,sid from sc group  by sid )as sc1 where 
sc1.sid=student.SId  and sc1.avgscore>=60;

修改后的sql

select student.Sname,student.SId,sc1.avgscore from student,(
select AVG(score) as avgscore,sid from sc group  by sid HAVING avgscore>=60)as sc1 where 
sc1.sid=student.SId ;
-- 尽量内层限制返回条数减少二次查询时间

（6）查询在 SC 表存在成绩的学生信息
select SQL_NO_CACHE * from student where sid in (SELECT DISTINCT sid from sc);

select SQL_NO_CACHE DISTINCT student.*
from student,sc
where student.SId=sc.SId



（6）查询所有同学的学生编号、学生姓名、选课总数、所有课程的总成绩(没成绩的显示为 null )

select sc1.co,sc1.su,student.Sname,student.Sname,student.SId from (
select SId,count(*) as co,SUM(score) as su from sc GROUP BY SId) 
as sc1 RIGHT JOIN student on student.SId=sc1.SId ;

-- 缺失数据的表放置在前用右查询，缺失数据在后用左查询（右缺钱，左缺后）



（7）查有成绩的学生信息
select * from student where EXISTS(select  sc.score from sc where student.SId=sc.SId);

-- EXISTS 适合子查询表数据比外查询的数据大的情况，in先是把子查询中结果缓存然后挨个和外查询比较，exists走的是数据库
select * from student where SId in (select  DISTINCT SId from sc);



（8）查询「李」姓老师的数量
select count(*) from teacher where Tname like '李%'



（9）查询学过「张三」老师授课的同学的信息
select student.Sage,student.SId,student.Sname,student.Ssex from student,sc,(select course.cid as cid from course ,teacher where teacher.TId = course.TId and teacher.Tname='张三') as cd

where student.SId=sc.SId and sc.CId= cd.cid  

优化：

select * from course,sc,student,teacher where student.SId =sc.SId and sc.CId=course.CId and course.TId=teacher.TId
and teacher.Tname='张三';



（10）查询没有学全所有课程的同学的信息(存在一门课都没有学的学生)
select * from student where sid not in (
select sid from sc group by  sid HAVING count(cid) =( select count(*) from course))



（11）查询至少有一门课与学号为" 01 "的同学所学相同的同学的信息
 select DISTINCT student.Sname from student,sc, (select  cid from sc where sid='01') as s1 
where sc.CId=s1.cid and sc.SId= student.SId

多表关联：select DISTINCT student.Sname from student,sc s1,sc s2 where student.SId=s1.SId and s1.CId=s2.CId and s2.SId=01



（12）查询和" 01 "号的同学学习的课程完全相同的其他同学的信息 （只要有课程和01的重合就选出来，总数和01的相等就必定是和01是一样的）
select * from student where sid in (select sid from sc where cid in(select cid from sc where sid=01) group by 
SId HAVING count(cid)=(select count(cid) from sc where sid=01))



（13）查询没学过"张三"老师讲授的任一门课程的学生姓名（有存在没有上过任何一门课的情况）
select * from student where sid not in(select DISTINCT sid from sc where cid in(
SELECT cid from course,teacher where course.TId=teacher.TId and teacher.Tname='张三'
))

内部级联查询

select * from student where sid not in(select sid from sc,teacher,course where sc.CId=course.CId and 
course.TId=teacher.TId and teacher.Tname='张三');



（14）查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩
 select s.sco,student.SId,student.Sname from student,(select sid,avg(score) sco from sc where sc.score<60 
group by sid HAVING count(sc.score)>1)as s where s.sid= student.SId

 直接级联

select student.sid,avg(sc.score),student.Sname from student,sc where sc.SId=student.SId and 
sc.score<60 group by sc.SId HAVING count(sc.score)>1



（15）检索" 01 "课程分数小于 60，按分数降序排列的学生信息
select student.sid,sc.score,student.Sname from sc,student where sc.SId=student.SId and  sc.cid ='01' and sc.score<60 ORDER BY sc.score DESC 