以Standalone模式为例。

# 1.Spark启动时的消息通信

Spark启动过程主要是进行Master和Worker之间的通信，如下图所示。

![image-20201022121236690](https://gitee.com/chengbo123/images/raw/master/image-20201022121236690.png)

详细过程如下：

（1）当Master启动时，随之启动各Woker,Worker启动时会创建通信环境RpcEnv和终端点EndPoint,并向Master发送注册Worker的消息RegisterWorker.

由于Woker可能需要注册到多个Master中(如HA环境),在Worker的tryRegisterAllMasters方法中创建注册线程池registerMasterThreadPool,把需要申请注册的请求放在该线程池中，然后通过该线程池启动注册线程。在该注册过程中，获取Master终端点引用，接着调用registerWithMaster方法，根据Master终端点引用的send方法发送注册RegisterWorker消息。

（2）Master收到消息后，需要对Worker发送的消息进行验证、记录。如果注册成功，则发送RegisteredWorker消息给对应的Worker，告诉Woker以将完成注册，随之进行步骤3，即Worker定期发送心跳信息给Master；如果在注册过程中失败，则会发送RegisterWorkerFailed消息，Worker打印出错日志并结束Worker启动。

在Master中，Master接收到Worker注册消息后，先判断Master当前状态是否处于STANDBY状态，如果是则忽略该消息，如果在注册列表中发现了该Worker的编号，则发送注册失败的消息。判断完毕后使用registerWorker方法把该Worker加入到列表中，用于集群进行处理任务时进行调度。

（3）当Worker接收到注册成功的消息后，会定时发送心跳信息Heartbeat给Master，以便Master了解Worker的实时状态。间隔时间可以在spark.worker.timeout中设置，注意的是，该设置值的1/4为心跳间隔。

当Woker获取到注册成功消息后，先记录日志并更新Master信息，然后启动定时调度进程发送心跳信息。



# 2. Spark运行时消息通信

用于提交应用程序时，应用程序的SparkContext会向Master发送应用注册消息，并由Master给该应用分配Executor，Executor启动后，Executor会向SparkContext发送注册成功消息；

