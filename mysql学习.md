# mysql学习

##  数据库操作

1. 查看所有数据库：`SHOW DATABASES;`
2. 创建数据库：`CREATE DATABASE database_name;`
3. 选择要使用的数据库：USE database_name;`
4. 删除数据库：DROP DATABASE database_name;`

## 表操作

- `SHOW TABLES;`
- `CREATE TABLE table_name (column1 datatype1, column2 datatype2,...);`

- `DESCRIBE table_name;` 或 `SHOW COLUMNS FROM table_name;`

- `DROP TABLE table_name;` 

## 数据查询

- `SELECT * FROM table_name;`
- `SELECT column1, column2 FROM table_name;`
- `UPDATE table_name SET column1 = value1, column2 = value2,... WHERE condition;`

- `SELECT * FROM table_name WHERE condition;`
- `DELETE FROM table_name WHERE condition;`
- 

## 索引操作

- `CREATE INDEX index_name ON table_name (column_name);`
- `DROP INDEX index_name ON table_name;`

## 用户操作

- `CREATE USER 'username'@'host' IDENTIFIED BY 'password';`
- `SHOW GRANTS FOR 'username'@'host';`
- `DROP USER 'username'@'host';`



**这个操作解决了我输入正确的密码 但还是显示错误的密码**

**MySQL 服务未运行**

1. 检查 MySQL 服务状态：
   - 在 Windows 上，可以通过 “服务” 管理工具查看 MySQL 服务是否正在运行。按下 Win + R，输入 “services.msc” 并回车，在服务列表中查找 MySQL 服务，确认其状态为 “正在运行”。



