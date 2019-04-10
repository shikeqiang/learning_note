# 一、10道mysql查询语句面试题

## 表结构

- 学生表student(id,name)
- 课程表course(id,name)
- 学生课程表student_course(sid,cid,score)

## 建表语句：

```sql
create table student(
	id int unsigned primary key auto_increament,
    name char(20) not null
);
insert into student(name) values("小明"),("小红");

create table course(
id int unsigned primary key auto_increment,
name char(20) not null
);
insert into course(name) values('语文'),('数学');

create table student_course(
sid int unsigned,
cid int unsigned,
score int unsigned not null,
foreign key (sid) references student(id),
foreign key (cid) references course(id),
primary key(sid, cid)
);
insert into student_course values(1,1,80),(1,2,90),(2,1,90),(2,2,70);
```

## 问题

1.查询student表中重名的学生，结果包含id和name，按name,id升序

```sql
select id,name from student where name in (
	select name from student group by name having(count(*)>1)
) order by name;
```

我们经常需要查询某一列重复的行，一般通过group by(有重复的列)然后取count>1的值。 关系型数据库有他的局限性， 有些看似简单的查询写出来的sql很复杂，而且效率也会很低。

2.在student_course表中查询平均分不及格的学生，列出学生id和平均分

```sql
select sid,avg(score) as avg_score from student_course group by sid having(avg_score<60);
```

​	group by和having是最常考的。 **==where子句中不能用聚集函数作为条件表达式，但是having短语可以，where和having的区别在于对用对象不同，where作用于记录，having作用于组==**。

3.在student_course表中查询每门课成绩都不低于80的学生id

```sql
select sid from student_course where sid not in (
	select sid from student_course where score < 80
);
```

用到反向思想，其实就是数理逻辑中的`∀x:P和¬∃x:¬P`是等价的。

4.查询每个学生的总成绩，结果列出学生姓名和总成绩 **如果使用下面的sql会过滤掉没有成绩的人**

```sql
select name,sum(score) from student,student_course where student.id=student_score.sid group by sid;
```

更保险的做法应该是使用 **左外连接**

```sql
select name,sum(score) from student left join student_course on student.id=student_course.id group by sid;
```





