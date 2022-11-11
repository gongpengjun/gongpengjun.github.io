---
layout: post
title: MySQL批量刷数据
date: 2022-11-11 09:00:00
categories: MySQL
---

使用Shell脚本对MySQL数据库中的数据进行批量刷新的方法。

### 0、MySQL脚本 `mysql.sh`

登录或执行SQL的脚本 `mysql.sh`:

```shell
#!/bin/bash

script_name=`basename "$0"`
if [ $# -gt 1 ] ; then
  echo "USAGE: ./${script_name} [optional_query.sql]"
  exit;
fi

if [ $# = 1 ] ; then
  # --batch (-B) --skip-column-names (-N) --execute (-e)
  echo "#execute: $1"
  MYSQL_PWD=g1p2j3 mysql -h127.0.0.1 -P3306 -ugongpengjun baby_database -BNe "SOURCE $1"
else
  MYSQL_PWD=g1p2j3 mysql -h127.0.0.1 -P3306 -ugongpengjun baby_database
fi
```

### 1、刷新前数据采样和统计

#### 1.1、`sample_record.sql`

数据采样的SQL `sample_record.sql`

```sql
select
  *
from
  `baby_database`.`users`
limit
  3;
```

#### 1.2、`count_old_host.sql`

老数据统计的SQL `count_old_host.sql`

```sql
select
  count(1) as avatar_url_old_host_count
from
  `baby_database`.`users`
where
  substring_index(substring_index(avatar_url, '/', 3), '/', -1) = 'old.gongpengjun.com';
```

#### 1.3、`count_new_host.sql`

新数据统计的SQL `count_new_host.sql`

```sql
select
  count(1) as avatar_url_new_host_count
from
  `baby_database`.`users`
where
  substring_index(substring_index(avatar_url, '/', 3), '/', -1) = 'new.gongpengjun.com';
```

#### 1.4、数据采样

```shell
$ ./mysql.sh sample_record.sql
#execute: sample_record.sql
1	https://old.gongpengjun.com/baby-public/a.png
2	https://old.gongpengjun.com/baby-public/b.png
3	https://old.gongpengjun.com/baby-public/c.png
```

#### 1.5、数据统计

```shell
$ ./mysql.sh count_old_host.sql
#execute: count_old_host.sql
3
$ ./mysql.sh count_new_host.sql
#execute: count_new_host.sql
0
```

### 2、执行数据刷新

#### 2.1、生成刷数据SQL

生成刷数据SQL的SQL `generate_old_to_new_update_sql.sql`:

```sql
select
  concat(
    'UPDATE `baby_database`.`users` ',
    'SET `avatar_url` = REPLACE(`avatar_url`, "old.gongpengjun.com", "new.gongpengjun.com") ',
    'WHERE `id` = ',
    id,
    ';'
  ) AS `baby_database.users.avatar_url.old2new.update_sql`
from
  `baby_database`.`users`
where
  substring_index(substring_index(avatar_url, '/', 3), '/', -1) = 'old.gongpengjun.com';
```

实际执行：

```shell
$ ./mysql.sh generate_old_to_new_update_sql.sql > avatar_url_host_old_to_new_update.sql
```

#### 2.2、执行刷数据SQL

生成的刷数据的SQL `avatar_url_host_old_to_new_update.sql`:

```sql
UPDATE `baby_database`.`users` SET `avatar_url` = REPLACE(`avatar_url`, "old.gongpengjun.com", "new.gongpengjun.com") WHERE `id` = 1;
UPDATE `baby_database`.`users` SET `avatar_url` = REPLACE(`avatar_url`, "old.gongpengjun.com", "new.gongpengjun.com") WHERE `id` = 2;
UPDATE `baby_database`.`users` SET `avatar_url` = REPLACE(`avatar_url`, "old.gongpengjun.com", "new.gongpengjun.com") WHERE `id` = 3;
```

实际执行：

```shell
$ ./mysql.sh avatar_url_host_old_to_new_update.sql
#execute: avatar_url_host_old_to_new_update.sql
```

### 3、刷新后数据采样和统计

#### 3.1、数据采样


```shell
$ ./mysql.sh sample_record.sql
#execute: sample_record.sql
1	https://new.gongpengjun.com/baby-public/a.png
2	https://new.gongpengjun.com/baby-public/b.png
3	https://new.gongpengjun.com/baby-public/c.png
```

#### 3.2、数据统计

```shell
$ ./mysql.sh count_old_host.sql
#execute: count_old_host.sql
0
$ ./mysql.sh count_new_host.sql
#execute: count_new_host.sql
3
```



