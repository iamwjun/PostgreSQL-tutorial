# PostgreSQL 常用命令

本文整理 PostgreSQL 日常使用中常见的命令和 SQL 语句，适合作为快速查询手册。命令分为 `psql` 客户端命令、数据库管理、表结构管理、数据操作、备份恢复、权限管理和运维排查几类。

## 1. 连接数据库

### 使用 psql 连接

```bash
psql -U 用户名 -d 数据库名 -h 主机地址 -p 端口号
```

示例：

```bash
psql -U postgres -d mydb -h localhost -p 5432
```

参数说明：

- `-U`：指定数据库用户。
- `-d`：指定要连接的数据库。
- `-h`：指定数据库服务器地址，本机一般为 `localhost`。
- `-p`：指定端口，PostgreSQL 默认端口是 `5432`。

### 切换数据库

在 `psql` 中切换到另一个数据库：

```sql
\c 数据库名
```

示例：

```sql
\c mydb
```

## 2. psql 常用元命令

`psql` 元命令以反斜杠 `\` 开头，不需要以分号结尾。

| 命令 | 说明 |
| --- | --- |
| `\l` | 查看所有数据库 |
| `\c 数据库名` | 连接或切换数据库 |
| `\dt` | 查看当前数据库中的表 |
| `\d 表名` | 查看表结构 |
| `\du` | 查看用户和角色 |
| `\dn` | 查看 schema |
| `\df` | 查看函数 |
| `\dv` | 查看视图 |
| `\x` | 切换扩展显示模式，适合查看宽表结果 |
| `\timing` | 开启或关闭 SQL 执行耗时显示 |
| `\q` | 退出 `psql` |

## 3. 数据库管理

### 创建数据库

```sql
CREATE DATABASE 数据库名;
```

示例：

```sql
CREATE DATABASE mydb;
```

### 删除数据库

```sql
DROP DATABASE 数据库名;
```

示例：

```sql
DROP DATABASE mydb;
```

注意：删除数据库会永久删除其中的表和数据，执行前应确认已经备份。

### 查看当前数据库

```sql
SELECT current_database();
```

### 查看当前用户

```sql
SELECT current_user;
```

## 4. 表结构管理

### 创建表

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

说明：

- `SERIAL`：自增整数类型，常用于主键。
- `PRIMARY KEY`：主键约束。
- `NOT NULL`：字段不能为空。
- `UNIQUE`：字段值唯一。
- `DEFAULT`：字段默认值。

### 查看表结构

```sql
\d users
```

### 修改表名

```sql
ALTER TABLE 原表名 RENAME TO 新表名;
```

示例：

```sql
ALTER TABLE users RENAME TO app_users;
```

### 添加字段

```sql
ALTER TABLE 表名 ADD COLUMN 字段名 数据类型;
```

示例：

```sql
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
```

### 修改字段类型

```sql
ALTER TABLE 表名 ALTER COLUMN 字段名 TYPE 新数据类型;
```

示例：

```sql
ALTER TABLE users ALTER COLUMN phone TYPE VARCHAR(50);
```

### 删除字段

```sql
ALTER TABLE 表名 DROP COLUMN 字段名;
```

示例：

```sql
ALTER TABLE users DROP COLUMN phone;
```

### 删除表

```sql
DROP TABLE 表名;
```

示例：

```sql
DROP TABLE users;
```

## 5. 数据操作

### 插入数据

```sql
INSERT INTO users (name, email)
VALUES ('Alice', 'alice@example.com');
```

### 查询数据

```sql
SELECT * FROM users;
```

常用查询条件：

```sql
SELECT * FROM users WHERE id = 1;
SELECT * FROM users WHERE name LIKE 'A%';
SELECT * FROM users ORDER BY created_at DESC;
SELECT * FROM users LIMIT 10 OFFSET 0;
```

说明：

- `WHERE`：过滤数据。
- `LIKE`：模糊匹配，`%` 表示任意字符。
- `ORDER BY`：排序。
- `LIMIT`：限制返回行数。
- `OFFSET`：跳过指定行数，常用于分页。

### 更新数据

```sql
UPDATE users
SET name = 'Bob'
WHERE id = 1;
```

注意：执行 `UPDATE` 时应尽量带上 `WHERE` 条件，否则会更新整张表。

### 删除数据

```sql
DELETE FROM users
WHERE id = 1;
```

注意：执行 `DELETE` 时应尽量带上 `WHERE` 条件，否则会删除整张表数据。

### 清空表数据

```sql
TRUNCATE TABLE users;
```

如果需要同时重置自增 ID：

```sql
TRUNCATE TABLE users RESTART IDENTITY;
```

## 6. 索引管理

### 创建索引

```sql
CREATE INDEX idx_users_email ON users (email);
```

索引用于提升查询速度，适合经常出现在 `WHERE`、`JOIN`、`ORDER BY` 中的字段。

### 创建唯一索引

```sql
CREATE UNIQUE INDEX idx_users_email_unique ON users (email);
```

### 删除索引

```sql
DROP INDEX 索引名;
```

示例：

```sql
DROP INDEX idx_users_email;
```

## 7. 用户和权限管理

### 创建用户

```sql
CREATE USER 用户名 WITH PASSWORD '密码';
```

示例：

```sql
CREATE USER app_user WITH PASSWORD 'strong_password';
```

### 修改用户密码

```sql
ALTER USER 用户名 WITH PASSWORD '新密码';
```

### 给用户授权数据库访问

```sql
GRANT CONNECT ON DATABASE 数据库名 TO 用户名;
```

### 给用户授权表权限

```sql
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLE 表名 TO 用户名;
```

### 给用户授权 schema 权限

```sql
GRANT USAGE ON SCHEMA public TO 用户名;
```

### 授权某个 schema 下所有表

```sql
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO 用户名;
```

### 撤销权限

```sql
REVOKE 权限 ON 对象 FROM 用户名;
```

示例：

```sql
REVOKE INSERT ON TABLE users FROM app_user;
```

## 8. 备份和恢复

### 使用 pg_dump 备份单个数据库

```bash
pg_dump -U 用户名 -h 主机地址 -p 端口号 -d 数据库名 -f 备份文件.sql
```

示例：

```bash
pg_dump -U postgres -h localhost -p 5432 -d mydb -f mydb_backup.sql
```

### 恢复 SQL 备份

```bash
psql -U 用户名 -d 数据库名 -f 备份文件.sql
```

示例：

```bash
psql -U postgres -d mydb -f mydb_backup.sql
```

### 备份为自定义格式

```bash
pg_dump -U 用户名 -d 数据库名 -F c -f 备份文件.dump
```

### 使用 pg_restore 恢复自定义格式备份

```bash
pg_restore -U 用户名 -d 数据库名 备份文件.dump
```

## 9. 常用排查命令

### 查看当前连接

```sql
SELECT pid, usename, datname, client_addr, state, query
FROM pg_stat_activity;
```

### 查看正在执行的 SQL

```sql
SELECT pid, now() - query_start AS duration, query
FROM pg_stat_activity
WHERE state = 'active';
```

### 终止指定连接

```sql
SELECT pg_terminate_backend(pid);
```

示例：

```sql
SELECT pg_terminate_backend(12345);
```

### 查看表大小

```sql
SELECT pg_size_pretty(pg_total_relation_size('users')) AS total_size;
```

### 查看数据库大小

```sql
SELECT pg_size_pretty(pg_database_size('mydb')) AS database_size;
```

### 查看慢查询配置

```sql
SHOW log_min_duration_statement;
```

设置慢查询日志阈值示例，单位是毫秒：

```sql
SET log_min_duration_statement = 1000;
```

## 10. 事务操作

### 开启事务

```sql
BEGIN;
```

### 提交事务

```sql
COMMIT;
```

### 回滚事务

```sql
ROLLBACK;
```

示例：

```sql
BEGIN;

UPDATE users
SET email = 'new@example.com'
WHERE id = 1;

ROLLBACK;
```

说明：在执行重要的更新或删除操作前，可以先开启事务，确认结果无误后再 `COMMIT`，如果发现问题则 `ROLLBACK`。

## 11. 实用技巧

### 查看 SQL 执行计划

```sql
EXPLAIN SELECT * FROM users WHERE email = 'alice@example.com';
```

查看实际执行耗时：

```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'alice@example.com';
```

### 导入 CSV 文件

```sql
\copy users(name, email) FROM 'users.csv' WITH CSV HEADER;
```

### 导出查询结果为 CSV

```sql
\copy (SELECT * FROM users) TO 'users_export.csv' WITH CSV HEADER;
```

### 查看 PostgreSQL 版本

```sql
SELECT version();
```

只查看服务端版本号：

```sql
SHOW server_version;
```
