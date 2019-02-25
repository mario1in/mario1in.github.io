---
title: 由于Python版本太低遇到的环境问题备忘
date: 2019-02-25 15:57:53
tags: 
- Python
- Flask
- OpenSSL
- Linux
---

这两天在倒腾用Python写个接口，遇到一些问题，特此做下记录备忘，方便后面查阅。

## Error - A true SSLContext object is not available

> InsecurePlatformWarning: A true SSLContext object is not available. This prevents urllib3 from configuring SSL
appropriately and may cause certain SSL connections to fail. For more information, see https://urllib3.readthedocs.org/en/latest/security.html#insecureplatformwarning.

这是因为我服务器上的Python版本太低了，如果Python版本低于2.7.9之前的话，就会有这个错误提示，因为2.7.9之前的Python提供的SSL环境不是很安全。

所以理所当然我们就要升级Python的版本了。

## Install Python3.7

`cat /etc/redhat-release`

看了一眼系统是 `CentOS 6`

于是先使用yum安装环境依赖。

```python
# wget 用于下载源码包
# gcc 和 make 用于编译
yum install wget gcc make

#make报错，Python 有个很重要的内建模块 zipimport 用于从 Zip 压缩包中导入模块
#zipimport.ZipImportError: can't decompress data; zlib not available
yum install zlib-devel

#make install报错，
#ModuleNotFoundError: No module named ‘_ctypes’
yum install libffi-devel

# 解决 import ssl 报错 No module named '_ssl'
yum install openssl-devel

# 解决 import bz2 报错
yum install  bzip2-devel

# 解决 import curses 报错
yum install  ncurses-devel

# 解决 import sqlite3 报错
yum install sqlite-devel

# 解决 _dbm _gdbm 缺失提醒
yum install gdbm-devel

# 解决 _lzma 缺失提醒
yum install xz-devel

# 解决 _tkinter 缺失提醒
yum install tk-devel

# 解决 readline 缺失提醒及方向键行为非预期的问题
yum install readline-devel
```

然后下载源码包

`wget https://www.python.org/ftp/python/3.7.1/Python-3.7.1.tar.xz`

解压缩

```
xz -d Python-3.7.1.tar.xz
tar -xvf Python-3.7.1.tar
```

编译&安装

```
cd Python-3.7.1
#--prefix 是预期安装目录，--enable-optimizations 是优化选项（LTO，PGO 等）
./configure --prefix=/usr/local/python3.7 --enable-optimizations
# 安装
make && make install
```

添加软链接

```
ln -s /usr/local/python3.7/bin/python3.7 /usr/bin/python3
ln -s /usr/local/python3.7/bin/pip3.7 /usr/bin/pip3
```

这样子应该一切就搞定啦。
然而，事情好像没有那么简单。

当我尝试使用 `pip3 install flask` 的时候，报错。

## Error - OpenSSL version

> pip is configured with locations that require TLS/SSL, however the ssl module in Python is not available.

前边说过，我的服务器系统是CentOS 6，默认的OpenSSL版本是1.0.1，而Python3.7需要OpenSSL1.0.2或者1.1.x才行，于是我们需要对OpenSSL进行升级并且重新编辑Pyton3.7。

## 升级OpenSSL

下载最新版的OpenSSL

`wget https://www.openssl.org/source/openssl-1.1.1-pre8.tar.gz`

编译&安装

```
cd openssl-1.1.1-pre8
./config --prefix=/usr/local/openssl no-zlib #不需要zlib
make & make install
```

备份原配置

```
mv /usr/bin/openssl /usr/bin/openssl.bak
mv /usr/include/openssl/ /usr/include/openssl.bak
```

添加软链接

```
ln -s /usr/local/openssl/include/openssl /usr/include/openssl
ln -s /usr/local/openssl/lib/libssl.so.1.1 /usr/local/lib64/libssl.so
ln -s /usr/local/openssl/bin/openssl /usr/bin/openssl
```

修改系统配置

```
#写入openssl库文件的搜索路径
echo "/usr/local/openssl/lib" >> /etc/ld.so.conf
#使修改后的/etc/ld.so.conf生效 
ldconfig -v
```

重新编译Python3.7

```
cd Python-3.7.1
#--prefix 是预期安装目录，--enable-optimizations 是优化选项（LTO，PGO 等）
./configure --prefix=/usr/local/python3.7 --enable-optimizations  --with-openssl=/usr/local/openssl
# 安装
make && make install
```


**至此，所有问题都解决了。**
**感想：Linux里的相关环境配置，真是一门大学问啊**

## 参考
*[Flask安装: A true SSLContext object is not available](https://blog.tanteng.me/2015/12/flask-sslcontext/)*
*[CentOS 7 下安装 Python3.7.1](https://segmentfault.com/a/1190000017313144)*
*[python3.7安装后ssl问题](https://blog.51cto.com/13544424/2149473)*


