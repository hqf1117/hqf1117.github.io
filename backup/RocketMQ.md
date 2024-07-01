## 基本概念

- message

- topic 

- tag

- queue

- 消息表示 messageId/Key 

  每个消息拥有唯一的messageId，且可以携带具有业务表示的Key，以方便对消息的查询。

  生产者send时会生成msgId，到达broker也会生成一个offsetMsgId。

  - msgId：$\color{#FF0000}{producerId + 进程pid + MessageClientIDSetter类的ClassLoader的hashcode + 当前时间+AutomicInteger自增计数器}$
  - offsetMsgId：$\color{#FF0000}{brokerIP + 物理分区的offset}$
  - key: 由用户指定的业务相关的唯一标识

## 角色架构

![image-20220602142400414](https://github.com/hqf1117/hqf1117.github.io/assets/30412361/ae2c59dc-3f23-4436-9432-b9fad00609e0)

### NameServer、Broker与Topic路由的注册中心

broker节点以长连接的方式与每一个nameServer节点建立通信，每30秒定时发送心跳包。$\color{#FF0000}{心跳包内容包含当前broker信息(IP+端口等)以及存储所有topic信息}$

nameserver中有一个定时任务每10秒扫描一次broker心跳数据，心跳最新时间戳超过120秒则判定broker失效，将broker列表中剔除。 

客户端先采用==随机==的策略选择nameserver，连接失败再采用==轮询==的方式，只需跟一台broker建立长连接即可。

<img width="848" alt="image-20220922104925782" src="https://github.com/hqf1117/hqf1117.github.io/assets/30412361/1d88bfd5-4b24-4e44-96e8-1ccd6b9e39d8">

- remoting module:整个broker的实体，负责处理来自clients端的请求。而这个broker实体由以下模块构成
  - Client manager：客户端管理器，负责接收解析producer和consumer的请求，管理客户端。如维护订阅信息等。
  - Store service：存储服务。提供方便简单的api接口，处理==消息持久化==和==消息查询==功能。
  - Ha service：高可用服务。提供主从broker之间的数据同步功能。
  - Index service：索引服务。根据特定的message key，对投递的broker的消息进行索引服务，以及快速查询。

producer发送消息的流程：启动时先跟nameserver集群中的一台建立长连接，来获取broker的路由信息缓存在本地，每30秒更新一次路由信息。然后根据算法策略选择一个queue，与queue所在broker建立长连接，来发送message。

consumer消费流程类似，额外多一个心跳连接确保broker存活。

## ACL权限

plain_acl.yml文件

<img width="381" alt="image-20221028235246156" src="https://github.com/hqf1117/hqf1117.github.io/assets/30412361/5e0b5fcc-0d41-4d79-9636-937758eb1a91">

## 控制台工具rocketmq-externals

## 消息消费模式

实际上都是pull的形式，但是利用长轮询来达到push模式的效果。

![image-20220602151048714](https://github.com/hqf1117/hqf1117.github.io/assets/30412361/5656a435-d91f-4e4d-ae0a-517159f2ab86)

## 消息类型

### 顺序消息

可以按照FIFO的模式来保证消息有序。可以是全局有序或分区有序。全局有序只是把消息发送消费全部依靠同一个broker而已。 

### 延迟消息

开源版本是预设的18个级别：1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h。可以在配置文件中修改。

商业版可以随意指定延迟的时间。

### 批量消息

### 过滤消息

可以根据Tags或者sql92的语法进行过滤消费对应的消息。==只有push模式下才可以使用过滤条件==。

### 事务消息

流程图

<img width="707" alt="image-20221028233843871" src="https://github.com/hqf1117/hqf1117.github.io/assets/30412361/5513830c-4dc6-47cb-a4aa-34f77df1f3dc">

直到接收COMMIT的信号以后，事务消息才能被consumer接收到并且消费。定时回查次数为15次。

###  消息轨迹

有一个默认的topic，RMQ_SYS_TRACE_TOPIC，用来记录消息的链路信息。也可以自己指定一个topic来记录。

## stream流计算

```xml
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-stream-rocketmq</artifactId>
</dependency>
```

## 高级特性

### 消息存储

1. MQ收到一条消息后，需要向生产者返回一个ACK响应，并将消息存储起来。
2. MQ Push一条消息给消费者后，等待消费者的ACK的响应，需要将消息标记为已消费。如果没有标记为消费，MQ会不断尝试往消费者推送这条消息。
3. MQ需要定期删除一些过期的消息。默认保存3天，超过3天的message会被删除。 

#### 提高存储性能

- 顺序写：提前申请一块连续的磁盘空间，然后顺序写入消息。

- 异步或同步刷盘：flushDiskType 配置项更改 

  <img width="397" alt="image-20221031190541650" src="https://github.com/hqf1117/hqf1117.github.io/assets/30412361/edd0e5d4-645f-40f0-bf7c-86623932b865">

- 零拷贝： MMap的方式进行零拷贝，文件限制大小为1.5->2G

#### 存储结构

- CommitLog：所有消息会顺序存入到CommitLog中，每个固定大小1G。以第一条消息的偏移量为文件名。
- ConsumerQueue：一个MessageQueue一个文件，记录当前被哪些消费者组消费到了哪一条。
- IndexFile：为了查询消息方便并且不影响主流程的索引文件。  

<img width="387" alt="image-20221031104842833" src="https://github.com/hqf1117/hqf1117.github.io/assets/30412361/a0dcdf55-e847-4038-b412-df9d541766dd">

#### 消息主从复制

同样存在同步和异步两种方式。

### 消息消费

RocketMQ是支持一个消费者订阅多个topic，==需要保证组内的消费者订阅的topic都必须一致==，否则就会出现订阅的topic被覆盖的情况。

messageQueue在clustering模式下只能被同一组中的其中一个consumer消费。所以需要配置相应的分配策略。

### 消息重试

当broker发送message给consumer消费，但是没有收到返回消息时，将会启动消息重试机制。

broker会自动创建（%RETRY%消费者组名）的重试队列。

<img width="419" alt="image-20221031224729597" src="https://github.com/hqf1117/hqf1117.github.io/assets/30412361/b66274ae-7cd2-4955-9337-910f36b6a7d1">

默认是16次，可以配置更多，统一为2小时的间隔。

### 死信队列

消息重试超过设置的次数，仍然失败的状态下，转为死信队列。  

%DLQ%消费者组名

默认权限是2，无法发送也无法读取。需要手动修改权限以后才能操作数据。  

### 消息幂等

- At most once
- At least once
- Exactly once:仅商业版支持

如何处理？

建议传递key属性作为唯一标识来保证消息幂等性。