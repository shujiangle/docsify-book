# nginx

标签（空格分隔）： linux

---

## Nginx 工作原理
Nginx WEB 服务器最主要就是各种模块的工作，模块从结构上分为核心模块、基础模块和第三方模块，其中三类模块分别如何：

 - 核心模块：HTTP模块、EVENT模块和MAIL模块
 - 基础模块：HTTP Access模块、HTTP FastCGI模块、HTTP Proxy模块和HTTP Rewrite模块：
 - 第三方模块：HTTP Upstream Reques Hash 模块、Notice模块和HTTP Access Key模块


Nginx的高并发得益于其采用了epoll 模型，异步非阻塞
apache 采用的是 select 模型

![image.png-111.1kB][1]





  [1]: http://static.zybuluo.com/sjl--3306/2gjhyp2ta3owbdv2afpwio6g/image.png
