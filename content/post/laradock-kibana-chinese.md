---
title: "laradock kibana 设置中文语言"
silg: "laradock-kibana-chinese"
description: "在kibana配置文件kibana.yml中加入一行 i18n.locale: 'zh-CN'，然后重启容器就是中文了。"
date: "2021-03-24T08:21:32Z"
thumbnail: ""
categories:
  - "编程开发"
tags:
  - "laradock"
  - "kibana"
---

进入kibana后台：http://127.0.0.1:5601/，可以看到默认是英文语言。

#### 首先进入容器：

```shell
docker-compose exec kibana bash
```

#### 编辑kibana配置文件：

```shell
vi /usr/share/kibana/config/kibana.yml
```

#### 在文件最底部加入中文设置：

```yml
i18n.locale: "zh-CN"
```

##### 然后重启kibana容器：

```shell
docker-compose restart kibana
```

刷新后台已经是中文了。


