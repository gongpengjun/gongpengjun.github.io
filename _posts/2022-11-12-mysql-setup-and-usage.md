---

layout: post
title: MySQL安装和使用
date: 2022-11-12 09:00:00
categories: MySQL
---

使用docker-compose安装MySQL数据库，初始化表、登录、查询。

## 1、前置条件

先安装docker和docker-compose。

### 1.1、`docker`

```shell
$ docker --version
Docker version 20.10.12, build e91ed57
```

### 1.2、`docker-compose`

```shell
$ docker-compose --version
docker-compose version 1.29.2, build 5becea4c
```

## 2、安装文件

### 2.1、目录结构

```shell
$ tree mysql_setup
mysql_setup
├── docker
│  ├── config
│  │  └── my.cnf
│  ├── data
│  └── initdb
├── docker-compose.yml
├── mysql.sh -> tools/mysql_user_gongpengjun.sh
├── run.md
└── tools
    ├── mysql_user_gongpengjun.sh
    ├── mysql_user_root.sh
    ├── show_mysql_uptime.sql    
    ├── rows_count.sh
    ├── rows_sample.sh
    ├── table_users_create.sql
    ├── table_users_init.sql
    └── table_users_show.sql
```

### 2.2、`my.cnf`

MySQL配置文件，简单配置即可。

`docker/config/my.cnf`:

```shell
[mysqld]
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci

[client]
default-character-set=utf8mb4
```

### 2.3、`data`

MySQL数据存储目录，留空即可。

### 2.4、`initdb`

存放MySQL初始化SQL和初始化脚本，可以留空。

### 2.5、`docker-compose.yml`

docker-compose配置文件。

`docker-compose.yml`:

```yaml
version: '3'

services:
  # MySQL
  db:
    image: mysql:5.7
    container_name: baby_mysql
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: baby_database
      MYSQL_USER: gongpengjun
      MYSQL_PASSWORD: g1p2j3
      TZ: 'Asia/Shanghai'
    command: mysqld --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
    volumes:
    - ./docker/data:/var/lib/mysql
    - ./docker/config/my.cnf:/etc/mysql/conf.d/my.cnf
    - ./docker/initdb:/docker-entrypoint-initdb.d
    ports:
    - 3307:3306

  # phpMyAdmin
  phpmyadmin:
    container_name: baby_phpmyadmin
    image: phpmyadmin/phpmyadmin
    environment:
    - PMA_ARBITRARY=1
    - PMA_HOSTS=baby_mysql
    - PMA_USER=root
    - PMA_PASSWORD=root
    ports:
    - 3308:80
```


## 3、安装MySQL

### 3.1、启动


```shell
$ docker-compose up -d
```

### 3.2、登录MySQL - `root`

#### 3.2.1、root账户登录脚本

`tools/mysql_user_root.sh`:

```shell
#!/bin/bash

script_name=`basename "$0"`
if [ $# -gt 1 ] ; then
  echo "USAGE: ./${script_name} [optional_query.sql]"
  exit;
fi

if [ $# = 1 ] ; then
	# --batch (-B) --skip-column-names (-N) --execute (-e)
	# echo "--execute: $1"
	MYSQL_PWD=root mysql -h127.0.0.1 -P3307 -uroot -BNe "SOURCE $1"
else
	MYSQL_PWD=root mysql -h127.0.0.1 -P3307 -uroot
fi
```

注：对于root用户，环境变量`MYSQL_PWD`和`docker-compose.yml`文件中配置的`MYSQL_ROOT_PASSWORD`相同

#### 3.2.2、查询SQL

`tools/show_mysql_uptime.sql`:

```sql
SHOW /*!50002 GLOBAL */ STATUS LIKE 'Uptime'
# show mysql uptime in unit seconds
```

#### 3.2.3、实际执行

```shell
$ tools/mysql_user_root.sh tools/show_mysql_uptime.sql
Uptime	119
```

### 3.3、登录MySQL - 普通用户

#### 3.3.1、普通用户登录脚本

`tools/mysql_user_gongpengjun.sh`:

```shell
#!/bin/bash

script_name=`basename "$0"`
if [ $# -gt 1 ] ; then
  echo "USAGE: ./${script_name} [optional_query.sql]"
  exit;
fi

if [ $# = 1 ] ; then
	# --batch (-B) --skip-column-names (-N) --execute (-e)
	# echo "--execute: $1"
	MYSQL_PWD=g1p2j3 mysql -h127.0.0.1 -P3307 -ugongpengjun baby_database -BNe "SOURCE $1"
else
	MYSQL_PWD=g1p2j3 mysql -h127.0.0.1 -P3307 -ugongpengjun baby_database
fi
```

注：用户名和密码参考`docker-compose.yml`文件中配置的`MYSQL_USER`和`MYSQL_PASSWORD`

#### 3.3.2、常用命令 - `mysql.sh`

```shell
$ ln -s tools/mysql_user_gongpengjun.sh mysql.sh
$ ls -l mysql.sh
lrwxr-xr-x  1 gongpengjun  staff  31 11 13 23:04 mysql.sh -> tools/mysql_user_gongpengjun.sh
```

#### 3.3.3、实际执行

```shell
$ tools/mysql_user_gongpengjun.sh tools/show_mysql_uptime.sql
Uptime	158
$ ./mysql.sh tools/show_mysql_uptime.sql
Uptime	191
```

### 3.4、登录phpmyadmin

[http://localhost:3308/](http://localhost:3308/)

## 4、初始化MySQL表 `users`

### 4.1、表`users` - 创建

`tools/table_users_create.sql`:

```sql
USE `baby_database`;
CREATE TABLE `users` (
  `id` bigint(11) unsigned NOT NULL AUTO_INCREMENT COMMENT 'id',
  `avatar_url` varchar(255) NOT NULL DEFAULT 'https://gongpengjun.com/baby-public/a.png' COMMENT '用户头像',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

实际执行：

```sql
$ ./mysql.sh tools/table_users_create.sql
```

### 4.2、表`users` - 填数据

`tools/table_users_init.sql`:

```sql
USE `baby_database`;
INSERT INTO `users` (`avatar_url`) VALUES('https://old.gongpengjun.com/baby-public/a.png');
INSERT INTO `users` (`avatar_url`) VALUES('https://old.gongpengjun.com/baby-public/b.png');
INSERT INTO `users` (`avatar_url`) VALUES('https://old.gongpengjun.com/baby-public/c.png');
```

实际执行：

```shell
$ ./mysql.sh tools/table_users_init.sql
```

### 4.3、表`users` - 查看

#### 4.3.1、数据表采样

`tools/rows_sample.sh`:

```shell
#!/bin/sh
if [ $# -ne 2 ]; then
  echo "Usage: ./count_rows.sh {database_name} {table_name}";
  exit 1;
fi

database_name=$1
table_name=$2
MYSQL_PWD=g1p2j3 mysql -h127.0.0.1 -P3307 -ugongpengjun ${database_name} <<MYSQL_INPUT
select
  *
from
  ${table_name}
limit
  3;
MYSQL_INPUT
```

数据库 `baby_database` 中表`users`数据采样：

```shell
$ tools/rows_sample.sh baby_database users
id	avatar_url
1	https://old.gongpengjun.com/baby-public/a.png
2	https://old.gongpengjun.com/baby-public/b.png
3	https://old.gongpengjun.com/baby-public/c.png
```

#### 4.3.2、数据表统计

`tools/rows_count.sh`:

```shell
#!/bin/sh
if [ $# -ne 2 ]; then
  echo "Usage: ./count_rows.sh {database_name} {table_name}";
  exit 1;
fi

database_name=$1
table_name=$2

MYSQL_PWD=g1p2j3 mysql -h127.0.0.1 -P3307 -ugongpengjun ${database_name} <<MYSQL_INPUT
SELECT COUNT(*) AS 'Rows in table: ${database_name}.${table_name}' FROM ${table_name};
MYSQL_INPUT
```

数据库 `baby_database` 中表`users`行数统计：

```shell
$ tools/rows_count.sh baby_database users
Rows in table: baby_database.users
3
```

## 5、MySQL启动和停止

### 5.1、启动MySQL

```shell
$ docker-compose up -d
```

### 5.2、查看MySQL启动状态

```shell
$ docker-compose ps
```

实际执行：

```shell
gongpengjun@mbp mysql_setup$ docker-compose ps
     Name                    Command               State                 Ports
--------------------------------------------------------------------------------------------
baby_mysql        docker-entrypoint.sh mysql ...   Up      0.0.0.0:3307->3306/tcp, 33060/tcp
baby_phpmyadmin   /docker-entrypoint.sh apac ...   Up      0.0.0.0:3308->80/tcp
```

### 5.3、关闭MySQL

```shell
$ docker-compose down
```

## 6、参考资料

- [Oreilly MySQL Cookbook](https://www.oreilly.com/library/view/mysql-cookbook-2nd/059652708X/ch01s30.html)
- [docker-compose でMySQL環境簡単構築](https://qiita.com/A-Kira/items/f401aea261693c395966)
- [教程：创建具有 MySQL 和 Docker Compose 的多容器应用](https://learn.microsoft.com/zh-cn/visualstudio/docker/tutorials/tutorial-multi-container-app-mysql)
