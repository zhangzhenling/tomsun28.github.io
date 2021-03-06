---
layout: post
title: oracle_learn
date: 2017-10-19
tag: oracle
---

## 遇到的 oracle 日常知识点

**round**

	返回一个按照指定的小数位进行四舍五入运算后的数值
	round(123.4567,0)  返回123
	round(123.4567,1)  返回123.5

**decode**

	decode(条件,值1,返回值1,值2,返回值2,....,值n,返回值n,缺省值)
	decode(table.id,'10',table.value/10,'100',table.value/100,0)

**into**  
	v_number integer;
	select count(*) into v_number from table;



**case when**

	case flag
	when '0' then 'false'
	when '1' then 'true'
	else 'others' end
	
	case
	when flag = '0' then 'false'
	when flag = '1' then 'true'
	else 'others' end

**to_date,to_char,to_number**

	字符串时间互转
	to_date('2017-10-19 13-55-20','yyyy-mm-dd hh24:mi:ss')
	to_char(sysdate,'yyyy-mm-dd hh24:mi:ss')
	to_number(字符串,'格式')

**substr**

	字符串截取
	substr('my name is tomsun28',0,5)返回'my na' 从左端第一个字符开始截取长度为5的字符串 
	substr('my name is tomsun28',1,5)返回'my na' 0和1都表示从第一个字符开始截取
	substr('my name is tomsun28',-5,5)返回'sun28' 表示从右端第5个字符开始向后截取5个字符

**`||`**

	字符串连接符 'aaaa'||'bbbb' = 'aaaabbbb'

**<>**

	不等于的标准语法，可以移植到其他平台

**execute_immediate**

	动态SQL执行语句
	execute_immediate 动态sql语句 using 绑定参数列表 returning into 输出参数列表;
	execute_immediate 'select name,salary from temp where id = 1' using id returning into p_name,p_salary;

**truncate,delete,drop**

	delete from tablename; 一条一条的删记录，会保留一个空的页
	truncate table tablename; 一次性删掉整页，不保留任何页

**nvl**

	nvl(expr1,expr2) 若expr1为null，则返回expr2，不为null则返回expr1

**merge into**

	用一个或多个数据源完成对表的数据更新与插入,有则更新,无则插入
	merge into test1 t1
	using (select userid,id from test2 where id >= 20) tw
	on (t1.userid = tw.userid)
	when matched then update set t1.id = tw.id
	when mot matched then insert(t1.userid,t1.id) values(tw.userid,tw.id);

**CURSOR**

	游标是SQL的一个内存工作区。一些情况需要将数据从存放在磁盘的数据库表调到计算机内存中处理，最后将处理结果返回数据库或显示出来，提高速度。这就用到游标，作用就是用于临时存储从数据库中提取的数据块。CURSOR类型：静态游标--显示游标和隐式游标，动态游标--REF游标,其是一种引用类型类似于指针。
	显示游标的for遍历循环
	declare
		cursor cur is select * from temp where temp.id > 0;
		cur_var temp%rowtype;
	begin
		for cur_var in cur loop
		exit when cur%notfound;
		数据处理;
	end loop;
	exception
		出错处理;
	end;
	
	带参数的cursor
	cursor c_user(c_id number)is select name from user where typeid=c_id;
	open c_user;
	    loop
	        fetch c_user into v_name;
	        exit fetch c_user%notfound;
	            do something;
	    end loop;
	close c_user;

**in/exists与not in/not exists**

	in是把外表和内表作连接,是对外表作loop循环。对于效率,查询的两个表大小相当则效率差别不大。若两个表一大一小,则子查询表大的用exists,子查询表小的用in。
	例如表A小,表B大
	select * from A where name in (select name from B)---效率低
	select * from A where exists (select name from B where name=A.name)---效率高  
	
	not in 会调用子查询,如果子查询字段任意记录有空值,则查询不会返回任何记录。尽量使用not exists。
	select * from A where name not in(select name from B)---执行结果无
	select 8 from A where not exists(select 1 from B where B.name=A.name)---执行结果正确

**级联查询**  

	select * from tablename start with subId = '3302' connect by prior subId=parentId ---查询一个节点下面的所有关联节点
	subId为子节点的ID parentId为父节点ID


<br>

- - -
- - -

<br>

**误删除表空间test的数据文件后，需要将该表空间删除重建**

	alter database datafile '对应数据文件地址' offline drop; 
	drop tablespace test;

**在sqlplus下执行sql文件**

	SQL>@/home/oracle/test.sql         @@表示当前目录下执行

**数据库数据插入注意编码格式**

	查看文件编码格式  file -i test.txt

**oracle启动，关闭，查看监听服务**

	lsnrctl start,lsnrctl stop,lsnrctl status

**启动关闭oracle数据库实例**

	sqlplus /nolog
	conn / as sysdba
	conn username
	startup  这里环境变量export要设置正确的数据库SID，就是启动环境变量ORACLE_SID所对应的那个数据库实例
	startdown

**oracle用户解锁**

	sqlplus /nolog
	conn / as sysdba
	alter user userName account unlock;



<br>
- - -
- - -
<br>

**oracle存储过程基本结构**

	create or replace procedure procedure_name(startData in integer,endData in varchar2,outData out varchar2,参数名 in/out 参数类型)is
	v_startData varchar2(10);
	v_endData   varchar2(10);
	变量名       变量类型;
	begin
	//变量赋值:
	v_startData:=startData;
	v_endData:=endData;
	//if判断
	if v_startData = 1 then
	    begin
	        do something;
	    end;
	end if;
	//while循环
	while v_startData <>1 loop
	    begin
	    do something;
	    end;
	end loop;
	
	//cursor用法见上
	
	EXCEPTION
	    when others then
	        异常错误处理;
	        do something;
	        rollback;//一般情况回滚
	end procedure_name;


<br>
<br>
<br>

*转载请注明* [from tomsun28](http://usthe.com)