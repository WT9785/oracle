## 学号：201810414218姓名：王顺顺    班级：2

# 实验4：对象管理

## 实验目的：
了解Oracle表和视图的概念，学习使用SQL语句Create Table创建表，学习Select语句插入，修改，删除以及查询数据，学习使用SQL语句创建视图，学习部分存储过程和触发器的使用。
## - 实验场景：
假设有一个生产某个产品的单位，单位接受网上订单进行产品的销售。通过实验模拟这个单位的部分信息：员工表，部门表，订单表，订单详单表。

## 实验内容：
### 录入数据：
要求至少有1万个订单，每个订单至少有4个详单。至少有两个部门，每个部门至少有1个员工，其中只有一个人没有领导，一个领导至少有一个下属，并且它的下属是另一个人的领导（比如A领导B，B领导C）。

###  序列的应用
插入ORDERS和ORDER_DETAILS 两个表的数据时，主键ORDERS.ORDER_ID, ORDER_DETAILS.ID的值必须通过序列SEQ_ORDER_ID和SEQ_ORDER_ID取得，不能手工输入一个数字。

###  触发器的应用：
维护ORDER_DETAILS的数据时（insert,delete,update）要同步更新ORDERS表订单应收货款ORDERS.Trade_Receivable的值。

###  查询数据：
    1.查询某个员工的信息
    2.递归查询某个员工及其所有下属，子下属员工。
    3.查询订单表，并且包括订单的订单应收货款: Trade_Receivable= sum(订单详单表.ProductNum*订单详单表.ProductPrice)- Discount。
    4.查询订单详表，要求显示订单的客户名称和客户电话，产品类型用汉字描述。
    5.查询出所有空订单，即没有订单详单的订单。
    6.查询部门表，同时显示部门的负责人姓名。
    7.查询部门表，统计每个部门的销售总金额。

## 表结构

- 部门表DEPARTMENTS,表空间：USERS

| 编号 | 字段名          | 数据类型          | 可以为空 | 注释           |
| ---- | --------------- | ----------------- | -------- | -------------- |
| 1    | DEPARTMENT_ID   | NUMBER(6,0)       | NO       | 部门ID，主键   |
| 2    | DEPARTMENT_NAME | VARCHAR2(40 BYTE) | NO       | 部门名称，非空 |

- 产品表PRODUCTS,表空间：USERS

| 编号 | 字段名       | 数据类型          | 可以为空 | 注释                               |
| ---- | ------------ | ----------------- | -------- | ---------------------------------- |
| 1    | PRODUCT_NAME | VARCHAR2(40 BYTE) | NO       | 产品名称，产品表的主键             |
| 2    | PRODUCT_TYPE | VARCHAR2(40 BYTE) | NO       | 产品类型，只能取值：耗材,手机,电脑 |

- 员工表EMPLOYEES,表空间：USERS

| 编号 | 字段名        | 数据类型          | 可以为空 | 注释                                                         |
| ---- | ------------- | ----------------- | -------- | ------------------------------------------------------------ |
| 1    | EMPLOYEE_ID   | NUMBER(6,0)       | NO       | 员工ID，员工表的主键。                                       |
| 2    | NAME          | VARCHAR2(40 BYTE) | NO       | 员工姓名，不能为空，创建不唯一B树索引。                      |
| 3    | EMAIL         | VARCHAR2(40 BYTE) | YES      | 电子信箱                                                     |
| 4    | PHONE_NUMBER  | VARCHAR2(40 BYTE) | YES      | 电话                                                         |
| 5    | HIRE_DATE     | DATE              | NO       | 雇佣日期                                                     |
| 6    | SALARY        | NUMBER(8,2)       | YES      | 月薪，必须>0                                                 |
| 7    | MANAGER_ID    | NUMBER(6,0)       | YES      | 员工的上司，是员工表EMPOLYEE_ID的外键，MANAGER_ID不能等于EMPLOYEE_ID,即员工的领导不能是自己。主键删除时MANAGER_ID设置为空值。 |
| 8    | DEPARTMENT_ID | NUMBER(6,0)       | YES      | 员工所在部门，是部门表DEPARTMENTS的外键                      |
|9|PHOTO|BLOB|YES|员工照片

- 订单表ORDERS, 表空间：分区表：USERS,USERS02

| 编号 | 字段名           | 数据类型          | 可以为空 | 注释                                                         |
| ---- | ---------------- | ----------------- | -------- | ------------------------------------------------------------ |
| 1    | ORDER_ID         | NUMBER(10,0)      | NO       | 订单编号，主键，值来自于序列：SEQ_ORDER_ID                   |
| 2    | CUSTOMER_NAME    | VARCHAR2(40 BYTE) | NO       | 客户名称，B树索引                                            |
| 3    | CUSTOMER_TEL     | VARCHAR2(40 BYTE) | NO       | 客户电话                                                     |
| 4    | ORDER_DATE       | DATE              | NO       | 订单日期，根据该属性分区存储：2015年及以前的数据存储在USERS表空间，2016年及以后的数据存储在USERS02表空间中。 |
| 5    | EMPLOYEE_ID      | NUMBER(6,0)       | NO       | 订单经手人，员工表EMPLOYEES的外键                            |
| 6    | DISCOUNT         | Number(8,2)       | YES      | 订单整体优惠金额。默认值为0                                  |
| 7    | TRADE_RECEIVABLE | Number(8,2)       | YES      | 订单应收货款，默认为0，Trade_Receivable= sum(订单详单表.Product_Num*订单详单表.Product_Price)- Discount |

- 订单详单表ORDER_DETAILS, 表空间：分区表：USERS,USERS02，分区参照ORDERS表。

| 编号 | 字段名        | 数据类型          | 可以为空 | 注释                                           |
| ---- | ------------- | ----------------- | -------- | ---------------------------------------------- |
| 1    | ID            | NUMBER(10,0)      | NO       | 本表的主键，值来自于序列：SEQ_ORDER_DETAILS_ID |
| 2    | ORDER_ID      | NUMBER(10,0)      | NO       | 所属的订单号，订单表ORDERS的外键               |
| 4    | PRODUCT_NAME  | VARCHAR2(40 BYTE) | NO       | 产品名称, 是产品表PRODUCTS的外键               |
| 5    | PRODUCT_NUM   | NUMBER(8,2)       | NO       | 产品销售数量，必须>0                           |
| 6    | PRODUCT_PRICE | NUMBER(8,2)       | NO       | 产品销售价格                                   |

- 数据关系图如下
![image-20210420170955810](test4.assets/image-20210420170955810.png)

## 实验步骤

1.创建一个表空间USERS02

```sql
Create Tablespace Users02
datafile
'/home/oracle/app/oracle/oradata/orcl/pdborcl/pdbtest_users02_1.dbf'
  SIZE 100M AUTOEXTEND ON NEXT 256M MAXSIZE UNLIMITED,
'/home/oracle/app/oracle/oradata/orcl/pdborcl/pdbtest_users02_2.dbf'
  SIZE 100M AUTOEXTEND ON NEXT 256M MAXSIZE UNLIMITED
EXTENT MANAGEMENT LOCAL SEGMENT SPACE MANAGEMENT AUTO;
```

![image-20210420171359738](test4.assets/image-20210420171359738.png)

2.创建用户study

```sql
CREATE USER STUDY IDENTIFIED BY 123
DEFAULT TABLESPACE "USERS"
TEMPORARY TABLESPACE "TEMP"
```

![image-20210420172356747](test4.assets/image-20210420172356747.png)

3.分配用户<u>STUDY</u>表空间

```sql
ALTER USER STUDY QUOTA UNLIMITED ON USERS;
ALTER USER STUDY QUOTA UNLIMITED ON USERS02;
ALTER USER STUDY ACCOUNT UNLOCK;
```

![image-20210420172516678](test4.assets/image-20210420172516678.png)

4.授予<u>STUDY</u>相应权限

```sql
GRANT "CONNECT" TO STUDY WITH ADMIN OPTION;
GRANT "RESOURCE" TO STUDY WITH ADMIN OPTION;
ALTER USER STUDY DEFAULT ROLE "CONNECT","RESOURCE";
```

![image-20210420172622843](test4.assets/image-20210420172622843.png)

5.删除表和序列（同时会一起删除主外键、触发器、程序包。）

![image-20210420172759800](test4.assets/image-20210420172759800.png)

6.创建DEPARTMENTS表

![image-20210420172847173](test4.assets/image-20210420172847173.png)

7.创建EMPLOYEES表并添加索引、字段等

![image-20210420173037112](test4.assets/image-20210420173037112.png)

8.创建PRODUCTS表

![image-20210420173115285](test4.assets/image-20210420173115285.png)

9.创建全局临时表ORDER_ID_TEMP并添加触发器Comment

![image-20210420173252758](test4.assets/image-20210420173252758.png)

10.创建订单表ORDERS

![image-20210420173433467](test4.assets/image-20210420173433467.png)

11.创建本地分区索引ORDERS_INDEX_DATE

![image-20210420173553203](test4.assets/image-20210420173553203.png)

12.创建其余索引

![image-20210420173625273](test4.assets/image-20210420173625273.png)

13.创建索引ORDER_DETAILS_ORDER使整个订单的详单存放在一起

![image-20210420173724123](test4.assets/image-20210420173724123.png)

14.创建三个触发器

![image-20210420173852708](test4.assets/image-20210420173852708.png)

15.批量插入订单数据之前，禁用触发器

![image-20210420173924472](test4.assets/image-20210420173924472.png)

16.创建触发器

![image-20210420174055962](test4.assets/image-20210420174055962.png)

17.创建触发器ORDER_DETAILS_ROW_TRIG

![image-20210420174226594](test4.assets/image-20210420174226594.png)

18.创建两个序列

![image-20210420174402537](test4.assets/image-20210420174402537.png)

19.创建视图VIEW_ORDER_DETAILS

![image-20210420174454238](test4.assets/image-20210420174454238.png)

20.插入DEPARTMENTS，EMPLOYEES数据

![image-20210420174633858](test4.assets/image-20210420174633858.png)

21.批量插入订单数据，注意ORDERS.TRADE_RECEIVABLE（订单应收款）的自动计算,注意插入数据的速度，2千万条记录，插入的时间是：18100秒（约5小时）

![image-20210420174717711](test4.assets/image-20210420174717711.png)