# centos 7 源码安装 python3

标签（空格分隔）： linux

---

```
#!/bin/bash

# 安装依赖包
yum install -y zlib-devel bzip2-devel openssl-devel ncurses-devel gcc gcc-c++

yum install -y sqlite-devel readline-devel tk-devel gdbm-devel

yum install -y db4-devel libpcap-devel xz-devel wget libffi-devel


# 下载软件包
wget https://www.python.org/ftp/python/3.6.6/Python-3.6.6.tar.xz

# 解压缩软件包
xz -d Python-3.6.6.tar.xz && tar xvf Python-3.6.6.tar

# 编译安装
cd Python-3.6.6 && ./configure --prefix=/usr/local/python3 && make && make install

# 创建python3和pip3的软链接
ln -s /usr/local/python3/bin/python3 /usr/bin/python3
ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3
```





