---
title: "MySql按中文（汉字）拼音首字母检索"
date: 2016-03-11T14:19:25Z
slug: "mysql-chinese-first-letter-searches"
description: "实现按拼音首字母检索一种是直接增加字段存储名称首字母，但是这样会使表都一个字段，每次录入都要转换 这是通常的做法，另一种是接下来介绍的这种，按照汉字编码排序来实现的，无需给表多增字段。"
thumbnail: ""
categories:
  - "编程开发"
tags:
  - "mysql"
---
实现按拼音首字母检索一种是直接增加字段存储名称首字母，但是这样会使表都一个字段，每次录入都要转换 这是通常的做法，另一种是接下来介绍的这种，按照汉字编码排序来实现的，无需给表多增字段。

首先我们有一个这样的数组：

```php
array(
	'A'=>'吖',
	'B'=>'八',
	'C'=>'嚓',
	'D'=>'咑',
	'E'=>'妸',
	'F'=>'发',
	'G'=>'旮',
	'H'=>'铪',
	'J'=>'丌',
	'K'=>'咔',
	'L'=>'垃',
	'M'=>'嘸',
	'N'=>'拏',
	'O'=>'噢',
	'P'=>'妑',
	'Q'=>'七',
	'R'=>'呥',
	'S'=>'仨',
	'T'=>'他',
	'W'=>'屲',
	'X'=>'夕',
	'Y'=>'丫',
	'Z'=>'帀'
);
```

查出姓名拼音首字母是a的

```sql
SELECT * FROMtbl WHERE CONVERT( name USING gbk ) COLLATE gbk_chinese_ci >='吖' AND CONVERT( name USING gbk ) COLLATE gbk_chinese_ci < '八'
```

查出姓名拼音首字母是b的

```sql
SELECT * FROMtbl WHERE CONVERT( name USING gbk ) COLLATE gbk_chinese_ci >='八' AND CONVERT( name USING gbk ) COLLATE gbk_chinese_ci < '嚓'
```

…

查出姓名拼音首字母是z的

```sql
SELECT * FROM tbl WHERE CONVERT( name USING gbk ) COLLATE gbk_chinese_ci >='z'
```

原理：GBK编码是按读音排序的，UTF-8不是按读音排序的，因此我们将汉字转为gbk编码，只要知道拼音首字母的a到b的界限就可以查出所有首字母是a的姓名，以此类推，可以按上面的数组查出。

**至于英文字母26个为什么数组只有23个，只因为汉字姓名中没有以那三个字母（I、U、V）开头的姓名，想深究的小伙伴可以参考[GBK编码表](http://ff.163.com/newflyff/gbk-list/)。**

提示：``CONVERT``是将name转换成gbk编码（上面已经解释过了），如果你本来就是gbk编码的汉字，那CONVERT这个就可以省略了,我们通常用的是utf8。

查询结果可以用片段缓存来缓存，提高速度。

实现的效果是这样的，点击相应字母，找出对应字段中文的第一个字拼音字母查询结果：

![数据库MySql按中文（汉字）拼音首字母检索](/uploads/first-letter-search.png)

