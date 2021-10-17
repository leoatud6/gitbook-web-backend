# MySQL

## Overview

 系统变量： 300多个

 Mysql是专门存储数据的，查询的数据只能是一行数据，没有数组





## Grammer

```sql
begin 
 ....
end;

while do
 ...
end while;

if 
 ...
else
 ...
end if;

//内置函数
char_length();
length();
concat();
instr();
left();
trim();
Now();
Curdate();
Datediff();


//触发器，事件trigger

```



![](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MFqvIOP6F1uRQ6y7Bto%2Fuploads%2Fp7jzYuMmQRZa3e95ENVn%2Ffile.png?alt=media)



### Mysql

#### InnoDB: storage engine

dirty read, non-repeatable read, fantom read

![](<../.gitbook/assets/image (104).png>)

#### SQL执行顺序: from > where > group by > having > select > order by

#### undoLog: rollback / redoLog(by InnoDB): for persistence use

 先写log, 再写disk --> 保证数据一致性(rollback操作)

**B+ tree:** primary key data
