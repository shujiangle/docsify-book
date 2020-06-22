# 基于Docker构建Jenkins CI平台

标签（空格分隔）： k8s学习

---

三台主机
系统： centos 7.5    IP:  192.168.153.101      (gitlab)
系统： centos 7.5    IP:  192.168.153.102      (jenkins)
系统： centos 7.5    IP:  192.168.153.103      (harbor)


三台主机分别安装docker
```
下载阿里云的docker yum源,并安装
# wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
# yum -y install docker-ce

启动docker,并设置docker开机自启
# systemctl start docker
# systemctl enable docker

配置docker 加速
cat > /etc/docker/daemon.json << EOF
{
  "registry-mirrors": ["https://6ze43vnb.mirror.aliyuncs.com"]
}
EOF

```

# 1. 部署gitlab   (192.168.153.101)
## 1.1 部署gitlab
```
docker run -d \
  --name gitlab \
  -p 8443:443 \
  -p 9999:80 \
  -p 9998:22 \
  -v $PWD/config:/etc/gitlab \
  -v $PWD/logs:/var/log/gitlab \
  -v $PWD/data:/var/opt/gitlab \
  -v /etc/localtime:/etc/localtime \
  lizhenliang/gitlab-ce-zh:latest
```
访问地址：http://IP:9999
![image_1ea9aukiiah1n7n1gnc11hi12or9.png-71.8kB][1]
初次会先设置管理员密码 ，然后登陆，默认管理员用户名root，密码就是刚设置的。
密码为  password123qwer
<br />
![image_1ea9b2foe19satttqe164b1bkcm.png-82.9kB][2]

### 1.1.1 新建一个java-demo项目
![image_1ea9eih4m3537do64ajrpglu1g.png-95.1kB][3]

## 1.2 创建项目，提交测试代码 (192.168.153.103执行操作)
把代码上传到192.168.153.103主机
```
# 项目改名为java-demo
[root@centos-3 ~]# mv tomcat-java-demo-master java-demo

# 提交代码到gitlab上
[root@centos-3 ~]# git init java-demo/
[root@centos-3 ~]# cd java-demo 
[root@centos-3 ~]# git add *
[root@centos-3 java-demo]# git config --global user.name "shujiangle"  
[root@centos-3 java-demo]# git config --global user.email "shujiangle@qq.com"
[root@centos-3 java-demo]# git commit -m "初始化提交"
[root@centos-3 java-demo]# git remote add origin http://192.168.153.101:9999/root/java-demo.git
[root@centos-3 java-demo]# git push -u origin master
```
![image_1ea9ea6401iqibfa1rjrrf9cdq13.png-115.4kB][4]


# 2.部署harbor (192.168.153.103)
## 2.1  Harbor安装
1) 安装 docker 与 docker-compose
上传  二进制包 docker-compose-Linux-x86_64 和 harbor 压缩包
```
[root@centos-3 ~]# mv docker-compose-Linux-x86_64 /usr/bin/docker-compose
[root@centos-3 ~]# chmod +x /usr/bin/docker-compose
[root@centos-3 ~]# cd harbor/
[root@centos-3 harbor]# vim harbor.yml
hostname: 192.168.153.103
[root@centos-3 harbor]# ./prepare
[root@centos-3 harbor]# ./install.sh
```
访问 http://192.168.153.103/
用户名 admin 密码 Harbor12345
![image_1eabgcdbq1r791in616je1rc3k9k9.png-112.4kB][5]


## 2.2 上传tomcat镜像到harbor
```
# Jenkins主机配置Docker可信任
[root@centos-3 tomcat]# cat /etc/docker/daemon.json 
{
  "registry-mirrors": ["https://6ze43vnb.mirror.aliyuncs.com"],
  "insecure-registries": ["192.168.153.103"]
}

[root@centos-3 harbor]# systemctl restart docker
[root@centos-3 harbor]# docker-compose up -d

# 进入tomcat目录
[root@centos-3 tomcat]# docker build -t tomcat:v1 .
[root@centos-3 tomcat]# docker tag tomcat:v1 192.168.153.103/library/tomcat:v1
[root@centos-3 tomcat]# docker push 192.168.153.103/library/tomcat:v1
```

效果如图
![image_1eabj0h1m16o1b3ha9j1ch3rmcm.png-111.2kB][6]




# 3. 部署jenkins (192.168.153.102)
## 3.1 准备JDK和Maven环境
```
[root@centos-2 ~]# tar -zxvf jdk-8u45-linux-x64.tar.gz
[root@centos-2 ~]# tar -zxvf apache-maven-3.5.0-bin.tar.gz
[root@centos-2 ~]# mv apache-maven-3.5.0 /usr/local/apache-maven-3.5.0
[root@centos-2 ~]# mv jdk1.8.0_45 /usr/local/jdk1.8.0_45
```
## 3.2 安装 
```
docker run -d --name jenkins -p 80:8080 -p 50000:50000 -u root \
-v /opt/jenkins_home:/var/jenkins_home \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /usr/bin/docker:/usr/bin/docker \
-v /usr/local/apache-maven-3.5.0:/usr/local/maven \
-v /usr/local/jdk1.8.0_45:/usr/local/jdk \
-v /etc/localtime:/etc/localtime \
--name jenkins jenkins/jenkins:lts
```
浏览器输入 192.168.153.102 访问  
![image_1eabjj4sh1sl4192odnv12691iru13.png-310.1kB][7]
  
```
# 查看管理员密码
[root@centos-2 local]# docker logs jenkins
```
![image_1eabjpoaq1u7d6eb1mtmcqc3oq1g.png-47.4kB][8]
  
![image_1eabk03i11jnoc6m1e747elgj41t.png-348.1kB][9]
  
![image_1eabk1vr81o9133kju71uqs11f82a.png-423.5kB][10]
  
![image_1eabk4h701ejed4qbv41ujg13tp2n.png-275.2kB][11]
  
![image_1eabk6a7c5e31kg35a610er1cvj34.png-293.5kB][12]
  
## 3.3 jenkins 插件安装加速和 maven 构建加速
```
[root@centos-2 ~]# cd /opt/jenkins_home/updates
[root@centos-2 updates]# sed -i 's/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' default.json && sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' default.json

vim /usr/local/maven/conf/settings.xml
<mirror>
    <id>aliyunmaven</id>
    <mirrorOf>*</mirrorOf>
    <name>阿里云公共仓库</name>
    <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```
## 3.4 安装jenkins 插件 git 和pipline
![image_1eabkv8r4128uhnot28741mfl4b.png-203.7kB][13]
<br />
![image_1eabl0frp1fjib3ent9n6n1i5i4o.png-98.7kB][14]
<br />
![image_1eabl1iu57mi1h3uc8b149mv8v55.png-155.8kB][15]
<br />
![image_1eabl32c38d8ucu1hcjvocmvh5i.png-153.1kB][16]
<br />
![image_1eabl3vhmec6pv2vnea61knv5v.png-125.3kB][17]
<br />
![image_1eabl7n9mokhea1uocdn8aca6c.png-145.9kB][18]
<br />
![image_1eabla0eh1jvl1ednli6q9bk536p.png-130.6kB][19]
<br />
## 3.5 新建项目
![image_1eabljkda1acd1bec1qlrbha14kv76.png-153.2kB][20]
<br />
![image_1eabln9c56se1foh3ag1j7g1u9m7j.png-248.5kB][21]
<br />
![image_1eabloigh5h1tu110ccdig183i80.png-91.5kB][22]
![image_1eablp8bucgp17guu8pk5c1kpo8d.png-468.1kB][23]
welcome 项目提前在 Harbor 仓库创建好 ↑ 。
![image_1eabmg4ga1d1obah1k321a817e2d9.png-91.6kB][24]
修改 docker_registry_auth、git_auth 凭据：
![image_1eablpv8ijhm15071o7j12cg1h8q.png-159kB][25]
![image_1eablq70b1mmu6me55r4dqvsv97.png-213.9kB][26]
![image_1eablqgm61d361fn01f2e1e3lt0j9k.png-88.5kB][27]
![image_1eablr7ef1g0nmj7vc9vts1ctua1.png-122.3kB][28]
![image_1eablrm60163v1hm21ott1iiof3ae.png-131.3kB][29]

添加拉取 git 代码的凭据：
![image_1eablt1pung11cuklnkhiht5par.png-179.1kB][30]
![image_1eabltavr1nm1n6o0teq2176cb8.png-175.9kB][31]
![image_1eabltj3ear8k231mlrd3d1lrgbl.png-222.9kB][32]
![image_1eabltvv71tuh4me1b5h1pd71jdic2.png-241.8kB][33]
![image_1eablu5ml12mn1ei31hr91nm9bd8cf.png-148.3kB][34]
![image_1eablucn4s5s1m8lqjn116r5b4cs.png-168kB][35]
![image_1eabnks1aj5e1tfe6uas3llvbdm.png-125.4kB][36]

最后效果
![image_1eabntce41g3b1tltbkiu8clvoe3.png-2619.2kB][37]


  [1]: http://static.zybuluo.com/sjl--3306/z8doo0lz4n6yjohp1nvvwgpi/image_1ea9aukiiah1n7n1gnc11hi12or9.png
  [2]: http://static.zybuluo.com/sjl--3306/2rcw9oekypvans66vqslynya/image_1ea9b2foe19satttqe164b1bkcm.png
  [3]: http://static.zybuluo.com/sjl--3306/o23b62w23jecfvtzcyom3zbn/image_1ea9eih4m3537do64ajrpglu1g.png
  [4]: http://static.zybuluo.com/sjl--3306/s1dbmuboscvl6msb5b72mamr/image_1ea9ea6401iqibfa1rjrrf9cdq13.png
  [5]: http://static.zybuluo.com/sjl--3306/p7705qmhxuupimq875wnn8nm/image_1eabgcdbq1r791in616je1rc3k9k9.png
  [6]: http://static.zybuluo.com/sjl--3306/audu78hrhkk47q3qlni3lrsw/image_1eabj0h1m16o1b3ha9j1ch3rmcm.png
  [7]: http://static.zybuluo.com/sjl--3306/60ej6m9qpoucu6nng1lax4cp/image_1eabjj4sh1sl4192odnv12691iru13.png
  [8]: http://static.zybuluo.com/sjl--3306/y4dt0sgfiwpzavxrl1lph6e5/image_1eabjpoaq1u7d6eb1mtmcqc3oq1g.png
  [9]: http://static.zybuluo.com/sjl--3306/nrj1hay4328dkszdg06ropvz/image_1eabk03i11jnoc6m1e747elgj41t.png
  [10]: http://static.zybuluo.com/sjl--3306/l9q2t2pwxpfe1tl3to2dp34s/image_1eabk1vr81o9133kju71uqs11f82a.png
  [11]: http://static.zybuluo.com/sjl--3306/n6dnx6woiyp80covxhrpc2zu/image_1eabk4h701ejed4qbv41ujg13tp2n.png
  [12]: http://static.zybuluo.com/sjl--3306/js6eqcgah3babbjsmhc4j07f/image_1eabk6a7c5e31kg35a610er1cvj34.png
  [13]: http://static.zybuluo.com/sjl--3306/90kccnqlmad89zr3dmvjiklv/image_1eabkv8r4128uhnot28741mfl4b.png
  [14]: http://static.zybuluo.com/sjl--3306/wi0opt5b3d1fh4mygs6ab4dw/image_1eabl0frp1fjib3ent9n6n1i5i4o.png
  [15]: http://static.zybuluo.com/sjl--3306/nuffsy5nfcz6mjpag0pe5snl/image_1eabl1iu57mi1h3uc8b149mv8v55.png
  [16]: http://static.zybuluo.com/sjl--3306/medq69f9uzjgwszxzes0b7ff/image_1eabl32c38d8ucu1hcjvocmvh5i.png
  [17]: http://static.zybuluo.com/sjl--3306/bgwv9nsjt88uuxo958qcs2mv/image_1eabl3vhmec6pv2vnea61knv5v.png
  [18]: http://static.zybuluo.com/sjl--3306/sllq75jn0etpfoa562cn2g1f/image_1eabl7n9mokhea1uocdn8aca6c.png
  [19]: http://static.zybuluo.com/sjl--3306/bfiyvk1b4j88enwz3gx2xpm4/image_1eabla0eh1jvl1ednli6q9bk536p.png
  [20]: http://static.zybuluo.com/sjl--3306/nkg7u5rnh97zmz9v4tovoq1r/image_1eabljkda1acd1bec1qlrbha14kv76.png
  [21]: http://static.zybuluo.com/sjl--3306/b5iqf5nrimt4ho9hfd7l1iow/image_1eabln9c56se1foh3ag1j7g1u9m7j.png
  [22]: http://static.zybuluo.com/sjl--3306/ug5v79tuof2p63frlpda9grp/image_1eabloigh5h1tu110ccdig183i80.png
  [23]: http://static.zybuluo.com/sjl--3306/1pwz28gdwuu3mu7x6ed1dmlm/image_1eablp8bucgp17guu8pk5c1kpo8d.png
  [24]: http://static.zybuluo.com/sjl--3306/vuij2xydjzj4swtytq8l2ckv/image_1eabmg4ga1d1obah1k321a817e2d9.png
  [25]: http://static.zybuluo.com/sjl--3306/w2ensz039objn8juuusuqxoo/image_1eablpv8ijhm15071o7j12cg1h8q.png
  [26]: http://static.zybuluo.com/sjl--3306/d1bhsy4pd54r0qwf12um3qxx/image_1eablq70b1mmu6me55r4dqvsv97.png
  [27]: http://static.zybuluo.com/sjl--3306/wdo13n5yzd1dj0nfcalv9v1k/image_1eablqgm61d361fn01f2e1e3lt0j9k.png
  [28]: http://static.zybuluo.com/sjl--3306/65ng00us80rmmq3e1wl5uiub/image_1eablr7ef1g0nmj7vc9vts1ctua1.png
  [29]: http://static.zybuluo.com/sjl--3306/ieqcsm4vup36ooaxjr44qv80/image_1eablrm60163v1hm21ott1iiof3ae.png
  [30]: http://static.zybuluo.com/sjl--3306/qraovo04wcrh9m8vc2vwhnda/image_1eablt1pung11cuklnkhiht5par.png
  [31]: http://static.zybuluo.com/sjl--3306/e6vvu9imtpvh3njckous60zs/image_1eabltavr1nm1n6o0teq2176cb8.png
  [32]: http://static.zybuluo.com/sjl--3306/fxxf7cuqnhpsurzftdyrta48/image_1eabltj3ear8k231mlrd3d1lrgbl.png
  [33]: http://static.zybuluo.com/sjl--3306/f5m9440bjx6a55jjrfivtia7/image_1eabltvv71tuh4me1b5h1pd71jdic2.png
  [34]: http://static.zybuluo.com/sjl--3306/f3rcp4bfkb2hyks6soj2i1aj/image_1eablu5ml12mn1ei31hr91nm9bd8cf.png
  [35]: http://static.zybuluo.com/sjl--3306/wu01l7o2f8fwz0io9wggx10d/image_1eablucn4s5s1m8lqjn116r5b4cs.png
  [36]: http://static.zybuluo.com/sjl--3306/h7lfts4kk3hyaj5iu880et4m/image_1eabnks1aj5e1tfe6uas3llvbdm.png
  [37]: http://static.zybuluo.com/sjl--3306/vyrz9h1a5a2g1ynze34ngxdi/image_1eabntce41g3b1tltbkiu8clvoe3.png