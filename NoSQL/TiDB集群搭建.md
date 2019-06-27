### TiDB集群搭建

机器准备：

4C8G 

[0  ] 192.168.20.50    22     tidb-tikv-2 [xiaosiyong] **TiKV2**

[1  ] 192.168.20.52    22     tidb-tikv-1 [xiaosiyong] **TiKV1**

[2  ] 192.168.20.59    22     tidb-tikv-3 [xiaosiyong] **TiKV3**

2C4G

[0  ] 192.168.20.55    22     tibd-pd-2 [xiaosiyong] **TiDB1、PD2**

[1  ] 192.168.20.65    22     tibd-pd-1 [xiaosiyong] **TiDB、PD1**

[2  ] go    22     tibd-pd-3 [xiaosiyong]  **monitoring_servers 、PD3、grafana**

#### 安装步骤

- 以root用户登录中控机，执行命令：

  1、yum -y install epel-release git curl sshpass

  2、yum -y install python2-pip

- 中控机创建tidb用户，生成ssh key，执行命令：

  1、创建 `tidb` 用户，useradd -m -d /home/tidb tidb

  2、设置 `tidb` 用户密码， passwd tidb    密码：ia8Moopu

  3、配置 `tidb` 用户 sudo 免密码，visudo，tidb ALL=(ALL) NOPASSWD: ALL

  4、生成 ssh key: 执行 `su` 命令从 `root` 用户切换到 `tidb` 用户下：su - tidb

#### TiDB功能验证

selet[select 选项] 
	字段列表[字段别名] /* 
from 数据源
[where条件字句]
[group by 字句]
[having 字句]
[order by 字句]
[limit 字句]



SHOW DATABASES;

 CREATE DATABASE MYTEST;

DROP DATABASE MYTEST;

单表插入10w条数据， select count(0) from employees;

select * from employees order by id desc limit 5;

select * from employees where id = 109940;

select * from employees where id >109938;

select * from employees where id between 109938 and 109942;

select * from employees where id in (109940,109941,109942,109943);

select * from employees where emp_no=576170000;

 select * from employees where emp_no>993951000 and emp_no <993971000 order by emp_no asc;

**Group by 不会像MySQL那样严格要求，查询的字段出现在聚合函数中，默认取第一条**

count有些许的不一致，

MySQL执行没问题，但在TiDB下会报错。

select max(id) as id,last_name,count(0) as q from employees  where emp_no>934674000 and id >97240 GROUP BY last_name HAVING COUNT(q)>1 order by q desc LIMIT 2；

select max(id) as id,last_name,count(0) as q from employees  where emp_no>934674000 and id >97240 GROUP BY last_name HAVING COUNT(*)>0 order by q desc LIMIT 2;





数据导入：https://www.bookstack.cn/read/pingcap-docs-cn/op-guide-migration.md











