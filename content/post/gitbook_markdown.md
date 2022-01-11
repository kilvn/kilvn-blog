---
title: "GitBook：使用Git+Markdown快速制作电子书"
slug: "gitbook_markdown"
description: "GitBook是一个命令行工具（Node.js库），我们可以借用该工具使用Github/Git和Markdown来制作精美的图书，但它并不是一本关于Git的教程哟。"
date: "2017-03-23T09:25:06Z"
thumbnail: ""
categories:
  - "实用工具"
tags:
  - "gitbook"
  - "markdown"
---

GitBook是一个命令行工具（Node.js库），我们可以借用该工具使用Github/Git和Markdown来制作精美的图书，但它并不是一本关于Git的教程哟。

**支持输出多种格式**

GitBook支持输出多种文档格式，如：

*   静态站点：GitBook默认输出该种格式，生成的静态站点可直接托管搭载[Github Pages](https://pages.github.com/)服务上；
*   PDF：需要安装[gitbook-pdf](https://github.com/GitbookIO/gitbook-pdf)依赖；
*   eBook：需要安装[ebook-convert](http://manual.calibre-ebook.com/cli/ebook-convert.html)；
*   单HTML网页：支持将内容输出为单页的HTML，不过一般用在将电子书格式转换为PDF或eBook的中间过程；
*   JSON：一般用于电子书的调试或元数据提取。

**结构简单**

使用GitBook制作电子书，必备两个文件：_README.md_和_SUMMARY.md_。README.md多为电子书的简介内容，SUMMARY.md用来定义电子书章节结构，如：

![使用Git+Markdown快速制作电子书](/uploads/SUMMARY_1.png)

同时，GitBook还支持嵌入JavaScript的交互式内容，未来版本会支持Python、Ruby等语言。

**GitBook项目官网**：[http://www.gitbook.io](http://www.gitbook.io/)

**GitBook Github地址**：[https://github.com/GitbookIO/gitbook](https://github.com/GitbookIO/gitbook)

废话不说，直接主题：

### gitbook安装

**安装npm**

从网站 [https://nodejs.org/#download](https://nodejs.org/#download) 下载node.js源代码（点击绿色的INSTALL），

**gitbook 安装**

```
$ npm install -g gitbook-cli
$ gitbook -V
```

查看gitbook是否安装成功。

### gitbook使用

1. 根据目录生成图书结构

1.1 README.md 与 SUMMARY编写

README.md 这个文件相当于一本Gitbook的简介。

```
$ mkdir test_gitbook
$ touch README.md
```

SUMMARY.md 这个文件是一本书的目录结构，使用Markdown语法，如我们这本书的SUMMARY.md：

```
$ touch SUMMARY.md
$ vim SUMMARY.md
```

输入

```md
 * [简介](README.md)
 * [第一章](chapter1/README.md)
 – [第一节](chapter1/section1.md)
 – [第二节](chapter1/section2.md)
 * [第二章](chapter2/README.md)
 – [第一节](chapter2/section1.md)
 – [第二节](chapter2/section2.md)
 * [结束](end/README.md)
```

1.2 生成图书结构

当这个目录文件创建好之后，我们可以使用Gitbook的命令行工具将这个目录结构生成相应的目录及文件：

```
$ gitbook init
$ tree . #查看建立的目录和文件
```

.  
├── chapter1  
│ ├── README.md  
│ ├── section1.md  
│ └── section2.md  
├── chapter2  
│ ├── README.md  
│ ├── section1.md  
│ └── section2.md  
├── end  
│ └── README.md  
├── README.md  
└── SUMMARY.md

我们可以看到，gitbook给我们生成了与SUMMARY.md所对应的目录及文件。

每个目录中，都有一个README.md文件，相当于一章的说明。

1.3 安装gitbook需要的npm包

在当前目录下执行命令：

```shell
gitbook install
```

1.4 设置首页链接和中文语言

编辑 `book.json` 文件，设置 `language` 和 `links`：

```json
{
  "language": "zh-hans",
  "links": {
    "sidebar": {
      "Home": "https://docs.kilvn.com/"
    }
  },
}
```

2. 生成图书

2.1 输出为静态网站

你有两种方式输出一个静态网站：

2.1.1 本地预览时自动生成

当你在自己的电脑上编辑好图书之后，你可以使用Gitbook的命令行进行本地预览：

```
$ gitbook serve .
```

然后浏览器中输入 <http://localhost:4000> 就可以预览生成的以网页形式组织的书籍。

这里你会发现，你在你的图书项目的目录中多了一个名为_book的文件目录，而这个目录中的文件，即是生成的静态网站内容。

2.2 使用build参数生成到指定目录

2.2.1 与直接预览生成的静态网站文件不一样的是，使用这个命令，你可以将内容输入到你所想要的目录中去：

```
$ mkdir /tmp/gitbook
$ gitbook build --output=/tmp/gitbook
```

