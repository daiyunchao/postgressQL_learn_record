# postgressQL_learn_record
postgressQL学习记录

### 触发器过程
> 在一个 PL/pgSQL 函数当做触发器调用的时候， 系统会在顶层的声明段里自动创建几个特殊变量。有如下这些
> NEW 表示新数据
> 
> OLD 表示旧数据
> 
> TG_NAME 触发器名称
> 
> TG_WHEN 触发时间 BEFORE 或者是 AFTER
> 
> TG_LEVEL 数据类型是 text；是一个由触发器定义决定的字符串， 要么是 ROW 要么是 STATEMENT(未明白)
> 
> TG_OP 执行的操作 可以是 INSERT，UPDATE 或者 DELETE
> 
> TG_RELID 数据类型是 oid；是导致触发器调用的表的对象标识（OID）(未明白)
> 
> TG_RELNAME 表名
> 
> TG_NARGS 是在CREATE TRIGGER 语句里面赋予触发器过程的参数的个数 参数个数
> 
> TG_ARGV[] 数据类型是 text 的数组；是 CREATE TRIGGER语句里的参数。 下标从 0 开始记数．非法下标（小于 0 或者大于等于 tg_nargs）导致返回一个 NULL 值
> 
**一个触发器函数必须返回 NULL 或者是 一个与导致触发器运行的表的记录/行完全一样的结构的数据**


## 常用内置函数

>NOW() 当前时间

## 条件表达式
1. CASE
   > SQL CASE 表达式是一种通用的条件表达式，类似于其它语言中的 if/else 语句
   
   ```sql
   --查询tbl_test表，如果sex等于'm'则显示'男',，如果是'f'则显示'女'
   select name,case when sex = 'm' then '男' else '女' end as sex from tbl_test;

   --一次查询tbl_test表中男女的分别人数
   select sum(case when sex = 'm' then 1 else 0 end) as 男,sum(case when sex='f' then 1 else 0 end)as 女 from tbl_test;
   ```

2. coalesce
   > 入参个数不限，返回参数中第一个非NULL值
   ```sql
   select coalesce(null,null,1,2,3);
   --返回1

   select coalesce(null,null,'one','two');
   --返回one
   ```

3. NULLIF
   > 两个参数相等,则返回NULL 如果不相等则返回参数一
   ```sql
   select nullif(1,1)
   --返回NULL
   select nullif(1,2);
   --返回1
   ```
4. GREATEST和LEAST
   > 入参个数不限，分别返回入参中的最大和最小值
   ```sql
   select greatest(1,2,3,4); --4
   select greatest('a','b','c','d'); --d
   select least('a','b','c','d'); --a
   select least(1,2,3,4); --1

   ```
 
 ## 特殊语法
 1. 双冒号 ::
   > https://stackoverflow.com/questions/15537709/what-does-do-in-postgresql
   简单的说就是类型转换
   例如:
   ```sql
   SELECT '{apple,cherry apple, avocado}'::text[];
   --表示 将查询出来的结果转换成 text的数组
   ```
 2. current_setting
   ```sql
   select current_setting('hws.current_user_id')
   --获取hws.current_user_id的配置信息,如果没有配置该数据会报出异常
   select current_setting('hws.current_user_id',true)
   --就算没有设置该属性,也不会报异常,只是会返回一个null
   ```
3. regclass ??
4. timestamp WITHOUT TIME ZONE 一个不带时区的时间格式
    ```sql
    
    select now():: timestamp without time zone
    --结果为:
    '2018-08-30 16:43:53.051775'

    --我们查询 now()的时候就是 with time zone,这里为了更明确:
    select now():: timestamp with time zone
     --结果为:
    '2018-08-30 16:44:54.38849+08'
    ```
5. RETURNING cloumname/* 将插入或是修改的新值返回出来
   ```sql
   declare v_created_user basic_user;--声明一个变量
   insert into basic_user (nickname) values
    (_nickname) 
    returning * into v_created_user; --将插入到basic_user的值返回到 变量v_created_user中

   ```

6. extract
    extract函数格式：
    extract （field from source）
    extract函数是从日期或者时间数值里面抽取子域，比如年、月、日等。source必须是timestamp、time、interval类型的值表达式。field是一个标识符或字符串，是从源数据中的抽取的域。
    具体可见:https://blog.csdn.net/nextaction/article/details/76473613
    其他值:
   ```sql
   select extract(epoch from now() + interval '30 days')
   --返回'1538215003.33026'
   --当前时间+30天的时间戳
   ```
 ## 创建序列
 ```sql
 CREATE SEQUENCE basic.basic_user_uid_seq INCREMENT by 1 MINVALUE 1 MAXVALUE 9223372036854775807 START WITH 1;
 -- 创建一个名为:basic_user_uid_seq的序列,按照1递增 最小值为 1 最大值为9223372036854775807 从1开始

 select nextval('basic.basic_user_uid_seq'); --查询获取序列下一个值
 select currval('basic.basic_user_uid_seq'); --查询序列当前的值
 ```

## postgresql----字符串函数与操作符
```sql
select 'Post'||'gresql'||' good!';--select 'Post'||'gresql'||' good!';
select 1||' one'; --1 one
select bit_length('one'); --24 字符串的bit
select char_length('中国'); --2 字符串长度
select length('中国'); --2 字符串长度
select octet_length('中国'); --6 字节数
select lower('HeLLO'); --hello 转换小写
select upper('HellO'); --HELLo 转换大写
select initcap('hello world !'); --Hello World 驼峰
select overlay('Txxxxas' placing 'hom' from 2 for 4); --Thomas 替换字符串

```
> 
## 理论
 > 行级安全的作用是什么?