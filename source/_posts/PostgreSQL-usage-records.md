---
title: PostgreSQL使用记录
date: 2018-08-29 22:10:07
tags: database
---

#### 安装`PostgreSQL`
仅限于Ubuntu。
```
https://www.postgresql.org/download/linux/ubuntu/
1. Create the file /etc/apt/sources.list.d/pgdg.list and add a line for the repository
deb http://apt.postgresql.org/pub/repos/apt/ xenial-pgdg main

2. Import the repository signing key, and update the package lists
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update

3. apt-get install postgresql-10

4. /usr/lib/postgresql/10/bin/pg_ctl -D /var/lib/postgresql/10/main -l logfile start

LD_LIBRARY_PATH=/usr/lib/postgresql/10/lib
PATH=/usr/lib/postgresql/10/bin
```


#### 创建新账户
```
sudo adduser dbuser
sudo su - postgres
psql
\password postgres
CREATE USER dbuser WITH PASSWORD 'password';
CREATE DATABASE exampledb OWNER dbuser;
GRANT ALL PRIVILEGES ON DATABASE exampledb to dbuser;
\q
```

#### 登录使用`PostgreSQL`
```
psql -U dbuser -d exampledb -h 127.0.0.1 -p 5432
```

#### 常用语句
```
# 创建新表
CREATE TABLE user_tbl(name VARCHAR(20), signup_date DATE);

# 插入数据
INSERT INTO user_tbl(name, signup_date) VALUES('张三', '2013-12-22');

# 选择记录
SELECT * FROM user_tbl;

# 更新数据
UPDATE user_tbl set name = '李四' WHERE name = '张三';

# 删除记录
DELETE FROM user_tbl WHERE name = '李四' ;

# 添加栏位
ALTER TABLE user_tbl ADD email VARCHAR(40);

# 更新结构
ALTER TABLE user_tbl ALTER COLUMN signup_date SET NOT NULL;

# 更名栏位
ALTER TABLE user_tbl RENAME COLUMN signup_date TO signup;

# 删除栏位
ALTER TABLE user_tbl DROP COLUMN email;

# 表格更名
ALTER TABLE user_tbl RENAME TO backup_tbl;

# 删除表格
DROP TABLE IF EXISTS backup_tbl;
```

参考[资料](http://www.ruanyifeng.com/blog/2013/12/getting_started_with_postgresql.html)。
