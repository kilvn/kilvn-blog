---
title: "shell下执行mysql查询导出到文件，简单高效"
slug: "mysql-export-to-file-in-shell"
description: "echo 'sql语句' | mysql -u用户名 -p密码 数据库 > /任意位置/xxx.txt"
date: "2020-03-31T23:17:04+08:00"
thumbnail: ""
categories:
  - "Develop"
tags:
  - "shell"
  - "mysql"
---

最近在做mysql多表查询生成报表之类的事情，表数据又比较大，大到一个表有几千万行两三个G大，小到几百万行一两个G，而且还要联合查询，又是group，又是子查询，索引都用不上，这样查几个小时不一定出来结果。

无意中试了下shell中执行sql并且导出到文件，平时在Navicat这种图形客户端半天都没结果的sql，在shell小到几分钟，大到个把小时，都不会等太久，而且导出的文件可以直接以TXT导入到数据库或者全部复制，粘贴到Excel中。

刚开始想着进入mysql-client执行查询然后导出到文件，结果失败了，提示没有权限创建文件，提供的导出文件路径只能在mysql安装目录下，其他位置没有权限新建文件，后来了解到用 echo + 管道符就好了，这种方法在root下面执行，不存在权限问题。

##### 导出为TXT

用法：

```shell
echo "sql语句" | mysql -h* -P* -u* -p 数据库 > /任意位置/xxx.txt
```

##### 导出为Excel

用法：

```shell
mysql -h* -P* -u* -p* 数据库 -Bse "set names utf8; select name,email from test" | sed 's/\t/","/g;s/^/"/;s/$/"/;s/\n//g' > /任意位置/xxx.csv
```

这样虽然方便，但是sql很长的时候就不那么方便了，写到shell里面再合适不过了，完整的shell示例如下：

随便建个shell文件吧，比如 export.sh

```shell
#!/bin/bash

MYSQL=/usr/bin/mysql

DB_HOST=127.0.0.1
DB_PORT=3306
DB_USER=root
DB_PASSWORD=root
DB_DATABASE=test

#OUT_FILE=/www/test.txt
OUT_FILE=/www/test.csv

SQL="select * from mysql"

# 导出为txt
#echo ${SQL} | ${MYSQL} -h${DB_HOST} -P${DB_PORT} -u${DB_USER} -p${DB_PASSWORD} ${DB_DATABASE} > ${OUT_FILE}

# 导出为csv
${MYSQL} -h${DB_HOST} -P${DB_PORT} -u${DB_USER} -p${DB_PASSWORD} ${DB_DATABASE} -Bse "set names utf8; ${SQL}" | sed 's/\t/","/g;s/^/"/;s/$/"/;s/\n//g' > ${OUT_FILE}
```

给sh文件执行权限

```
chmod +x export.sh
```

执行sh文件，开始导出了

```
./export.sh
```

写的比较简单，没有做执行结果判断。
