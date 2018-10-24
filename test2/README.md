# 实验2：用户管理 - 掌握管理角色、权根、用户的能力，并在用户之间共享对象。

## 实验内容：
     Oracle有一个开发者角色resource，可以创建表、过程、触发器等对象，但是不能创建视图。本训练要求：
    - 在pdborcl插接式数据中创建一个新的本地角色con_res_view，该角色包含connect和resource角色，同时也包含CREATE VIEW权限，
      这样任何拥有con_res_view的用户就同时拥有这三种权限。
    - 创建角色之后，再创建用户new_user，给用户分配表空间，设置限额为50M，授予con_res_view角色。
    - 最后测试：用新用户new_user连接数据库、创建表，插入数据，创建视图，查询表和视图的数据。
## 实验步骤(Git Bash)：
- 第一步：以system登录到pdborcl，创建角色con_res_view和用户new_user，并授权和分配空间：
```SQL
[oracle@deep02 ~]$ sqlplus system/123@pdborcl

SQL*Plus: Release 12.1.0.2.0 Production on 星期三 10月 24 08:55:20 2018

Copyright (c) 1982, 2014, Oracle.  All rights reserved.

上次成功登录时间: 星期三 10月 24 2018 08:55:12 +08:00

连接到:
Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options

SQL> CREATE ROLE con_wwh_view;

角色已创建。

SQL>  GRANT connect,resource,CREATE VIEW TO con_wwh_view;

授权成功。

SQL> CREATE USER new_usr IDENTIFIED BY 123 DEFAULT TABLESPACE users TEMPORARY TABLESPACE temp;

用户已创建。

SQL> ALTER USER new_usr QUOTA 50M ON users;

用户已更改。

SQL> GRANT con_wwh_view TO new_usr;

授权成功。

SQL> exit

```
- 第二步：新用户new_user连接到pdborcl，创建表mytable和视图myview，插入数据，最后将myview的SELECT对象权限授予hr用户。
```SQL
$ sqlplus new_usr/123@pdborcl
SQL> show user;
USER 为 "SYSTEM"
SQL> CREATE TABLE mytable (id number,name varchar(50));

表已创建。

SQL> INSERT INTO mytable(id,name)VALUES(1,'zhang');

已创建 1 行。

SQL> INSERT INTO mytable(id,name)VALUES (2,'wang');

已创建 1 行。

SQL> CREATE VIEW myview AS SELECT name FROM mytable;

视图已创建。

SQL> SELECT * FROM myview;

NAME
--------------------------------------------------------------------------------
zhang
wang

SQL> GRANT SELECT ON myview TO hr;

授权成功。

SQL> exit

```

- 第三步：用户hr连接到pdborcl，查询new_user授予它的视图myview
```SQL
$ sqlplus new_usr/123@pdborcl
SQL> SELECT * FROM new_usr.myview;

NAME
--------------------------------------------------------------------------------
zhang
wang
zhang
WANG

SQL> exit

```
### 查看数据库使用情况
 ```SQL
 sqlplus system/123@pdborcl
 SQL> SELECT tablespace_name,FILE_NAME,BYTES/1024/1024 MB,MAXBYTES/1024/1024 MAX_MB,autoextensible FROM dba_data_files  WHERE  tablespace_name='USERS';

TABLESPACE_NAME
--------------------------------------------------------------------------------
FILE_NAME
--------------------------------------------------------------------------------
        MB     MAX_MB AUTOEXTEN
---------- ---------- ---------
USERS
/home/oracle/app/oracle/oradata/orcl/users01.dbf
         5 32767.9844 YES
         
SQL>SELECT a.tablespace_name "表空间名",Total/1024/1024 "大小MB",
        group  BY tablespace_name)b
  2   free/1024/1024 "剩余MB",( total - free )/1024/1024 "使用MB",
 Round(( total - free )/ total,4)* 100 "使用率%"
  4   from (SELECT tablespace_name,Sum(bytes)free
        FROM   dba_free_space group  BY tablespace_name)a,
       (SELECT tablespace_name,Sum(bytes)total FROM dba_data_files
        group  BY tablespace_name)b
  8   where  a.tablespace_name = b.tablespace_name;

表空间名
--------------------------------------------------------------------------------
    大小MB     剩余MB     使用MB    使用率%
---------- ---------- ---------- ----------
SYSAUX
       950         47        903      95.05

UNDOTBS1
       365    341.875     23.125       6.34

USERS
         5      3.625      1.375       27.5


表空间名
--------------------------------------------------------------------------------
    大小MB     剩余MB     使用MB    使用率%
---------- ---------- ---------- ----------
SYSTEM
       800        .25     799.75      99.97


 ```
## 实验步骤(Oracle SQL Developer)：
- SQL-DEVELOPER修改用户的操作界面： 
![Image text](https://github.com/WwhKiller/Oracle/blob/master/test2/QQ%E6%88%AA%E5%9B%BE20181024094644.png)
- SQL-DEVELOPER授权对象的操作界面： 
![Image text](https://github.com/WwhKiller/Oracle/blob/master/test2/QQ%E6%88%AA%E5%9B%BE20181024094858.png)
