# Redis 学习


---

## Redis企业实战
1）Redis基于C语言开发的内存数据库，数据都是存储到内存中

2）Redis基于K-V(key-values),称为Nosql数据库，MYSQL|Mariadb SQL数据库，关系型数据库
数据存储在表中，每张表由不同的行和列，每行是存储数据真实内容，每列是存储真实内容的数据段

3）Redis主要用来做什么？可以作为关系型数据库的补充，存储用户、密码、信息、工资条订单信息、
微博信息， id, Redis 最大的特点就是快！数据全部 是存储于内存，当服务宕机或者异常退出，内存
中的数据就丢了，Redis引入机制，定时将内存中的数据刷新到硬盘上，从而实现数据持久化保存，如果
Redis宕机，数据已经存储到硬盘，当Redis再次启动时，会加载硬盘中的数据到内存中去

4）CPU是中央处理器，类似人的大脑，主要负责计算和处理数据，其中数据从内存中、硬盘中获取的

5）Mongodb 属于 nosql数据库，主要用于存储图片、文件、视频、JSON文件，支持分布式、副本集，
硬盘存储
<br />
<br />
|   名称    |       数据库类型        |               数据存储选项               |      操作类型      |                   备注                   |
|:---------:|:-----------------------:|:----------------------------------------:|:------------------:|:----------------------------------------:|
|   Redis   |  内存存储，Nosql数据库  | 支持字符串、列表、集合、散列表、有序集合 |  增、删、改、更新  | 支持分布式集群、主从同步及高可用、单线程 |
| Memcached | 内存缓存数据库，键值对  |              键值之间的映射              |  增、删、改、更新  |                支持多线程                |
|   Mysql   | 典型关系型数据库，RDBMS |     数据库由多表组成，每张表包含多行     |  增、删、改、更新  |               支持ACID性质               |
|  MongoDB  |  硬盘存储,Nosql数据库   |             数据库包含多个表             | 增、删、修改、更新 |     主从复制，分片，副本集、空间索引     |
<center>表1-1  常见数据库功能对比</center>

### Redis部署
```bash
# 下载redis-2.8.13安装包
wget http://download.redis.io/releases/redis-2.8.13.tar.gz
tar zxf redis-2.8.13.tar.gz
cd redis-2.8.13
make PREFIX=/usr/local/redis install
cp redis.conf /usr/local/redis
```

#### Nohup后台启动及停止 Redis服务命令
```bash
nohup /usr/local/redis/bin/redis-server /usr/local/redis/redis.conf &
/usr/local/redis/bin/redis-cli -p 6379 shutdown
```

#### redis 相关命令
```bash
# 查看所有的键值对
127.0.0.1:6379> keys *
1) "shu1"
2) "shu3"
3) "shu4"

# 获取某个键的值
127.0.0.1:6379> get shu1
"1"

# 创建键和值
127.0.0.1:6379> set zhang san
OK

# 清空键和值
127.0.0.1:6379> flushall
OK
127.0.0.1:6379> keys *
(empty list or set)

# 删除某个键
127.0.0.1:6379> del shu1
(integer) 1

#　设置redis的密码为flzx3qc
127.0.0.1:6379> config set requirepass 'flzx3qc'
OK

# 连接redis（前提：redis已经设置了密码） 没有输入密码
127.0.0.1:6379> keys *
(error) NOAUTH Authentication required.
127.0.0.1:6379> auth flzx3qc
OK

# 连接redis
redis-cli -h host -p port -a password

# 返回当前数据库key的数量
127.0.0.1:6379> dbsize

# 查看cpu的信息
(integer) 2
```


## Redis数据库常用的备份方式
RDB 备份方式：Redis 定期将内存中数据刷到磁盘上，从而保证数据库的持久化，永久保存Redis数
也可以称为半持久化模式
save 900 1
save 300 10
save 60 10000
总结：900秒发生一次，才进行快照，减轻磁盘的I/O操作，不急着刷新数据到硬盘；另外一种情况当
Redis 大量写入，为了防止Redis宕机，将刷新时间间隔缩小，例如改成60 10000，能够保证数据损失
更小

AOF 备份方式: Redis 实时将内存中数据刷到磁盘上，从而保证数据库的持久化，永久保存Redis数
也可以称为全持久化模式
Redis 主从复制，主库有更新操作，key 全部同步到从库，从而保证对主库起到数据备份的用途
