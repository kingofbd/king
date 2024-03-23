此文档是关于HiveSQL的一些常见常用的练习题。分初中高分别进行说明。

首先准备数据如下，在hive中创建表。建表语句这里就省略了，直接给出相关表的结构说明。

学生表student_info

![img.png](img/img.png)

教师表teacher_info

![img_1.png](img/img_1.png)

课程表course_info

![img_2.png](img/img_2.png)

分数表score_info

![img_3.png](img/img_3.png)

# 基础查询

## 简单查询

### 查询姓名中带“山”字的学生名单

比较简单，考察的是where过滤，配合like模糊查询即可。语句如下

```sql
select * from student_info where stu_name like '%山%';
```

### 查询姓“王”老师的个数

这题考察的是where条件过滤，配合like模糊查询，然后使用count进行计数，语句如下

```sql
select count(1) from teacher_info where tea_name like '王%';
```

### 查询课程编号为“04”且分数小于60的学生的分数信息，结果按照分数降序排列

此题考察的是where条件过滤，然后配合order by实现排序。语句如下

```sql
select * from score_info where course_id='04' and score<60 order by score desc;
```

### 查询数学成绩不合格的学生信息，及对应的数学学科成绩，按照学号升序排序

这个题可以拆分出来好几步，然后整合起来即可。步骤如下：

1. 根据数学这个学科，查询课程表，得到数学学科对应的课程id
2. 根据上一步查询到的数学课程id，以及成绩不合格，作为过滤条件，查询分数表，得到学生id和数学分数
3. 前两步得到的结果，和学生信息表进行join，将学生的详细信息填充到结果中

```sql
select
    t1.*,
    t2.score    
from student_info t1
    join
(select
    stu_id,
    score
from score_info
where course_id=(select course_id from course_info where course_name='数学') and score<60) t2
on t1.stu_id=t2.stu_id
order by t1.stu_id asc;
```

## 分组与汇总

### 查询编号为“02”的课程的总成绩

查询分数表，过滤出课程编号为02的课程，然后按照课程编号进行分组（实际上就只有一个组），然后对组内的分数进行sum求和即可。

```sql
select
    course_id,
    sum(score)
from score_info
where course_id='02'
group by course_id;
```

### 查询参加考试的学生的个数

查询分数表，对参与考试的学生进行计数，但是考虑到一个学生可能会参加多门考试，所以要对结果进行去重。综合下来，可以使用count(distinct)

```sql
select count(distinct stu_id) from score_info;
```

### 查询各科成绩最高分和最低分，以如下形式展示：课程号、最高分、最低分

查询分数表，按照课程id进行分组，通过max求最高分，通过min求最低分。

```sql
select course_id,max(score),min(score) from score_info group by course_id;
```

### 查询每门课程有多少学生参加了考试（有考试成绩）

查询分数表，按照课程id进行分组，然后对学生编号stu_id字段进行求count计数即可。

```sql
select course_id,count(stu_id) from score_info group by course_id;
```

### 查询男生，女生的人数

查询学生表，按照性别sex进行分组，然后对学生编号stu_id进行求count计数即可

```sql
select sex,count(stu_id) from student_info group by sex;
```

### 查询平均成绩大于60分的学生的学号和平均成绩

查询分数表，按照学生编号stu_id进行分组，然后对score分数求avg平均值，然后对求出来的平均值使用having子句进行过滤，过滤出平均成绩大于60分的。

```sql
select
    stu_id,
    avg(score) as avg_score
from score_info
grouy by stu_id
having avg_score>60;
```

### 查询至少考了4门课程的学生学号

查询分数表，按照学生编号stu_id进行分组，然后对课程编号course_id进行count计数统计，对统计出来的聚合结果，使用having子句进行过滤筛选，筛选出总数大于等于4的即可。

```sql
select
    stu_id,
    count(course_id) as ct_course_id
from score_info
group by stu_id
having ct_course_id>=4;
```

### 查询每门课程的平均成绩，结果按照平均成绩升序排序，当平均成绩相同时，按课程号降序排序

查询分数表，按照课程id进行分组，对score分数求平均值，然后在order by子句中，对平均成绩升序排，对课程号降序排。

```sql
select
    course_id,
    avg(score) as avg_score
from score_info
group by course_id
order by avg_score asc,course_id desc;
```

### 统计参加考试的人数大于等于15的学科

查询分数表，按照课程id进行分组，对学生编号进行count计数统计，然后对统计之后的结果使用having子句进行过滤。

```sql
select
    course_id,
    count(stu_id) as ct_stu_id
from score_info
group by course_id
having ct_stu_id>=15;
```

### 查询学生的总成绩并按照总成绩降序排序

查询分数表，按照学生id进行分组，对score分数进行sum统计，然后对统计之后的结果使用order by降序排序即可。

```sql
select
    stu_id,
    sum(score) as sum_score
from score_info
group by stu_id
order by sum_score desc;
```

### 查询一共参加三门课程考试且其中一门为语文课程的学生的id和姓名

此题较为复杂，需要多个步骤综合执行得到结果。具体的分解步骤如下

* 首先通过查询课程表，获得课程名称为“语文”的课程的id
* 然后查询分数表，获得所有参加过语文考试的学生的id
* 有了上一步得到的学生id，查询分数表，将这些学生通过where in子句过滤出来
* 对上一步过滤后剩下的学生按照学生id进行分组，并对课程id进行count计数统计，并通过having子句过滤出只学习了三门课程的学生
* 对上一步得到的学生id，关联join学生表，获得学生的姓名等信息

```sql
--和学生表进行关联，获得学生的其他信息
select
    t1.stu_id,
    t1.stu_name
from student_info t1
join
(
    --获得既学习过语文课程，并且学习总课程数量为3的学生id
    select
        stu_id,
        count(course_id) as ct_course_id
    from score_info where stu_id in (
        --获得学习过语文课程的学生id
        select
            stu_id
        from score_info where course_id=(
            --获得语文课程id
            select course_id from course_info where course_name='语文'
        )
    )
    group by stu_id
    having ct_course_id=3
) t2
on t1.stu_id=t2.stu_id;
```

## 复杂查询

### 查询没有学全所有课程的学生学号和姓名

此题容易出现的思路是先对分数表按照学生编号进行分组，然后对课程进行count计数统计，过滤出所学课程不满足所有课程的学生，然后再和学生表进行关联，获得姓名等信息。乍一看是对的思路，但是有个问题容易忽略，那就是分数表中，有可能有些学生没有学习任何的课程，所以在分数表中，这些学生的数据并不会出现，但是按理来讲，这些学生的信息也是要输出的。所以需要换个思路。

正确的思路是，首先将学生表和分数表进行关联，而且是学生表left join分数表这种方式。然后对join之后的宽表，按照学生表的学生id和姓名进行分组，对分数表的课程id进行count计数统计，然后再使用having子句，过滤出那些没有学全所有课程的学生信息。针对有些没有学习任何课程的学生，在对课程id进行count计数统计的时候，实际上统计的是null值，对null值进行count统计，最终的结果是0。这样就把那些没有学习过任何课程的学生也统计进来了，保证了数据的准确性。

```sql
select
    --对学生表按照学生id和学生姓名进行分组 
    t1.stu_id,
    t1.stu_name,
    --求出分数表中该学生一共学习了几门课程（也包含没有学习过任何课程的学生）
    count(t2.course_id) as ct_course_id
from student_info t1
left join score_info t2
--将学生表和分数表进行关联
on t1.stu_id=t2.stu_id
group by t1.stu_id,t1.stu_name
--过滤出没学全所有课程的
having ct_course_id<(
    --求出所有课程的总数量
    select count(*) from course_info
);
```

### 查询出只选修了三门课程的全部学生的学号和姓名

查询分数表，按照学生id进行分组，对课程id进行count计数统计，通过having子句过滤出只学习了三门课程的学生id，然后再和学生表进行join关联，补全学生的姓名即可。

```sql
select
    t1.stu_id,
    t1.stu_name
from student_info t1
join (
    select
        stu_id,
        count(course_id) as ct_course_id
    from score_info
    group by stu_id
    having ct_course_id=3
) t2
on t1.stu_id=t2.stu_id;
```

## 多表查询

### 查询所有学生的学号，姓名，选课数，总成绩

通过题目的分析，学生的学号，选课数和总成绩，可以通过查询分数表得到。至于姓名，再和学生表进行关联即可。但是同时有个问题，就是有些学生并没有选择学习任何一门课程，那么在分数表中是没有这个学生的数据的。所以为了保险起见，还是采用先让学生表和分数表关联，然后对学生表的学生id和学生姓名分组，对分数表中的课程id进行count计数，对分数表中的score分数进行sum求和。

在进行count计数和sum求和时，如果列为null值，可以使用nvl函数进行判空处理。

```sql
select
    t1.stu_id,
    t1.stu_name,
    --这里需要对null值列进行判空特殊处理
    nvl(count(t2.course_id),0)
    nvl(sum(t2.score),0)
from student_info t1
left join score_info t2
on t1.stu_id=t2.stu_id
group by t1.stu_id,t1.stu_name;
```

### 查询平均成绩大于85的所有学生的学号，姓名和平均成绩

让学生表和分数表进行关联join，得到的宽表，按照学生表的id和姓名进行分组，然后使用avg求出分数表中的score的平均值。再使用having进行过滤即可。

```sql
select
    t1.stu_id,
    t1.stu_name,
    avg(t2.score) as avg_score
from student_info t1
join score_info t2
on t1.stu_id=t2.stu_id
group by t1.stu_id,t1.stu_name
having avg_score>85;
```

### 查询学生的选课情况，按照如下形式输出：学号，姓名，课程号，课程名称

既然是查询学生的选课情况，那么学生表肯定是主表，它里面有的学生信息是必须要输出的。学生的选课情况，就需要学生表和分数表进行关联，获取到该学生所选修的所有课程的id，至于课程的名字，还需要再和课程表进行关联获取到。所以该题是一个三表关联的场景，具体的步骤如下：

* 查询学生表，获取到学生的id和姓名
* 上一步得到的结果和分数表进行关联，获取到学生所对应的选课的课程id
* 上一步得到的结果继续和课程表进行关联，获取到课程的名称

```sql
select
    t1.stu_id,
    t1.stu_name,
    t2.course_id,
    t3.course_name
from student_info t1
--学生表和分数表关联，获取课程id
left join score_info t2
on t1.stu_id=t2.stu_id
--分数表和课程表关联，获取课程名称
left join course_info t3
on t2.course_id=t3.course_id;
```

### 查询课程编号为03且课程成绩在80以上的学生的学号和姓名，以及课程信息

该题也是一个多表联查的场景。根据题目分析，主要有以下几个步骤：

* 查询分数表，通过where过滤出课程id为03的，并且score大于80分的数据
* 上一步得到的结果和学生表通过学生id进行关联，获取到学生的姓名
* 上一步得到的结果和课程表通过课程id进行关联，获取到课程的名称

```sql
select
    t1.stu_id,
    t2.stu_name,
    t1.course_id,
    t3.course_name
from (
    select
        stu_id,
        course_id
    from score_info
    --过滤出来课程id为03且分数大于80的
    where course_id='03' and score>80
) t1
--和学生表关联获取学生姓名
join student_info t2
on t1.stu_id=t2.stu_id
--和课程表关联获取课程名称
join course_info t3
on t1.course_id=t3.course_id;
```

### 查询课程编号为01且课程分数小于60的学生信息，按课程分数降序排列

首先查询分数表，过滤出课程id为01且分数小于60的信息，然后和学生表进行join，补全学生信息。最后根据分数order by

```sql
select
    t2.*,
from (
    select
        stu_id,
        score
    from score_info
    where course_id='01' and score<60
) t1
join student_info t2
on t1.stu_id=t2.stu_id
order by t1.score desc;
```

### 查询所有课程成绩在70分以上（含70分）的学生的姓名，课程名称和分数，并按照分数升序排列

首先分析题目需求，要学生的姓名，那么肯定会和学生表进行关联，要课程的名称，那么肯定会和课程表进行关联。要分数，那么肯定会和分数表进行关联。现在只剩下一个条件，就是如何获得所有课程成绩均在70以上的学生的id

要求得所有课程成绩均在70以上的学生，肯定是从分数表进行查询。我们可以对score分数列，进行if判断，如果分数小于70分，那么记为1，如果大于等于70分，记为0，然后按照学生id进行分组，对这个标记位进行sum求和，如果总和是0，证明该学生所有课程的成绩都是在70分以上的。这样就能获取到学生的id了。然后就是从分数表中过滤出这些学生id的成绩信息，再然后就是顺理成章的和学生表关联获得学生姓名，和课程表关联获得课程名称。

综上，该题的分解步骤如下

* 查询分数表，对score分数列打标记，使用sum(if())函数进行统计，对学生id进行分组，对统计的和进行判断，从而获取到所有课程成绩都在70以上的学生的id
* 上一步获取到的学生id，和分数表进行关联，该将学生的所有课程成绩筛选出来
* 上一步获得的结果和学生表进行关联，获得学生的姓名
* 上一步获得的结果和课程表进行关联，获得课程的名称
* 最后按照分数进行升序排序

```sql
select
    t3.stu_name,
    t4.course_name,
    t2.score
from (
    --通过sum-if结合分组，获取到所有成绩都在70分以上的学生的id
    select
        stu_id,
        sum(if(score>=70,0,1)) as num
    from score_info
    group by stu_id
    having num=0
) t1
--和分数表关联，将所有成绩都在70以上的学生的成绩信息获取到
join score_info t2
on t1.stu_id=t2.stu_id
--和学生表关联，获取学生姓名
join student_info t3
on t2.stu_id=t3.stu_id
--和课程表关联，获取课程名称
join course_info t4
on t2.course_id=t4.course_id
--按照分数表中的分数升序排序
order by t2.score asc;
```

### 查询某学生不同课程但成绩相同的情况下，该学生的编号，课程编号，以及学生成绩

首先分析题目需求，要获得学生的编号，课程编号，以及学生成绩，直接通过查询分数表就可以得到。关键的点在于，怎么筛选出不同的课程，但是成绩相同的同一个学生。

要获得这个需求，是需要同一个学生的前提下，拿不同的课程之间，进行成绩的比较，然后取那些成绩相同的。是自己和自己的比较，所以是一个自连接的应用场景，也就是表自己和自己进行join。

```sql
select
    t1.stu_id,
    t1.course_id,
    t1.score
from score_info t1 
join score_info t2
--学生编号相同
on t1.stu_id=t2.stu_id
--课程编号不同
and t1.course_id!=t2.course_id
--成绩相同
and t1.score=t2.score;
```

### 查询课程编号为01的课程比02的课程成绩高的所有学生的学号

这个需求，是要找同一个学生的前提下，01课程成绩比02课程成绩高的那些学生，所以其实也是一个自连接。只不过自连接有前提条件，就是先把01课程的学生成绩筛选出来，然后把02课程的学生成绩筛选出来，然后让两个查询结果表进行join，关联的条件就是相同的学生id，同时过滤条件是01的课程成绩比02的课程成绩高。

```sql
select 
    t1.stu_id
from 
(select
    stu_id,
    score
from score_info
where course_id='01') t1
join
(select
    stu_id,
    score
from score_info
where course_id='02') t2
on t1.stu_id=t2.stu_id
where t1.score>t2.score;
```

### 查询学过编号为01的课程，并且也学过编号为02的课程的学生的学号和姓名

要获取同时学习过课程01和02的学生，可以拆开求解。先查询分数表，获得学习过01课程的学生id，然后再次查询分数表，获得学习过02课程的学生id，然后让这两个中间表进行join，关联的条件就是学生id相同，就可以获得同时学习过01和02课程的学生id，然后结果再关联学生表，获得姓名即可。

```sql
select
    t3.stu_id,
    t4.stu_name
from (
    select
        t1.stu_id as stu_id
    from
    (select
        stu_id
    from score_info
    where course_id='01') t1
    join
    (select
        stu_id
    from score_info
    where course_id='02') t2
    on t1.stu_id=t2.stu_id
) t3
join student_info t4
on t3.stu_id=t4.stu_id;
```

### 查询学过“张三”老师所教的所有课的学生的学号，姓名

首先分析题目，要查学生的学号和姓名，自然有关联学生表的操作。最主要的是这个条件，如何得到“张三”老师所教的所有的课程？这个可以通过课程表和老师表进行join得到。

有了张三老师所教授的所有的课程id之后，去查询分数表，过滤条件使用in子句，就是课程id在张三老师所教授的所有课程内。但是这样得到的，不是学过张三老师教授的所有课程的学生，而是有可能部分学生只学习过张三老师教授的所有课程中的一门或几门而已，不是全部的课程。为了得到学过张三老师教授的全部课程的学生，在刚才的基础上，在分数表中按照学生id进行分组，对学生对应的课程id进行count计数统计，然后对统计结果进行having子句过滤，如果得到的总数量和张三老师所教授的所有课程的总数量一致，则该学生就是要找的学生。

最后，再对上面得到的学生id结果和学生表关联，获取学生的姓名即可。

综上，此题可分为以下几个步骤

* 让课程表和老师表进行join，得到张三老师所教授的所有课程的id
* 让课程表和老师表进行join，得到张三老师所教授的所有课程的总数量
* 查询分数表，过滤条件是，课程id在张三老师所教授的所有课程id内即可，然后按照学生id进行分组，求count总量计数，同时having子句中过滤，要求总量等于张三老师所教授的所有课程的总量
* 上一步得到的学生id关联学生表，获得学生的姓名

```sql
select 
    t5.stu_id,
    t6.stu_name
from (
    --查询分数表，按照学生id分组，同时要求课程id必须在张三老师所教授的所有课程范围内，顺便对课程id进行统计计数，要求统计的结果和张三老师所教授的所有课程的总量相同
    select 
        stu_id
        count(*) as ct
    from score_info
    where course_id in (
        --求出张三老师所教授的所有课程的id
        select
            t1.course_id
        from course_info t1
        join teacher_info t2
        on t1.tea_id=t2.tea_id
        where t2.tea_name='张三'
    )
    group by stu_id
    having ct=(
        --求出张三老师所教授的所有课程的总数量
        select
            count(*)
        from course_info t3
        join teacher_info t4
        on t3.tea_id=t4.tea_id
        where t4.tea_name='张三'
    )
) t5
--关联学生表，获取姓名
join student_info t6
on t5.stu_id=t6.stu_id;
```

### 查询学过“张三”老师所教授的任意一门课程的学生的学号，姓名

这个需求和上面题目的需求相似，甚至更简单一些。上个题目中，我们在查询分数表的时候，让课程id，通过in子句，在张三老师所教授的所有课程id之内即可。这就已经获得了学过张三老师所教授的任意一门课程的学生id，但是需要额外的处理一步，就是对得到的学生id，进行去重。因为有些学生可能学习了张三老师所教授的所有课程中的多门课程。综合以上，得到的sql如下

```sql
select
    t3.stu_id,
    t4.stu_name
from (
    select
        --这里需要对学生id进行去重，因为有些学生可能学习了多门张三老师的课程
        distinct(stu_id) as stu_id
    from score_info
    where course_id in (
        --求出张三老师所教授的所有课程的id
        select
            t1.course_id
        from course_info t1
        join teacher_info t2
        on t1.tea_id=t2.tea_id
        where t2.tea_name='张三'
    )
) t3
--关联学生表，获得姓名
join student_info t4
on t3.stu_id=t4.stu_id;
```

### 查询没学过“张三”老师教授的任意一门课程的学生姓名

这个题目，和上个题目的需求有关联性，上个题目查询的是学过张三老师教授的任意一门课程的学生，这个题目要求的是没学过张三老师教授的任意一门课程的学生，两者是一个互补的关系。

可以在上个题目的基础上，我们获取到学过张三老师教授的任意一门课程的学生id，然后查询学生表，让学生表中的学生id not in即可。关键的语法是not in

```sql
select 
    stu_id,
    stu_name
from student_info
--查询学生表，把那些学习过张三老师任意一门课程的学生id排除掉，剩下的就是张三老师任意一门都没学过的学生
where stu_id not in (
    select
        --这里需要对学生id进行去重，因为有些学生可能学习了多门张三老师的课程
        distinct(stu_id) as stu_id
    from score_info
    where course_id in (
        --求出张三老师所教授的所有课程的id
        select
            t1.course_id
        from course_info t1
        join teacher_info t2
        on t1.tea_id=t2.tea_id
        where t2.tea_name='张三'
    )
);
```

### 查询至少有一门课程与学号为001的学生所学课程相同的学生的学号和姓名

分析题目，此题还是一个where in子句的使用。我们通过查询分数表，获取到001号学生所学的所有课程id，然后再次查询分数表，让课程id in，也就是存在于001号学生所学的所有课程id的范围内即可。这里需要注意的一点是，第二次查询分数表时，过滤条件中，需要将学号001的学生排除掉。因为自己和自己本来就相同。综上，语句如下

```sql
select
    t2.stu_id,
    t1.stu_name
from student_info t1
join (
    select
        --这里需要注意去重
        distinct(stu_id) as stu_id
    from score_info
    where course_id in (
        select 
            course_id
        from score_info
        where stu_id='001'
    )
    --这里需要注意的是，二次查询时需要将学号001的学生排除 
    and stu_id!='001'
) t2
on t1.stu_id=t2.stu_id;
```

### 按平均成绩从高到低显式所有学生的所有课程的成绩以及平均成绩

此题需要分步骤进行计算。因为要的是所有学生，所以也包含那些从没学过课程的学生，他们的课程分数是null，平均成绩也是null，但是也需要显示出来。所以这里需要将学生表，分数表，以及课程表进行关联。至于平均成绩，需要单独查询分数表，按照学生id进行分组，对score进行avg计算，得到的结果，和上一步的结果按照学生id进行关联即可。只不过会有一个现象就是，每个学生的每门课程，对应的平均成绩是一样的。（仅是展示需要，无业务价值）

```sql
select
    t1.stu_id,
    t1.stu_name,
    t2.course_id,
    t3.course_name,
    t2.score,
    t4.avg_score
from student_info t1
left join score_info t2
on t1.stu_id=t2.stu_id
left join course_info t3
on t2.course_id=t3.course_id
left join (
    select 
        stu_id,
        avg(score) as avg_score
    from score_info
    group by stu_id
) t4
on t2.stu_id=t4.stu_id
order by t4.avg_score desc;
```





