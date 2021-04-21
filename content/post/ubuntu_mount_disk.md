---
title: "ubuntu挂载硬盘"
slug: "ubuntu_mount_disk"
description: "其中第一列为UUID, 第二列为挂载目录（该目录必须为空目录），第三列为文件系统类型，第四列为参数，第五列0表示不备份，最后一列必须为２或0(除非引导分区为1)"
date: "2021-03-02T02:58:08Z"
thumbnail: ""
categories:
  - "编程开发"
tags:
  - "ubuntu"
---

**以下命令大部分都需要root权限执行**

如果当前登录的是非root用户，切换到root

```shell
sudo -i
```

#### 查看磁盘信息

```shell
fdisk -l
```

#### 查看分区的UUID

```shell
blkid
```

#### 1. 查看磁盘挂载点

```shell
df -kh
```

#### 2. 卸载原磁盘

```shell
umount /dev/sdb
```

**再次查看磁盘挂载点，可以看到该磁盘已经没有挂载了。**

#### 3. 永久性挂载分区

**先找到/dev/sdb分区对应的UUID**

```shell
blkid /dev/sdb
```

得到：`/dev/sdb: UUID="0001D3CE0001E53B" TYPE="ext4"`

**修改分区文件**

```shell
nano /etc/fstab
```

我们按照文件中的格式添加一行如下内容：

```shell
UUID=0001D3CE0001E53B /data ext4 defaults 0 0
```

**注意：如果是云服务器，不用获取UUID，直接写磁盘卷路径**

```shell
/dev/sdb /data ext4 defaults 0 0
```

第一列为UUID；

第二列为挂载目录（该目录必须为空目录）；

第三列为文件系统类型；

第四列为参数；

第五列0表示不备份；

最后一列必须为0或2(除非引导分区为1)

#### 4. 挂载

```shell
mount -a
```

查看是否成功:

```shell
df -kh
```

可以看见`/dev/sdb`已经成功挂载到了`/data`目录下了，以后开机就会自动挂载。


