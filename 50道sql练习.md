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
    (select * from sc where sc.CId = '01') as t1, 
    (select * from sc where sc.CId = '02') as t2
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