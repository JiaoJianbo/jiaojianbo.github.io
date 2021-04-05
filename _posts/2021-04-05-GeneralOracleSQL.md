---
layout: post
title: "Oracle 常用查询"
date: 2021-04-05 11:40:00 +0800
categories: Oracle
tags: Oracle
author: Bobby
---

* content
{:toc}

主要记录一些工作中偶尔会用到的一些 Oracle 专有的查询和操作。



1. 切换当前 schema，以及查看当前 schema 和 当前登录 User。

```sql
alter session set current_schema=<target schema>;
select user, sys_context('USERENV','CURRENT_SCHEMA') from dual;

--查看当前用户的所有表
select * from user_tables;
--查看所有用户表的列
select * from user_tab_columns;
--查看表的创建，修改时间
select * from user_objects;
```

2. 查看所有用户

```sql
select * from dba_users;
select * from all_users;
select * from user_users;
```

3. 查看用户或角色系统权限（直接赋给用户或角色的系统权限）

```sql
select * from dba_sys_privs;
select * from user_sys_privs;

--查看角色（只能看登录用户所拥有的角色）所包含的权限
select * from role_sys_privs;
```

比如我要查询用户 Bobby 所拥有的权限：

```sql
SQL> select * from dba_sys_privs where grantee='BOBBY';

GRANTEE             PRIVILEGE                   ADMIN_OPTION
------------------- --------------------------- ---------------
BOBBY               CREATE_TRIGGER               NO
BOBBY               UNLIMITED TABLESPACE         NO
```


4. 查看用户对象权限

```sql
select * from dba_tab_privs;
select * from all_tab_privs;
select * from user_tab_privs;
```

5. 查看所有角色

```sql
select * from dba_roles;
```

6. 查看用户所拥有的角色

```sql
select * from dba_role_privs;
select * from user_role_privs;
```

比如我要查询用户 Bobby 所拥有角色：

```sql
SQL> select * from dba_role_privs where grantee='BOBBY';

GRANTEE         GRANTED_ROLE        ADMIN_OPTION        DEFAULT_ROLE
--------------- ------------------- ------------------- ---------------
BOBBY                 DBA              NO                   YES
```

7. 查看哪些用户具有 sysdba 或者 sysoper 系统权限（查询时需要具有相应的权限）

```sql
select * from V$PWFILE_USERS;
```

查看一个用户所有的权限及角色

```sql
select privilege from dba_sys_privs where grantee='BOBBY'
union
select privilege from dba_sys_privs where grantee in （
  select GRANTED_ROLE from dba_role_privs where grantee='BOBBY'
);
```

8. 查看数据库 NLS_LENGTH_SEMANTICS 参数值 （这个参数只有两个可选值，即 byte 和 char。默认是 byte）

```sql
select * from nls_database_parameters t where t.parameter='NLS_LENGTH_SEMANTICS';

--修改字符串的单位，varchar2(32 CHAR)
ALTER SESSION SET NLS_LENGTH_SEMANTICS=CHAR;
```

9. 查看数据库 NLS_CHARACTERSET 参数值

```sql
select * from nls_database_parameters t where t.parameter='NLS_CHARACTERSET';
```

10. 查看当前库中被锁的资源以及被谁锁了

```sql
select o.object_name,s.machine, l.os_user_name, s.sid, s.serial#, s.username, s.logon_time 
from v$locked_object l, dba_objects o, v$session s
where l.object_id=o.object_id and l.session_id=s.sid;
```

根据 sid 查看锁操作的具体 SQL 语句， 如果 SQL 不重要，可以直接 kill 掉。

```sql
select sql_text from v$session a, v$sqltext_with_newlines b
where DECODE(a.sql_hash_value, 0, prev_hash_value, sql_hash_value)=b.hash_value
and a.sid=&sid
order by piece;
```

kill 掉该事务（需要具有相应权限才行）

```sql
alter system kill session '&sid,&serial';
```

11. 左/右两端补空格的 Oracle 函数。

```sql
lpad(string, padded_length, [padding_string])
rpad(string, padded_length, [padding_string])
```

12. 修改表结构

```sql
--修改列名
alter table taget_table RENAME COLUMN col_1 to col_1;

--添加列
alter table 表名 ADD (字段名 字段类型) [default '默认值'] [ null / not null]

--删除列
alter table 表名 drop (字段名)

--修改列长度或类型，（如果长度由长变短，且数据库中现有值超过这个最大长度，那么直接修改会失败，需要修改表中现有值）
alter table employee_info modify sex char(2);
```

13. 创建同义词

```sql
--首先将所有者（onwer）的表的权限分配给目标用户 (target_uesr)
GRANT SELECT, INSERT, UPDATE ON table_owner.TABLE TO target_user;

--再给目标用户(target_uesr) 创建同义词，指向所有者(owner) 的表
CREATE OR REPLACE SYNONYM target_user.TABLE FOR table_owner.TABLE;
```

14. 查找所有表上的外键约束

```sql
select * from user_constraints where constraint_type='R';

--生成禁用所有外键的 SQL
select 'alter table ' || table_name || ' disable constraint' || constraint_name || ';' 
from user_constraints where constraint_type='R'; 
```

15. NVL2(expr1, expr2, expr3) 表达式

如果参数表达式 expr1 的值为 NULL，则 NVL2 函数的返回值即 expr3 的值，否则为 expr2 的值。

16. 查看表的赋权情况

```sql
select * from user_tab_privs_made;
```

17. 查看 SQL 的执行记录

```sql
select * from v$sql where sql_text like 'delete from employee%';

select sql_text, module, first_load_time from v$sqlarea
where first_load_time > '2021-04-05' and sql_text like '%EMPLOYEE%'
order by first_load_time desc;
```

18. 时间计算

使用内置函数 `numtodsinterval` 增加小时数，分钟和秒

```sql
-- 加 8 小时
select sysdate, (sysdate + numtodsinterval(8, 'hour')) as LOCAL_TIME from dual;
```

加一个简单的数来增加天

```sql
-- 加 1 天
select sysdate, (sysdate + 1) as NEXT_DAY from dual;
```

使用内置函数 `add_months` 来增加年和月

```sql
-- 加 1 月
select sysdate, add_months(sysdate, 1) as NEXT_MONTH from dual;
```

19. `to_date` 和 `to_char` 函数

```sql
to_date('2021-04-05 11:40:45', 'yyyy-mm-dd hh24:mi:ss')
to_char(data_value, 'yyyy-mm-dd hh24:mi:ss')
```

20. SQL 语句中使用 `json_value` 计算 JSON 值

```sql
select json_body
from table1
where json_value(json_body, '$.reqbody.country')='CHINA'
and create_time > (sysdate-1);
```

21. 查看数据中每个表所占大小

```sql
select t.bytes, round(BYTES/1024/1024, 2) as MB, t.*
from sys.dba_segments t
where t.OWNER='<USRE_NAME>'
and segment_type='TABLE'
order by t.bytes desc;

--或
select t.bytes, round(BYTES/1024/1024, 2) as MB, t.*
from user_segments t
where segment_type='TABLE'
order by MB desc;
```

22. 查看数据库运行的最大连接数

```sql
select value from v$parameter where name='processes';

--最大连接 
show parameter processes; 

--查看当前连接数
select count(*) from v$session;
select count(*) from v$process;
```

23. 查看当前有哪些用户正在使用数据

```sql
SELECT osuser, a.username,cpu_time/executions/1000000||'s', sql_fulltext, machine 
from v$session a, v$sqlarea b
where a.sql_address =b.address order by cpu_time/executions desc;
```

