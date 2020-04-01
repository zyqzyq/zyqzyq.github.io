---
author: zyqzyq
date: 2020-04-01 10:10:32+00:00
layout: post
title: python升级sqlite3
key: 20200401
tags:
- sqlite3
---

#### 转载自：

[https://www.cnblogs.com/animalize/p/6337440.html](https://www.cnblogs.com/animalize/p/6337440.html)



## 查看当前sqlite3版本

​	用sqlite3.sqlite_version看一下SQLite引擎的版本​		

```python
import sqlite3
sqlite3.sqlite_version
```



## 安装 sqlite

官网：

[https://www.sqlite.org/download.html]: https://www.sqlite.org/download.html

下载，编译，安装

```
wget https://www.sqlite.org/2019/sqlite-autoconf-3280000.tar.gz
tar zxvf sqlite-autoconf-3280000.tar.gz
cd sqlite-autoconf-3280000

./configure
make
sudo make install
```

确认sqlite是否安装成功

```
ls -l /usr/local/lib/*sqlite*
ls -l /usr/local/include/*sqlite*
```

## 安装python

```
wget https://www.python.org/ftp/python/3.7.3/Python-3.7.3.tgz
tar zxvf Python-3.7.3.tgz
cd ./Python-3.7.3
LD_RUN_PATH=/usr/local/lib ./configure LDFLAGS="-L/usr/local/lib" CPPFLAGS="-I/usr/local/include"  --prefix=/opt/python3.7
LD_RUN_PATH=/usr/local/lib make
sudo make install
```



安装完成后重新查看sqlite3版本。
