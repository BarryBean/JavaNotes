# 概述

REST 即 `Respresentational State Transfer`，**(资源)表现层状态转化**。若一个架构符合 REST 原则，则称为 RESTFul 架构。RESTful API是目前比较成熟的一套互联网应用程序的API设计理论。

# 解释

1. 资源

所谓资源，理解为网络上的一个实体，可以是文本、图片、音频、服务，可以使用一个特定的 URI(统一资源定位符) 访问。

2. 表现层

资源呈现出的形式称为表现层。比如文本可以用txt，也可以用json、html。

3. 状态转化

HTTP是无状态的协议，客户端想操作服务器，必须要让服务端发生状态转化，这个前提是建立在表现层之上。具体手段就是HTTP中的GET、POST、PUT和DELETE。


## 总结
RESTFul架构可以定义为：
1. 每一个URI代表一种资源；
2. C端和S端中间有传递资源的表现层；
3. 客户端通过HTTP的4个动词，对服务端进行操作。


# 设计
1. 协议：HTTP或HTTPS；
2. 域名：api放主域名之下；
3. 版本：将版本号放在HTTP头信息中；
4. 路径：只能有名词，返回集合的需要用复数形式；
5. 动词：HTTP的四种 + PATCH(更新资源，改变部分属性)；
6. 过滤：limit、offset、page、perpage等；
7. 状态码：HTTP协议提供的状态码；
8. 错误处理：对于4xx的状态码，封装处理返回；
9. Hypermedia API：返回结果中提供其他/相关的API方法；
10. 数据格式：能用json用json，不能就xml或yaml。







