---
title: "MySQL每日分表定时备份shell"
slug: "mysql-backup-shell"
description: ""
date: "2020-03-03T19:57:43+08:00"
thumbnail: ""
categories:
  - "Develop"
tags:
  - "mysql"
  - "shell"
  - "crontab"
  - "分表"
  - "备份"
---

最近有个需求：项目是21G的mysql数据库，需要做每日备份归档，本来考虑做增量备份的，从时间安排考虑，还是先做个每日全量备份吧。

思路是先连接数据库，切换到要备份的库，然后取出所有表名，接下来遍历每张表，挨个导出为 .sql 文件，放到日期为命名的文件夹里，最后压缩为 .tar.gz 压缩包。

使用的是crontab做每日定时任务。

```shell
#!/bin/sh

# 临时备份路径（未压缩的文件）
OUT_DIR=/www/backup/tmp

# 压缩后的备份存放路径
TAR_DIR=/www/backup/data

# 数据库信息
URL=127.0.0.1
PORT=3306
USERNAME=root
PASSWORD=root
DBNAME=your_dbname

# 当前系统时间
DATE=`date +%Y-%m-%d`

# 指定命令所用的全路径
MYSQL=/usr/bin/mysql
MYSQLDUMP=/usr/bin/mysqldump
MYSQLADMIN=/usr/bin/mysqladmin

# 日志文件
logfile=/www/backup/data.txt

[ -d ${OUT_DIR} ] || mkdir -p ${OUT_DIR}
[ -d ${TAR_DIR} ] || mkdir -p ${TAR_DIR}

# 最终保存的数据库备份文件
TAR_BAK="your_dbname_$DATE.tar.gz"

cd ${OUT_DIR}

rm -rf ${OUT_DIR}/*

echo "${DBNAME} backup start - ${DATE}\n" >>${logfile}

# ${MYSQL} -h $URL:$PORT -u $USERNAME -p $PASSWORD -d $DBNAME -o $OUT_DIR/`$DATE +%Y-%m-%d`

TABLE=$(${MYSQL} -h${URL} -P${PORT} -u${USERNAME} -p${PASSWORD} -e "use ${DBNAME};show tables;"|sed '1d') #依次列出需要备份的表的名字
  for tname in ${TABLE}
    do
        #echo "${DBNAME}_${tname}_${DATE}"
        # 创建备份表的目录
        [ -d ${OUT_DIR}/${DATE} ] || mkdir ${OUT_DIR}/${DATE}
        ${MYSQLDUMP} --set-gtid-purged=off --single-transaction --column-statistics=0 --default-character-set=utf8 --compress-output=LZ4 --default-parallekism=3 -h${URL} -P${PORT} -u${USERNAME} -p${PASSWORD} ${DBNAME} ${tname} >${OUT_DIR}/${DATE}/${tname}.sql

        if [ $? = 0 ]
        then
            echo "${DBNAME} ${tname} backup Successful!" >>${logfile}
        else
            echo "${DBNAME} ${tname} backup fail!" >>${logfile}
        fi
    done

if [ $? = 0 ]
then
    echo "${DBNAME} backup all tables Successful!\n\n">>${logfile}
    # 压缩格式为 .tar.gz 格式
    cd ${OUT_DIR}
    tar -zcvf ${TAR_DIR}/${TAR_BAK} ${DATE}

    # 删除 15 天前的备份文件
    #find ${TAR_DIR}/ -mtime +15 -print | xargs rm -rf

    exit
else
    echo "${DBNAME} backup all tables fail!\n\n">>${logfile}
    exit 4
fi
```
