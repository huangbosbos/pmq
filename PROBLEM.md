# 环境
idea 2019.2
mysql 5.6
maven 3.3.9

# 问题1 数据库mysql5.6
导入数据异常，进行字符修改大小 con_str` varchar(64) NOT NULL COMMENT '数据库链接字符串'

# 问题2 数据库报错
```
2019-10-18 23:07:14,939 ERROR com.alibaba.druid.pool.DruidDataSource- create connection SQLException, url: jdbc:mysql://boss-mq:3306/mq_basic?useUnicode=true&characterEncoding=utf8&useSSL=false, errorCode 0, state null,the guid is 

java.sql.SQLException: Unable to load authentication plugin 'caching_sha2_password'.
	at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:868)
	at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:864)
```
环境：数据库使用的mysql8.0
处理：
```	
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>8.0.11</version>
		</dependency>
```
