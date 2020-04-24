# 1. 基础

ZooKeeper 是一个分布式数据一致性解决方案。常用于担任服务生产和消费者的注册中心。





## ZAB 原子广播协议

### 角色 
- Leader领导者：集群中唯一能提供写服务的，能发起投票；
- Follower跟随者：接收客户端请求，读请求自己处理，写请求转发给Leader，有选举和被选举权；
- Observer观察者：没有选举权和被选举权的Follower。
 

### 模式

1. 消息广播
    - 集群中有过半的 Follower 完成了和 Leader 的状态同步，则进入消息广播模式。
    - 使用**队列**和**递增事务ID**保证顺序性。
2. 崩溃恢复
    - 当有节点GG，也能保证数据一致性。
    - Leader GG，需要进行重新选举。为保证数据一致性，需要将已被提交的提案能够被所有的Follower提交 和 跳过那些已经被丢弃的提案 


### Leader选举
- Leader election选举阶段：节点在一开始都处于选举阶段，只要有一个节点得到超半数节点的票数，就当选准 leader。
- Discovery发现阶段：followers 跟准 leader 同步 followers 最近接收的事务提议。
- Synchronization同步阶段：同步集群中所有的副本，同步完成后成为真正的 leader。
- Broadcast广播阶段：集群对外提供事务服务，并且 leader 可以进行消息广播。同时如果有新的节点加入，还需要对新节点进行同步。







## 数据模型
Zookeeper是树型结构，使用 znode 作为数据节点，用来保存数据和挂载子节点。

节点类型：
- 持久节点：一旦创建只能主动移除，否则一直保存在ZooKeeper上；
- 临时节点：生命周期和客户端会话绑定，会话失效，临时节点就移除。只能作为叶子节点。

节点状态:

维护一个 Stat 的数据结构，记录每个ZNode的版本，包括当前ZNode版本，当前ZNode子节点版本，当前ZNode的ACL版本。

## 会话
zookeeper是 tcp长连接 的会话机制。

## Watcher机制

Watcher事件监听器，类似发布订阅。
- Client向Server注册指定的Watcher；
- 当Server触发特定事件时，事件通知客户端；
- Client执行相应的回调方法。


## ACL

ACL(Access Control Lists)权限控制。定义5种权限：
- create：创建子节点；
- read：获得节点数据和子节点列表；
- write：更新节点数据；
- delete：删除子节点；
- admin：设置节点acl权限。


# 应用场景

1. 选主。高并发情况下保证节点全局唯一，利用临时节点和watch机制实现。临时节点用来选举，watch机制用来判断master的活性。
2. 分布式锁。多个Client同时创建一个临时节点，创建成功的获取锁，其他的创建watch监听节点状态，被释放就回调重新竞争锁。
3. 命名服务。树形结构的全路径全局唯一。
4. 注册中心。
    - Server创建临时节点填入ip、port、调用方式等信息；
    - Client 第一次调用通过 zookeeper 找到服务地址列表缓存本地；
    - 之后通过负载均衡算法找到地址内的服务器调用。

