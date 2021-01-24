![image-20201020123034445](https://gitee.com/chengbo123/images/raw/master/image-20201020123034445.png)

# 1.Spark基本概念

* Application(应用程序)：是指用户编写的Spark应用程序，包含驱动程序(Driver)和分布在集群中多个节点上运行的Executor代码，在执行过程中由一个或多个作业组成。

* Driver(驱动程序)：Spark中的Driver即运行上述Application的main函数并且创建SparkContext，其中创建SparkContext的目的是为了准备Spark应用程序的运行环境。在Spark中由SparkContext负责与ClusterManager通信，进行资源的申请、任务的分配和监控等；当Executor部分运行完毕后，Driver负责将SparkContext关闭。通常用SparkContext代表Driver。

* Cluster Manager(集群资源管理器)：是指在集群上获取资源的外部服务，目前包括以下几种：

  * Standalone:Spark原生的资源管理，由Master负责资源的管理
  * Hadoop Yarn:由Yarn中的Resource Manager负责资源的管理。
  * Mesos：由Mesos中的Mesos Master负责资源的管理

* Worker：集群中任何可以运行Application代码的节点，类似于Yarn中的NodeManager节点。在Standalone模式中指的是通过Slave文件配置的Worker节点，在Spark on Yarn模式中指的是NodeManger节点。

* Master：Spark Standalone运行模式下的主节点，负责管理和分配集群资源来运行Spark Application。

* Executor:Application 运行在Worker节点上的一个进程，该进程负责运行Task，并负责将数据存在内存或者磁盘上，每个Application都有各自独立的一批Executor。在Spark on Yarn模式下，其进程名称为CoarseGrainedExecutorBackend,类似于Hadoop MapReduce中的YarnChild。一个CoarseGrainedExecutorBackend进程有且仅有一个Executor对象，它负责将Task包装成taskRunner,并从线程池中抽取一个空闲线程运行Task。每个CoarseGrainedExecutorBackend能并行运行Task的数量就取决于分配给它的CPU的个数。

  

# 2. Spark消息通信架构

![image-20201020124652861](https://gitee.com/chengbo123/images/raw/master/image-20201020124652861.png)



在Spark定义了通信框架接口，这些接口实现中调用Netty的具体方法(Spark2.0之前使用Akka)。在框架中以RpcEndPoint和RpcEndpointRef实现了Actor和ActorRef相关动作，其中RpcEndpointRef是RpcEndPoint的引用，在消息通信中消息发送方持有引用RpcEndpointRef。

通信框架使用了工厂设计模式实现，实现对Netty的解耦，能够根据需要引入其他的消息通信工具。具体实现步骤

（1）首先定义了RpcEnv和RpcEnvFactory两个抽象类，在RpcEnv定义了RPC通信框架启动、停止和关闭等抽象方法，在RpcEnvFactory中定义创建抽象方法。

（2）然后在NettyRpcEnv和NettyRpcEnvFactory类中使用Netty对继承的方法进行了实现。在NettyRpcEnv中启动setuoEndpoint方法，把RpcEndpoint和RpcEndpointRef相互以键值对方式存放在线程安全的ConcurrentHashMap中。

（3）最后在RpcEnv的object类中通过反射方式实现了创建RpcEnv的实例的静态方法。

在各模块的使用中，如Master、Worker等，会先使用RpcEnv的静态方法创建RpcEnv实例，然后实例化Master，由于Master继承自ThreadSafeRpcEndpoint,创建的Master实例是一个线程安全的终端点，接着调用RpcEnv启动终端点方法，把Master的终端点和其对应的引用注册到RpcEnv中。在消息通信中，其对象只要获取了Master终端点的引用，就能够发送消息给Master进行通信。