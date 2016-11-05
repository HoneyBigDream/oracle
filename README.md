问题一 : sql命令连接到数据库
sqlplus dzjc_zhejiang/12345678@localhost/orcl

问题二：数据库导出表格和导入表格
(1)导入本地数据库文件
imp dzjc_zhejiang/12345678@orcl file=d:\dzjc_zhejiang.dmp full=y ignore=y buffer=81920
(2)导入远程数据库文件
imp dzjc_zhejiang/12345678@10.21.1.160/orcl file=d:\dzjc_zhejiang.dmp full=y ignore=y buffer=81920
(3)数据库文件本地导出
exp dzjc_zhejiang/12345678@orcl file=d:\dzjc_zhejiang.dmp   数据和表格
exp dzjc_zhejiang/12345678@orcl file=d:\dzjc_zhejiang.dmp rows=n  只要表格
(4)数据库文件远程导出
exp dzjc_zhejiang/12345678@10.21.1.160/orcl file=d:\dzjc_zhejiang.dmp
(5)导出指定的表格
exp dzjc_zhejiang/1234567@localhost/orcl file=f:/3.dmp tables=（tables1,tables2） 
(6)清空表数据
truncate table abc(表名)

问题三：账号锁定与解锁账号
ALTER USER dzjc_zhejiang ACCOUNT UNLOCK;
问题四：创建数据库表空间和用户
/*分为四步 */
/*第1步：创建数据表空间  */
create tablespace user_data  
logging  
datafile 'D:\oracle\oradata\Oracle9i\user_data.dbf' 
size 50m  
autoextend on  
next 50m maxsize 20480m  
extent management local;  
/*第2步：创建临时表空间  */
create temporary tablespace user_temp  
tempfile 'D:\oracle\oradata\Oracle9i\user_temp.dbf' 
size 50m  
autoextend on  
next 50m maxsize 20480m  
extent management local;  
/*第3步：创建用户并指定表空间  */
create user username identified by password  
default tablespace user_data  
temporary tablespace user_temp;  
/*第4步：给用户授予权限  */
grant connect,resource,dba to username;  
下面的代码和上面一样，可以直接复制。
--创建永久表空间
create tablespace userSpace  --表空间名称
datafile 'C:\app\yeduanqiao\oradata\dbname\useSpacer.dbf'    --文件路径及文件名
size 50M   --表空间大小
AUTOEXTEND ON NEXT 50M   --每次自动扩展50M
--创建临时表空间
create temporary tablespace userTemp
tempfile  'C:\app\yeduanqiao\oradata\dbname\userTemp.dbf'
size 50M
查看表空间信息
select tablespace_name,file_id,file_name,bytes
from dba_data_files
order by file_id
对表空间进行扩容
alter database datafile 'e:\oracle\dzjc.dbf' autoextend on next 5M maxsize 100M;
ALTER DATABASE DATAFILE 'C:\SDE.dbf' AUTOEXTEND ON NEXT 100M MAXSIZE UNLIMITED或
ALTER TABLESPACE SDE AUTOEXTEND ON NEXT 100M MAXSIZE UNLIMITED

问题五：数据库表分区和Range分区

(1)创建表分区，用时间进行分区
create table t_test (
   pk_id                number(30)                      not null,
  add_date_time        DATE,
   constraint PK_T_TEST primary key (pk_id)
)
PARTITION BY RANGE (add_date_time)
(
  PARTITION PMI VALUES LESS THAN (TO_DATE('2014-01-01 00:00:00','yyyy-mm-ddhh24:mi:ss')) TABLESPACE DZJC,
  PARTITION P201401 VALUES LESS THAN (TO_DATE('2014-02-01 00:00:00','yyyy-mm-ddhh24:mi:ss')) TABLESPACE DZJC,
  PARTITION P201402 VALUES LESS THAN (TO_DATE('2014-03-01 00:00:00','yyyy-mm-dd hh24:mi:ss'))TABLESPACE DZJC,
  PARTITION P201403 VALUES LESS THAN (TO_DATE('2014-04-01 00:00:00','yyyy-mm-ddhh24:mi:ss')) TABLESPACE DZJC,
  PARTITION P201404 VALUES LESS THAN (TO_DATE('2014-05-01 00:00:00','yyyy-mm-dd hh24:mi:ss'))TABLESPACE DZJC,
  PARTITION P201405 VALUES LESS THAN (TO_DATE('2014-06-01 00:00:00','yyyy-mm-ddhh24:mi:ss')) TABLESPACE DZJC,
  PARTITION P201406 VALUES LESS THAN (TO_DATE('2014-07-01 00:00:00','yyyy-mm-dd hh24:mi:ss'))TABLESPACE DZJC,
  PARTITION P201407 VALUES LESS THAN (TO_DATE('2014-08-01 00:00:00','yyyy-mm-ddhh24:mi:ss')) TABLESPACE DZJC,
  PARTITION P201408 VALUES LESS THAN (TO_DATE('2014-09-01 00:00:00','yyyy-mm-dd hh24:mi:ss'))TABLESPACE DZJC
)

问题六：表锁与解锁表
杀死用户被锁进程：
  select object_name as 对象名称,s.sid,s.serial#,p.spid as 系统进程号
  from v$locked_object l , dba_objects o , v$session s , v$process p
  where l.object_id=o.object_id and l.session_id=s.sid and s.paddr=p.addr;
  alter system kill session '174,10982';  
问题七：常用函数                           

(1 ) substr 函数  截取字符串  substr('aa',0,4)
(2)  nvl 函数,判断是否是空值，将空值转换为0 nvl(aaa,0)
(3)  lower(name)小写函数  select lower(pwd) from t_sys_user u
(4)  upper(name)大写函数  select upper(u.loginid) from t_sys_user u
(5)  length(name)求长度   select length(pwd) from t_sys_user u
(6)  subStr(name,1,3)截取函数      select subStr(pwd,2,3),pwd from t_sys_user u
(7 ) inStr(name) 字符串包含函数  select Instr(pwd,'D55AD283AA'),pwd from t_sys_user u
(8)  replace(pwd,'25D55AD','009990')  pwd为源字符串 ，用009990替换25D55AD       select replace(pwd,'25D55AD','009990'),pwd from t_sys_user u
(9)  round 四舍五入  round(aa,3) 四舍五入函数（会四舍五入）
demo:
    select round((sum(case when r.curstatustype = '5' then 1 else 0 end ) /
    sum(case when r.curstatustype = '2' then 1 else 0 end)),3)
    from t_engine_deal_source s, t_engine_deal_result r
    where s.busiid = r.rbusiid
    and r.source_id = s.id
    and r.isvalidity = '1'
(10)decode函数  select decode(性别，男，1，0），decode(性别，女，1，0） from 表  (分开)
select decode(u.sex,'1','女','0','男','空') sex from t_sys_user u  （合并）
            demo:   select sum(decode(r.curstatustype， '5' ，1，0)) redCount,
                         sum(decode(r.curstatustype， '2' ，1，0)) yellowCount
                         from t_engine_deal_source s, t_engine_deal_result r
                         where s.busiid = r.rbusiid
                           and r.source_id = s.id
                           and r.isvalidity = '1'
(11) case when 字段 then 1 else 0
            demo:
                      select sum(case when r.curstatustype = '5' then 1 else 0 end ) redCount  ,
                      sum(case when r.curstatustype = '2' then 1 else 0 end) yellowCount
                      from t_engine_deal_source s, t_engine_deal_result r
                      where s.busiid = r.rbusiid
                      and r.source_id = s.id
                      and r.isvalidity = '1'
(12)truck(aa,3) 保留几位函数（不会四舍五入）
                   select trunc((sum(case when r.curstatustype = '5' then 1 else 0 end ) 
                   sum(case when r.curstatustype = '2' then 1 else 0 end)),3)
                   from t_engine_deal_source s, t_engine_deal_result r
                   where s.busiid = r.rbusiid
                   and r.source_id = s.id
                   and r.isvalidity = '1'
(13)to_char(aa,'yyyyMMdd HH:mm:ss')  
                  select to_char(s.createdate,'yyyyMMdd HH:mm:ss')  from t_busi_index_source s 
(14)to_date(aa,'yyyyMMdd HH:mm:ss')
(15) to_number(c.orderno)
                    demo   select to_number(c.orderno) orderno
                    from t_config_rule_compute c
                    order by c.orderno desc

问题八：给用户分配创建视图的权限
            grant select any table,create view to ll_fx_cms;

问题九： 修改字符串的某一位
            update t_busi_index_source s set s.expscanflag=substr(to_char(s.expscanflag),1,1)||'N'||substr(to_char(s.expscanflag,),3);
            update t_busi_index_source s set s.expscanflag=substr(to_char(s.expscanflag),1,3)||'N'||substr(to_char(s.expscanflag,),5);
            update t_busi_index_source s set s.expscanflag=substr(to_char(s.expscanflag),1,9)||'N'||substr(to_char(s.expscanflag,),10);


问题十: 查看用户所属的表空间和临时表空间
select username,default_tablespace,temporary_tablespace from user_users;

问题十一： ORACLE 创建序列
create sequence DEP_ID_SEQ
increment by 1
start with 1
maxvalue 999999999

问题十二 ：  查看ORACLE下的所有用户账号
select * from dba_users;

问题十三： 删除用户
drop user dzjc_cz cascade
