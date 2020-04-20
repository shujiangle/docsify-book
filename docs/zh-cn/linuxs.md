# linux的常用命令

## 网络故障排除命令
---

 - ping
 - traceroute
 - mtr
 - nslookup
 - telnet
 - tcpdump
 - netstat
 - ss

```bash
[root@centos-1 ~]# traceroute -w 1 www.baidu.com

[root@centos-1 ~]# telnet www.chinah3c.com 80 # 表示畅通
Trying 148.70.34.222...
Connected to www.chinah3c.com.
Escape character is '^]'.

[root@centos-1 ~]# nslookup www.chinah3c.com
Server:         114.114.114.114
Address:        114.114.114.114#53

Non-authoritative answer:
Name:   www.chinah3c.com
Address: 148.70.34.222

# -i any 所有网卡的数据包，-n 如果有域名就解析成ip
[root@centos-1 ~]# tcpdump -i any -n host 10.0.0.1 and port 80

```

## Linux中如何找出文件的最长行和最短行

```
最短行：awk '(NR==1||length(min)>length()){min=$0}END{print min}'   data.txt

最长行：awk '{if (length(max)<length()) max=$0}END{print max}'  data.txt 

打印最长长度
cat hbl-member.log | wc -l
```
