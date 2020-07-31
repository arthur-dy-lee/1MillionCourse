

## 一、生产者

### 1.1 生产者流程启动

![](blogpic/Snipaste_2020-06-14_14-22-02-生产者.png)



![Snipaste_2020-06-14_16-22-02-Kafka生产者初始化](blogpic/Snipaste_2020-06-14_16-22-02-Kafka生产者初始化.png)

 Kafka同步和异步(回调)

![](blogpic/Snipaste_2020-06-14_15-21-08-同步和异步(回调).png)



初始化时，参数配置

生产者向broker发送消息，重试机制，默认100ms

![](blogpic/Snipaste_2020-06-14_15-48-04-重试100ms.png)



![Snipaste_2020-06-14_15-51-25-参数1](blogpic/Snipaste_2020-06-14_15-51-25-参数1.png)



![Snipaste_2020-06-14_15-52-50-参数2](blogpic/Snipaste_2020-06-14_15-52-50-参数2.png)



Kafka整个网络都是由NetworkClient负责的

connections.max.idle.ms 一个网络最多空闲9分钟，超过后就关闭网络

![Snipaste_2020-06-14_16-06-51-空闲9分钟](blogpic/Snipaste_2020-06-14_16-06-51-空闲9分钟.png)

第二个比较重要的NetworkClient参数

max.in.flight.requests.per.connection 默认是5，producer向broker 发送数据的时候，其实有很多个网络连接。每个网络连接可以忍受produce 端发送给broker 消息然后消息滑响应个数

已经发送出去5个了，消息返回还未响应，第6条消息是不能发送的。因为Kafka的重试机制，那么会造成乱序发送。如果想不乱序，设置成1即可。

![Snipaste_2020-06-14_16-01-55_发送5个消息](blogpic/Snipaste_2020-06-14_16-01-55_发送5个消息.png)



![Snipaste_2020-06-14_16-12-27-数据更安全，不会丢数据](blogpic/Snipaste_2020-06-14_16-12-27-数据更安全，不会丢数据.png)



![](blogpic/Snipaste_2020-06-14_16-20-13-线程隔离.png)



生产者从broker中获取元数据，以感知往哪个Partition中写。

![Snipaste_2020-06-14_16-23-55-元数据](blogpic/Snipaste_2020-06-14_16-23-55-元数据.png)



cluster数据结构

![Snipaste_2020-06-14_16-26-06-cluster数据结构](blogpic/Snipaste_2020-06-14_16-26-06-cluster数据结构.png)



![Snipaste_2020-06-14_16-28-36-partitionInfo](blogpic/Snipaste_2020-06-14_16-28-36-partitionInfo.png)

元数据关系图如下

![Snipaste_2020-06-14_16-29-23-Metadata元数据](blogpic/Snipaste_2020-06-14_16-29-23-Metadata元数据.png)



### 1.2 发送消息send

![Snipaste_2020-06-14_16-35-20-send](blogpic/Snipaste_2020-06-14_16-35-20-send.png)

[发送步骤](https://blog.csdn.net/weixin_43167418/article/details/104218328) 

1. 同步等待拉取元数据 maxBlockTimeMs 最多能等待多久
2. 对消息的key，value进行序列化
3. 根据分区器选择消息应该发往的分区
4. 确认一下消息的大小是否超过了最大值。默认初始化maxRequestSeze为1M，如果大于1M，则会抛异常。如果超过32M也会报销，因为缓存大小RecordAccumulator为32M。
5. 根据元数据信息封装分区对象
6. 给消息绑定回调函数
7. 消息存入 RecordAccumulator



![Snipaste_2020-06-14_16-40-29-sender2](blogpic/Snipaste_2020-06-14_16-40-29-sender2.png)



sender线程获取元数据Metadate，Sender#run方法获取元数据。刚启动的时候，获取的元数据为空，只是获取了cluster。刚开始执行run方法时，使用`this.client.poll(pollTimeout, now)`拉取元数据信息。client就是NetworkClient，即调用NetworkClient#poll。

处理服务器端发回来的响应

![Snipaste_2020-06-14_19-17-47-处理服务器端发回来的响应1](blogpic/Snipaste_2020-06-14_19-17-47-处理服务器端发回来的响应1.png)





![Snipaste_2020-06-14_19-21-15-更新元数据的信息](blogpic/Snipaste_2020-06-14_19-21-15-更新元数据的信息.png)

Metadata版本号+1

![Snipaste_2020-06-14_19-21-40-更新版本号](blogpic/Snipaste_2020-06-14_19-21-40-更新版本号.png)

Metadata唤醒sender线程

![Snipaste_2020-06-14_19-22-08-获取元数据后唤醒线程](blogpic/Snipaste_2020-06-14_19-22-08-获取元数据后唤醒线程.png)



send发消息的时候，发现没有元数据（元数据中有分区partition），不知道往哪个partition里写，然后sender纯程休眠。另一个线程去获取元数据，从Metadata中获取元数据，获取后版本号+1，更新元数据后，唤醒 sender线程发送数据。

![Snipaste_2020-06-14_19-22-37-获取元数据的总流程](blogpic/Snipaste_2020-06-14_19-22-37-获取元数据的总流程.png)



元数据在发送的时候，如果指定Key，则取hash值，会发送到同一个分区partition上；如果不指定Key，则会出对partiton轮循，实现负载均衡。



RecordAccumulator缓存后，如果批次满了或者新创建出来的一个批次，但会唤醒sender线程进行发送。

![Snipaste_2020-06-14_19-42-00-RecordAccumulator缓存批次](blogpic/Snipaste_2020-06-14_19-42-00-RecordAccumulator缓存批次.png)



刚开始进来的时候，Batchs是空的，也没有批次。

![Snipaste_2020-06-14_21-28-01-批次为空](blogpic/Snipaste_2020-06-14_21-28-01-批次为空.png)

然后计算批次的大小。根据当前消息大小和批次默认值（16K）计算批次大小，如果消息大于批次默认值，则消息大小就是一个批次。所以批次大小要根据公司生产的消息大小进行调整，如果太小就展现不出批次的优势。

向Dqueue中添加批次

![Snipaste_2020-06-14_21-38-24-添加批次](blogpic/Snipaste_2020-06-14_21-38-24-添加批次.png)



Batches = new CopyOnWriteMap<>() ，它是一个Kafka自定义的数据结构，它继承ConcurrentMap。Map中get和put方法，put时synchronized加锁，是一个线程安全的操作，每插入一条数据，都开一个新的内存空间，说明读写是分离的。那么读是操作就是线程安全的，读不需要加锁，读的性能就很高。

实现了在保证线程安全的前提下，实现了高性能。这此适合多读少写的情况。

![Snipaste_2020-06-14_21-46-59-CopyOnWriteMap#put](blogpic/Snipaste_2020-06-14_21-46-59-CopyOnWriteMap#put.png)

只能创建队列的时候，才会进行写操作。

如果是来20万条消息，TopicA分为0和1，分别对应TopicPartition 0和 TopicPartition 1

只会写2次队列，但会读20万次队列。



![Snipaste_2020-06-14_21-54-10-batchs读写分离](blogpic/Snipaste_2020-06-14_21-54-10-batchs读写分离.png)



在RecordAccumulator中使用分段加锁的方法，缩小锁的粒度，提高性能！RecordAccumulator的2次synchronized(dq)可以看一下。

线程1进来申请内存，线程2进来申请内存， synchronized(dq)加锁了，所以线程1申请到了批次（RecordAppendResult写完批次RecoredBatch后的返回值），等线程1释放锁后，到线程2的时候，就有批次了，就会释放申请到的内存。

![Snipaste_2020-06-14_22-15-13-批次申请时线程2释放内存](blogpic/Snipaste_2020-06-14_22-15-13-批次申请时线程2释放内存.png)





#### 内存池设计

程序刚开始包的时候，availableMemory默认是32M的，然后从32M从不创分配ByteBuffer（默认一个批次16K）内存，为RecordBatch封装出来一个批次(RecordBatc)，availableMemory逐渐减少。在RecoredBatch发送后，再次清空，然后把空的ByteBuffer放回内存池。下次再分配的时候，直接把ByteBuffer内存块给RecordBatch即可。

内存池32M大小=已分配的内存块ByteBuffer+availableMemory

![Snipaste_2020-06-14_22-17-30-缓存设计](blogpic/Snipaste_2020-06-14_22-17-30-缓存设计.png)



内存池分配内存 

free.allocate （BufferPool#allocate），先判断是否大于缓存32M，如果申请的大小大于32M，那么直接抛异常。然后通过lock.lock加锁。

判断poolableSize默认批次的大小，申请 的这个内存大小是否等于默认的内存大小，如果相等则直接返回；如果不相等，则计算可用的总内存，总内存要大于你要申请的内存。还有一种情况，剩下的内存不够用了，比如还剩下10K的内存，但我们这次申请的内存是32K，我们就等别人释放内存，别释放多少就先分配多少，一点一点累加，直到满足足够的内存。



内存池释放内存

![Snipaste_2020-06-14_23-11-21-内存池释放内存](blogpic/Snipaste_2020-06-14_23-11-21-内存池释放内存.png)

归还了内存后，就会唤醒正在等待分配内存的线程

![Snipaste_2020-06-14_23-14-19-内存池释放内存-唤醒等待内存的线程](blogpic/Snipaste_2020-06-14_23-14-19-内存池释放内存-唤醒等待内存的线程.png)



![](blogpic/Snipaste_2020-06-14_23-21-18-producer发送消息过程.png)



![](blogpic/Snipaste_2020-06-14_23-21-18-producer发送消息过程2.png)



sender被唤醒后获取元数据

![Snipaste_2020-06-14_23-24-54-sender被唤醒后获取元数据](blogpic/Snipaste_2020-06-14_23-24-54-sender被唤醒后获取元数据.png)

哪些有消息可以发送

![Snipaste_2020-06-14_23-27-43-哪些有消息可以发送](blogpic/Snipaste_2020-06-14_23-27-43-哪些有消息可以发送.png)

在RecordAccumulator中解析。哪些有消息可以发送，来看一下一个批次可以发送出去的条件。然后找可发送的主机。



lingerMs默认是0，就是有一条就发一条，生产应该设置成100ms。

当时时间-上一次重次时间，换句话说，一个批次到100ms，即使没有写满就会发送出去。

1. 写满批次则发送。

2. 如果等超时了，则发送。

3. 内存不够了（消息发送后，就会释放内存）

![Snipaste_2020-06-14_23-29-26-遍历主机](blogpic/Snipaste_2020-06-14_23-29-26-遍历主机.png)



![Snipaste_2020-06-15_00-26-10-client#ready](blogpic/Snipaste_2020-06-15_00-26-10-client#ready.png)





如果发往同一台服务器的批次相同，但合并。

同一个broker的partition为同一组，对分组进行合并，这样和服务器broker通讯减少了。减小网络传输次数，性能就上来了。

![Snipaste_2020-06-14_23-31-36-同一个broker的partition为同一组](blogpic/Snipaste_2020-06-14_23-31-36-同一个broker的partition为同一组.png)

#### 客户端网络设计

NetworkClient发送

![Snipaste_2020-06-14_23-36-00-NetworkClient发送](blogpic/Snipaste_2020-06-14_23-36-00-NetworkClient发送.png)

NetworkClient结构

![Snipaste_2020-06-15_00-30-50-NetworkClient-selector](blogpic/Snipaste_2020-06-15_00-30-50-NetworkClient-selector.png)



看一下世界级的Kafka网络通讯给的默认值是多少？

![Snipaste_2020-06-15_00-37-20-待查询](blogpic/Snipaste_2020-06-15_00-37-20-待查询.png)

建立连接

![Snipaste_2020-06-15_00-39-54-建立连接](blogpic/Snipaste_2020-06-15_00-39-54-建立连接.png)



![Snipaste_2020-06-15_00-40-58-建立连接2](blogpic/Snipaste_2020-06-15_00-40-58-建立连接2.png)



![Snipaste_2020-06-15_00-47-50-sender建立连接](blogpic/Snipaste_2020-06-15_00-47-50-sender建立连接.png)

注册读事件，是为了服务器端回来的时候，接收请求。

![Snipaste_2020-06-15_00-46-23-建立连接后，更新事件](blogpic/Snipaste_2020-06-15_00-46-23-建立连接后，更新事件.png)

client.send发送请求的时候，为了向服务器端发送数据，会注册一个写事情。

client.send->selector.send

绑定op_write事件

![Snipaste_2020-06-15_00-54-10-绑定op_write事件](blogpic/Snipaste_2020-06-15_00-54-10-绑定op_write事件.png)



把消息发送出去

![Snipaste_2020-06-15_00-57-23-把消息发送出去](blogpic/Snipaste_2020-06-15_00-57-23-把消息发送出去.png)

















