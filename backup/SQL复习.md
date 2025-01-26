## DDL 操作数据库，数据表
```sql
    show databases;
    create database if not exists ''
    drop database if exists ''
    use ''
    select database();//查看当前使用的数据库
    
    show tables;
    desc ''//查询表的结构
    create table '' (字段名1 数据类型1,字段名2 数据类型2,字段名2 数据类型2);
    drop if exists '';
    
    
    alter table '' rename to '';
    alter table '' add 字段名 类型;
    alter table '' modify 字段名 新类型 约束;
    alter table '' change 字段 新字段 新类型;alter table '' drop 字段;
```
## DML 对表中的数据进行增删改查
```sql
    insert into 'table'(字段1，字段2) values (值1, 值2),(值1, 值2),...;
    
    update 'table' set 字段1=值1, 字段2=值2,...[where 条件];
    
    delete from 'table' [where 条件]
```
## DQL 对表中的数据进行查询
```sql
    select 字段【distinct(去重)】【as 别名】
    ||聚合函数【count(统计非空数据 *) max min sum avg】(列名)
    from 表名
    where 条件【<>不等于 =等于 in(,,...) null 用is与is not like(_代表单个任意字符，%代表任意个字符)】 //不能执行聚合函数
    group by 分组字段名
    ///注意，分组之后只能查分组字段和聚合函数
    having 分组后条件//可以执行聚合函数
    order by 排序字段【ASC(默认) DESC(降序)】, 排序字段【ASC(默认) DESC(降序)】
    limit 起始索引，查询条目数（MySQL方言）
    //起始索引 = （当前页码-1）* 每页显示条数
```
## 约束
```sql
    auto_increment //数字类型，唯一约束
    非空约束 NOT NULL
    唯一约束 UNIQUE
    主键约束 PRIMARY KEY
    默认约束 DEFAULT
    外键约束 FOREGIN KEY
    constraint 【约束名称】 foreign key(从表字段) references 主表名(主表字段)
    // 括号是必须的！！！
    
    alter table ’table‘ alter '字段' set DEFAULT 默认值
    alter table ’table‘ alter '字段' drop DEFAULT 默认值
```
## DCL 对数据库进行权限控制

## 数据库设计

需求分析，逻辑分析（ER图），物理设计，维护设计

一对多（多的一方建立外键）

多对多（建立一个中间表(也可以添加业务字段)）

一对一（任意一方设置唯一外键。）

## 多表查询

### 内连接（交集）

#### 显式

select 字段 from 表名1 inner join 表名2 ON 条件

#### 隐式

select 字段 from 表名1，表名2 where 条件

### 外连接

#### 左外连接

select 字段 from 表名1 left join 表名2 ON 条件

（表1全部以及符合条件的）

#### 右外连接

select 字段 from 表名1 right join 表名2 ON 条件

（表2全部以及符合条件的）

子查询

    单行单列 <> = 
    多行单列 in
    多行多列 作为虚拟表

**练习的启发**
```sql
    -- 2.查询员工编号，员工姓名，工资，职务名称，职务描述，部门名称，部门位置
    SELECT
        emp.id,
        emp.ename,
        emp.salary,
        job.jname,
        job.description,
        dept.dname,
        dept.loc
    FROM
        emp,
        job,
        dept
    WHERE
        emp.job_id = job.id
    AND emp.dept_id = dept.id;
    
    
    SELECT
        t.id,
        t.ename,
        t.salary,
        t.jname,
        t.description,
        dept.dname,
        dept.loc
    FROM
        (
            SELECT
                emp.id,
                emp.ename,
                emp.salary,
                job.jname,
                job.description,
                emp.dept_id -- 必须要添加
            FROM
                emp
            LEFT JOIN job ON emp.job_id = job.id
    ) t
    LEFT JOIN dept ON t.dept_id = dept.id;
    
    SELECT t.id, t.ename, t.salary, t.jname, t.description, dept.dname, dept.loc 
    FROM (
        SELECT emp.id, emp.ename, emp.salary, job.jname, job.description, emp.dept_id 
        FROM emp 
        LEFT JOIN job ON emp.job_id = job.id
    ) t 
    LEFT JOIN dept ON t.dept_id = dept.id;
    
    -- 5.查询出部门编号、部门名称、部门位置、部门人数
    SELECT t.dept_id, t.dname, t.loc, count(*) from
    (SELECT
        emp.id,
        emp.dept_id,
        dept.dname,
        dept.loc
    FROM
        emp,dept
    WHERE
        emp.dept_id = dept.id) t
    GROUP BY t.dept_id;
```
    

## 事务四大特征
```
    Begin;
    Rollback;
    Commit;
```
1. 原子性: 事务中的操作，要么同时成功，要么同时失败
  
2. 一致性: 事务必须是所有的数据都保持统一状态
  
3. 隔离性: 多个事物之间，操作的可见性
  
4. 持久性: 事务一旦提交或回滚，堆数据库的数据的改变就是永久的