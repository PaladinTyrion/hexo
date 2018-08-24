---
title: Paxos_and_Zookeeper
date: 2017-09-10 18:20:23
categories: 一致性
type: "tags"
tags: 
  - 一致性
---

#### （一）散点：

- 三态：成功、失败、超时。
- ACID/CAP/BASE
- Lamppost论文：
    - The Part-Time Parliament
    - Time, Clocks, and the Ordering of Events in a Distributed System

#### （二）Chubby：

- 编号：实例编号、文件内容编号、锁编号、ACL编号。
- Chubby的Paxos实现：
    - 最底层容错日志系统
    - 日志层之上是Key-Value类型的容错数据库
    - 存储层之上是Chubby对外提供的分布式LockService和小文件存储服务
- 优化：
    - Prepare—>Promise—>Propose—>Accept过程中选举得到Master后转态为Propose—>Accept。失去Master再转换为PPPA。

<!-- more -->

#### （三）Zookeeper基础：

- 版本：
    - version：当前ZNode的版本
    - version：当前ZNode的子节点版本
    - aversion：当前ZNode的ACL版本
- ACL权限：见(五)-2-3.Permission权限
- ZAB(Zookeeper Atomic Broadcast) 原子广播协议
    - ZAB协议选举Leader
    - Leader持续工作：决定事务PC结果，Follower与Learner同步结果
- ZXID：事务编号，共64位的数字
    - 高32位代表leader的周期epoch
    - 低32位为一计数累加数

#### （四）Zookeeper使用：

- %ZK_HOME%/conf下由zoo.cfg配置文件生效配置
    - dataDir（&& dataLogDir）
    - clientPort
    - server.x=IPn:Port1:Port2   (Port1为消息交换端口，Port2为选举端口)
- dataDir目录下myid与server.id中的id一致
- 两款开源客户端：
    - ZkClient
    - Curator（Fluent 风格）
- 分布式锁的改进
    - 读请求：向比自己序号小的最后一个写请求节点注册Watcher监听
    - 写请求：向比自己小的最后一个阶段注册Watcher监听
    - 细粒度锁提高可用性，粗粒度锁降低系统复杂性，需要折中。当然细粒度是好事
- 常见应用：发布/订阅、负载均衡、日志收集器、分布式锁
- Kafka：大规模分布式消息中间件
    - 消息生产者
    - 消息消费者
    - Topic：建立订阅关系，同一Topic会进行消息分区并分布在多个Broker上
    - 消息分区：用于负载均衡
    - Broker：存储消息
    - Group：消费者分组
    - OffSet：消费者消费数据在文件中的偏移量
- SEDA(Staged Event-Driven Architecture)

#### （五）Zookeeper细节

- Watcher：

    - Watcher对象存储在客户端WatchManager中
    - Watcher接口需实现process函数，入参为WatchedEvent对象
    - WatchedEvent：有三个属性

        - 通知状态keeperState
        - 事件类型eventType
        - 节点路径path
    - WatcherEvent：WatchedEvent三属性+序列化接口
- ACL控制：
    - Scheme权限模式：
        - IP
        - Digest
        - World
        - Super
    - ID授权对象：如username/ip:password
    - Permission权限：
        - CREATE
        - DELETE
        - READ：获取节点数据与子节点列表的权限
        - WRITE
        - ADMIN：设置节点ACL权限
- 序列化：
    - Jute：实现Record接口，方法有serialize和deserialize
    - Avro
- 通信协议：自制协议
- 客户端
    - 核心组件：
        - Zookeeper实例
        - ClientWatchManager
        - HostProvider
        - ClientCnxn：客户端核心线程，内部有两个线程
            - SendThread：I/O线程，负责与服务器的通信
            - EventThread：事件线程，负责对服务端事件处理
    - 会话创建过程：
        - 初始化Zookeeper对象，创建ClientWatchManager
        - 设置默认Watcher
        - 构造方法中传入服务器地址列表，构造HostProvider
        - 创建ClientCnxn：负责与服务端的网络交互，底层I/O处理器是ClientCnxnSocket，客户端初始化两个核心队列
            - outgoingQueue：客户端请求发送队列
            - pendingQueue：服务端响应等待队列
        - 创建两个核心线程：
            - SendThread：ClientCnxnSocket——>SendThread
            - EventThread：待处理事件队列waitingEvents
        - 启动SendThread与EventThread
        - 在HostProvider获取一个服务器地址，由ClientCnxnSocket创建TCP长连接
        - 构造ConnectRequest请求，包装为Packet对象，放入outgoingQueue发送至服务端
        - SendThread线程中ClientCnxnSocket接收服务端响应
            - 会话创建请求的响应，交由readConnectResult处理
            - ClientCnxnSocket反序列化响应，得到ConnectResponse，并得到sessionId
        - 连接成功，更新SendThread线程会话参数设置，readTimeout、connectTimeout等，更新客户端状态
        - SendThread生成SyncConnected-None事件，通知EventThread
        - 查询ClientWatchManager中的Watcher，先将事件放入waitingEvents队列，然后调用Watcher的process进行处理
    - ClientCnxn：
        - Packet：属性
            - requestHeader、replyHeader、request、response
            - 节点路径
            - watchRegistration
        - 网络传输时，只有少量属性被序列化，参见Packet的createBB()等方法
        - 请求发送：在outgoingQueue中取出可发送Packet，写入XID至Packet请求头，序列化后发送。发送后会将Packet保存至pendingQueue等待响应
        - 响应接收：三种情况
            - 会话创建时，ByteBuffer序列化为ConnectResponse对象
            - 正常会话周期时，收到一个事件，ByteBuffer序列化为WatcherEvent对象，放入waitingEvents队列
            - 正常请求响应，pendingQueue中取出一个Packet，ByteBuffer序列化为Response对象
        - SendThread维护客户端与服务端的会话生命周期，想服务端发送PING包实现心跳检测
    - 会话
        - 会话状态：CONNECTING、CONNECTED、RECONNECTING、RECONNECTED、CLOSE
        - session属性：sessionID、TimeOut、TickTime、isClosing
        - sessionID生成：
            - 当前时间的毫秒表示
            - 左移24位
            - 右移8位
            - 添加机器标识SID，SID即myid中的内容
                - 64位表示SID
                - 左移56位，即把最低8位移至64位最高8位
                - 将sessionID第三步与上步得到64位SID进行位或操作
        - sessionTracker（session管理）：存在三份session存储
            - sessionById：HashMap<Long, SessionImpl>，由sessionID进行管理
            - sessionWithTimeout：ConcurrentHashMap<Long, Integer>，由sessionID管理会话超时。定期持久化至快照
            - sessionSets：HashMap<Long, SessionSet>，由下次会话超时时间归档会话，分桶策略进行管理
        - 会话清理：
            - isClonsing标记为true
            - 发起关闭会话请求，交给PreRequestProcessor
            - 收集需要清理的相关临时节点
            - 添加节点删除事务变更，放入事务变更队列outstandingChanges
            - 删除临时节点
            - 删除会话，在sessionTracker中移除会话
            - 关闭NIOServerCnxn
- 集群启动
    - QuorumPeerMain启动
    - 解析zoo.cfg
    - 创建并启动历史文件清理器DatadirCleanupManager
    - 判断模式选择集群模式启动Zookeeper
    - 创建并初始化ServerCnxnFactory
    - 创建Zookeeper数据管理器FileTxnSnapLog
    - 创建QuorumPeer实例：QuorumPeer会根据检测服务器实例运行状态来发起Leader选举
    - 创建内存数据库ZKDatabase
    - 初始化QuorumPeer，将FileTxnSnapLog、ServerCnxnFactory、ZKDatabase等核心组件注册到QuorumPeer。配置QuorumPeer参数，包括服务器地址列表、选举算法（现在只剩下FastLeaderElection算法）、会话超时时间等。
        - 每机初始化一个Vote，由SID、lastLoggedZxid、epoch等生成。
        - 注册JMX服务
        - 检测服务器状态，QuorumPeer是Zookeeper服务器实例的托管者，检测LOOKING、LEADING、FOLLOWING/OBSERVING状态
        - Leader选举
            - Zxid最大的一般做Leader，若Zxid都一致，一般选SID最大者
            - FastLeaderElection算法相比Paxos超级简单。不论故障恢复还是第一次投票均适用。先比Zxid，自身的Zxid小则更变投票重投，否则再比SID，SID小则重投，再否则就维持投票
    - 恢复本地数据
    - 启动ServerCnxnFactory主线程
- 选举细节：
    - I/O：
        - ClientCnxn是ZK客户端负责与服务端的网络I/O
        - QuorumCnxManager负责选举时的网络I/O
    - QuorumCnxManager维护一系列队列：
        - recvQueue：消息接收队列，1个（后续统计Vote）：WorkerReceiver 1个，从QuorumCnxManager中取出其他服务器发来的消息，放入recvQueue（WorkerReceiver处理Vote PK过程）
        - queueSendMap：消息发送队列，数据结构为Map，由SID分组（即每台其他服务器对应一个单独队列）
        - senderWorkerMap：发送器集合，由SID分组，每个发送器对应一台单独的服务器（与queueSendMap相当于Key-Key的对应关系）
        - lastMesageSent：为每个SID保留最近发送过的消息
    - QuorumCnxManager创建ServerSocket监听选举端口。TCP连接规则是只允许SID大的服务器连接SID小的
    - FastLeaderElection算法
        - 服务器状态变为LOOKING时，选举开始
        - 自增选举轮次：logicalclock+1
        - 初始化自票、发送自票、接收外部投票
        - PK Vote：
            - 自票选举轮次小于外票，logicalclock+1，清空收集的投票，重新发起自票
            - 忽略logicalclock小的外票
            - 选举轮次logicalclock一致，Zxid PK，决定是否更变自票票重新投票
            - logical clock、Zxid均一致，SID PK，决定是否更变自票票重新投票
        - 选票归档、选票统计
        - 选票过半后，更新服务器状态
- Leader请求处理责任链：
    - PreRequestProcessor：预处理，如创建请求事务头、事务体、会话检查、ACL检查、版本检查。
    - ProposalRequestProcessor：投票处理器。对于非事务请求，直接交给CommitProcessor。对于事务请求则创建Proposal提议，发起事务投票。ProposalRequestProcessor还将事务请求交给SyncRequestProcessor进行事务日志记录。
    - SyncRequestProcessor：事务日志记录。也负责触发ZK数据快照。
    - AckRequestProcessor：事务日志记录完成后，向Proposal投票收集器发送ACK反馈通知。
    - CommitProcessor：非事务请求直接交给ToBeAppliedRequestProcessor。事务请求待收集投票达到可被提交，再交给ToBeAppliedRequestProcessor。
    - ToBeAppliedRequestProcessor：将CommitProcessor放入到toBeApplied队列的可提交Proposal逐个交给FinalRequestProcessor，处理完成后从toBeApplied中移除。
    - FinalRequestProcessor：为客户端请求返回响应。对事务，还负责将事务应用到内存数据库中。
    - ——其中，LearnerHandler：Leader服务器为每一个Follower/Observer服务器创建LearnerHandler实例，负责网络通信、数据同步、请求转发、Proposal提议的投票
- Follower请求处理链：
    - FollowerRequestProcessor：识别事务请求，转发给Leader。
    - SendAckRequestProcessor：承担事务日志记录反馈，向Leader服务器发送ACK消息。
    - SyncRequestProcessor、CommitProcessor、FinalRequestProcessor与Leader上的相同。
- Observer请求处理链：除了不参与投票，其他均与Follower相同。
- 事务处理均由ProposalRequestProcessor发起，都需要经过Sync、Proposal与Commit三个子流程协作完成。

#### （六）Zookeeper数据与存储

- DataTree：Zookeeper整棵树的数据结构，完整数据
- DataNode：数据存储的最小单元。包括了数据内容、ACL列表、节点状态、父节点的引用、子节点列表。
- nodes：DataTree内部存储实际节点的ConcurrentHashMap键值对结构。定义：private final ConcurrentHashMap<String, DataNode> nodes. 另外，临时节点为便于实施访问和清理，有单独的定义：private final Map<Long, HashSet<String>> ephemerals.  其中，nodes中key为path；ephemerals中Key为sessionID
- ZKDatabase：内存数据库，负责管理Zookeeper的所有会话，DataTree存储和事务日志。ZKDatabase定时向磁盘dump快照数据。

#### （七）日志
FileTxnLog负责维护事务日志，主要方法append(TxnHeader hdr, Record txn)

- 日志写入
    - 确定是否有日志可写：先检查FlieTxnLog是否已经关联上一个可写的事务日志，若没有则建立一个事务日志，由Zxid作后缀，并构建事务日志文件头信息（magic、version、dbid），写入日志文件。将该日志的文件流交给streamsToFlush。
    - 确定日志文件是否需要扩容：预分配64MB，当日志文件剩余空间不足4K，则扩容64MB。
    - 事务序列化。
    - 生成Checksum。
    - 写入事务日志文件流：将事务头、事务体、Checksum值写入文件流（BufferedOutputStream）中。
    - streamsToFlush提取文件流将数据输入磁盘。
- 日志截断：非Leader机器上事务ID比Leader服务器大，则由Leader发送TRUNC命令进行日志截断。

#### （八）SnapShot

- 格式：与日志一样，后缀为首条记录的Zxid
- 快照存储触发：snapCount次事务日志后，进行内存数据库的全量数据Dump至本地文件
- 过程流程：
    - 确实是否需要进行快照：logCount > （snapCount/2 + randRoll），过半触发策略
    - 切换事务日志文件
    - 创建快照异步线程
    - 获取全量数据与会话信息，生成快照文件名
    - 数据序列化（同日志一样，先文件头信息、再会话信息和DataTree，最后Checksum），写入快照文件

#### （九）初始化

- FileTxnSnopLog：FileTxnLog、FileSnap
- ZKDatabase：初始化DataTree，把FileTxnSnopLog交给ZKDatabase，创建sessionWithTimeouts
- 创建PlayBackListener监听器：接收事务应用过程中的回调
- 从快照文件恢复数据：获取最新的100（或不足100个）snapfile，从最新的file解析并校验，如校验通知则用此文件恢复
- 使用事务日志+快照恢复数据
- 获取最新的Zxid，校验epoch
