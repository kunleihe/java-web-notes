# 语法
## 创建 create
```sql
create table  表名(
    字段1  字段1类型 [约束]  comment  字段1注释,
    字段2  字段2类型 [约束]  comment  字段2注释,
    ......
    字段n  字段n类型 [约束]  comment  字段n注释  
) comment  表注释 ;
```
## 数据库操作 - DML 
### 增加 insert
向指定字段添加数据
```SQL
insert into 表名 (字段名1, 字段名2) 
values (值1, 值2);
```

向全部字段添加数据
```SQL
insert into 表名 
values (值1, 值2,...);
```

批量添加数据（指定字段）
```SQL
insert into 表名 (字段名1, 字段名2) 
values (值1, 值2), 
	   (值1, 值2);
```

批量添加数据（全部字段）
```SQL
insert into 表名 
values (值1, 值2, ...), 
	   (值1, 值2, ...);
```
### 修改 update
```SQL
update 表名 
set 字段名1 = 值1,
	字段名2 = 值2,
	...,
	where 条件;
```

### 删除 delete
```SQL
delete from 表名
where 条件;
```

## 数据库操作 - DQL
```SQL
SELECT
    字段列表
FROM
    表名列表
WHERE
    条件列表
GROUP  BY
    分组字段列表
HAVING
    分组后条件列表
ORDER BY
    排序字段列表
LIMIT
    分页参数
```
### 条件查询
```SQL
SELECT  字段列表  
FROM   表名   
WHERE   条件列表 ; -- 条件列表：意味着可以有多个条件
```

查询 姓名 为两个字的员工信息
```SQL
SELECT id, username, password, name, gender, image, job, entrydate, create_time, update_time
FROM tb_emp
WHERE name LIKE '__';  # 通配符 "_" 代表任意1个字符
```

查询 姓 '张' 的员工信息
```SQL
SELECT id, username, password, name, gender, image, job, entrydate, create_time, update_time
FROM tb_emp
WHERE name LIKE '张%'; # 通配符 "%" 代表任意个字符（0个 ~ 多个）
```
### 聚合函数
```SQL
SELECT  聚合函数(字段列表)  FROM  表名 ;
```
常用聚合函数
- count
- max
- min
- avg
- sum
### 分组查询
```SQL
SELECT  字段列表  
FROM  表名  
WHERE 条件  
GROUP BY 分组字段名  
HAVING 分组后过滤条件;
```

```SQL
SELECT job, COUNT(*)
FROM tb_emp
WHERE entrydate <= '2015-01-01'   -- 分组前条件
GROUP BY job                      -- 按照job字段分组
HAVING COUNT(*) >= 2;             -- 分组后条件
```

`WHERE` 与 `HAVING` 的区别
- 执行时机不同：where 是分组之前进行过滤，不满足 where 条件，不参与分组；而 having 是分组之后对结果进行过滤。
- 判断条件不同：where 不能对聚合函数进行判断，而 having 可以
### 排序查询
```SQL
SELECT id, username, password, name, gender, image, job, entrydate, create_time, update_time
FROM tb_emp
ORDER BT entrydate ASC; -- 按照entrydate字段下的数据进行升序排序

SELECT id, username, password, name, gender, image, job, entrydate, create_time, update_time
FROM tb_emp
ORDER BT entrydate DESC; -- 按照entrydate字段下的数据进行降序排序
```
### 分页查询
```SQL
SELECT  字段列表  
FROM   表名  
LIMIT  起始索引, 查询记录数 ;
```

例子
```SQL
SELECT id, username, password, name, gender, image, job, entrydate, create_time, update_time
FROM tb_emp
LIMIT 0 , 5; -- 从索引0开始，向后取5条记录
```
### 案例
```SQL
-- if(条件表达式, true取值 , false取值)
SELECT if(gender=1,'男性员工','女性员工') AS 性别, COUNT(*) AS 人数
FROM tb_emp
GROUP BY gender;
```

```SQL
-- case 表达式 when 值1 then 结果1  when 值2  then  结果2 ...  else  result  end
SELECT (case job
             when 1 then '班主任'
             when 2 then '讲师'
             when 3 then '学工主管'
             when 4 then '教研主管'
             else '未分配职位'
        end) AS 职位 ,
       COUNT(*) AS 人数
FROM tb_emp
GROUP BY job;
```
# 多表设计
## 一对多
- 一对多关系实现：在数据库表中多的一方，添加字段，来关联属于一这方的主键。
#### 外键约束
```SQL
-- 创建表时指定
CREATE TABLE 表名(
    字段名    数据类型,
    ...
    CONSTRAINT   [外键名称]  FOREIGN  KEY (外键字段名)   
    REFERENCES   主表 (主表列名)    
);


-- 建完表后，添加外键
ALTER TABLE  表名  
ADD CONSTRAINT  外键名称  
FOREIGN KEY(外键字段名) 
REFERENCES 主表(主表列名);
```
## 一对一
- 在任意一方加入外键，关联另外一方的主键，并且设置外键为唯一的(UNIQUE)
## 多对多
- 建立第三张中间表，中间表至少包含两个外键，分别关联两方主键
```SQL
-- 学生表
CREATE TABLE tb_student(
    id int auto_increment primary key comment '主键ID',
    name varchar(10) comment '姓名',
    no varchar(10) comment '学号'
) comment '学生表';
-- 学生表测试数据
INSERT INTO tb_student(name, no) VALUES ('黛绮丝', '2000100101'),('谢逊', '2000100102'),('殷天正', '2000100103'),('韦一笑', '2000100104');

-- 课程表
CREATE TABLE tb_course(
   id int auto_increment primary key comment '主键ID',
   name varchar(10) comment '课程名称'
) comment '课程表';
-- 课程表测试数据
INSERT INTO tb_course (name) VALUES ('Java'), ('PHP'), ('MySQL') , ('Hadoop');

**-- 学生课程表（中间表）**
CREATE TABLE tb_student_course(
   id int auto_increment comment '主键' primary key,
   student_id int NOT null comment '学生ID',
   course_id  int NOT null comment '课程ID',
   constraint fk_courseid foreign key (course_id) references tb_course (id),
   constraint fk_studentid foreign key (student_id) references tb_student (id)
)comment '学生课程中间表';
-- 学生课程表测试数据
insert into tb_student_course(student_id, course_id) values (1,1),(1,2),(1,3),(2,2),(2,3),(3,4);
```

# 多表查询
- 笛卡尔积：两个集合(A集合和B集合)的所有组合情况
- 在SQL语句中，去除无效的笛卡尔积呢需要给多表查询加上连接查询的条件
```SQL
select * from tb_emp , tb_dept 
where tb_emp.dept_id = tb_dept.id ;
```
## 分类
### 内连接
- 查询多表中交集部分的数据
- 分为显式内连接和隐式内连接

显式内连接
```SQL
select  字段列表   
from   表1 AS 别名1
inner join 表2 AS 别名2
on  连接条件 ... ;
```

隐式内连接
```SQL
select  字段列表   
from   表1 , 表2   
where  连接条件 ... ;
```

> [!info] 别名
> 一旦为表起了别名，就不能再使用表名来指定对应的字段了，此时只能够使用别名来指定字段
### 外连接
1. 左外连接：查询表1(左表)的所有数据，当然也包含表1和表2交集部分的数据。

```SQL
select  字段列表   
from   表1  
left  [ outer ]  join 表2  
on  连接条件 ... ;
```

2. 右外连接：查询表2(右表)的所有数据，当然也包含表1和表2交集部分的数据。

```SQL
select  字段列表   
from   表1  
right  [ outer ]  join 表2  
on  连接条件 ... ;
```
### 子查询
SQL语句中嵌套select语句，称为嵌套查询，又称子查询。

```SQL
SELECT  *  
FROM   t1   
WHERE  column1 =  ( SELECT  column1  FROM  t2 ... );
```

#### 标量子查询
- 子查询返回的结果是单个值(数字、字符串、日期等)，最简单的形式，这种子查询称为标量子查询。

案例1：查询"教研部"的所有员工信息
可以将需求分解为两步：
1. 查询 "教研部" 部门ID
2. 根据 "教研部" 部门ID，查询员工信息

```SQL
-- 1.查询"教研部"部门ID
select id 
from tb_dept 
where name = '教研部';    #查询结果：2
-- 2.根据"教研部"部门ID, 查询员工信息
select * 
from tb_emp 
where dept_id = 2;

-- 合并出上两条SQL语句
select * 
from tb_emp 
where dept_id = (select id from tb_dept where name = '教研部');
```

#### 列子查询
- 子查询返回的结果是一列(可以是多行) - 用 where ... in ...

```SQL
-- 1.查询"销售部"和"市场部"的部门ID
select id 
from tb_dept 
where name = '教研部' or name = '咨询部';    #查询结果：3,2
-- 2.根据部门ID, 查询员工信息
select * 
from tb_emp 
where dept_id in (3,2);

-- 合并以上两条SQL语句
select * 
from tb_emp 
where dept_id in (select id from tb_dept where name = '教研部' or name = '咨询部');
```

#### 行子查询
- 子查询返回的结果是一行(可以是多列) - 用 where ... = ...
```SQL
-- 查询"韦一笑"的入职日期 及 职位
select entrydate , job 
from tb_emp 
where name = '韦一笑';  #查询结果： 2007-01-01 , 2
-- 查询与"韦一笑"的入职日期及职位相同的员工信息
select * 
from tb_emp 
where (entrydate,job) = ('2007-01-01',2);

-- 合并以上两条SQL语句
select * 
from tb_emp 
where (entrydate,job) = (select entrydate , job from tb_emp where name = '韦一笑');
```
# 事务 Transaction
- 事务是一组操作的集合，它是一个不可分割的工作单位
- 事务会把所有的操作作为一个整体一起向系统提交或撤销操作请求，即这些操作要么同时成功，要么同时失败。
- 事务作用：保证在一个事务中多次操作数据库表中数据时，要么全都成功,要么全都失败。

MYSQL中有两种方式进行事务的操作：
1. 动提交事务：即执行一条sql语句提交一次事务。（默认MySQL的事务是自动提交）
2. 手动提交事务：先开启，再提交
## 事务语句
- start transaction; / begin; 开启手动控制事务
- commit; 提交事务
- rollback; 回滚事务

```SQL
-- 开启事务
start transaction ;

-- 删除学工部
delete from tb_dept where id = 1;

-- 删除学工部的员工
delete from tb_emp where dept_id = 1;

-- 提交事务（成功时执行）
commit ;

-- 回滚事务（失败时执行）
rollback ;
```
## 事务的特性 (ACID)
- 原子性（Atomicity）：事务是不可分割的最小单元，要么全部成功，要么全部失败。
- 一致性（Consistency）：事务完成时，必须使所有的数据都保持一致状态。
- 隔离性（Isolation）：数据库系统提供的隔离机制，保证事务在不受外部并发操作影响的独立环境下运行。
- 持久性（Durability）：事务一旦提交或回滚，它对数据库中的数据的改变就是永久的。

# 索引 Index
- 索引(index)：是帮助数据库高效获取数据的数据结构 。

```SQL
-- 添加索引
create index idx_sku_sn on tb_sku (sn);  #在添加索引时，也需要消耗时间

-- 查询数据（使用了索引）
select * from tb_sku where sn = '100000003145008';
```

优点：
1. 提高数据查询的效率，降低数据库的IO成本
2. 通过索引列对数据进行排序，降低数据排序的成本，降低CPU消耗。

缺点
1. 索引会占用存储空间。
2. 索引大大提高了查询效率，同时却也降低了insert、update、delete的效率。

## 结构
- 默认为 B+Tree 结构
- 每一个节点，可以存储多个key（有n个key，就有n个指针）
- 节点分为：叶子节点、非叶子节点
	- 叶子节点，就是最后一层子节点，所有的数据都存储在叶子节点上
    - 非叶子节点，不是树结构最下面的节点，用于索引数据，存储的的是：key+指针

> [!info] 为什么不适用平衡二叉树或红黑树
> 最大的问题就是在数据量大的情况下，树的层级比较深，会影响检索速度。因为不管是二叉搜索数还是红黑数，一个节点下面只能有两个子节点。此时在数据量大的情况下，就会造成数的高度比较高，树的高度一旦高了，检索速度就会降低。

## 语法
```SQL
create index 索引名 on  表名 (字段名,... ) ;
```

案例
```SQL
create index idx_emp_name on tb_emp(name);
```

查看索引
```SQL
show  index  from  表名;
```

删除索引
```SQL
drop  index  索引名  on  表名;
```

> [!info] 主键字段，在建表时，会自动创建主键索引

