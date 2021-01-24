## 1. 简介
&emsp;&emsp;SparkConf类负责管理Spark的所有配置项,每个Spark程序都离不开SparkConf，基本上在Spark程序的开始都会进行一个参数的配置，如
new SparkConf().setMaster("local").setAppName("My app")。这里主要讲下SparkConf类源码的基本内容。

## 2.Spark配置
### 2.1 SparkConf构造方法

```
class SparkConf(loadDefaults: Boolean) extends Cloneable with Logging with Serializable
```

```
  //如果loadDefaults为true，则加载系统属性配置
  if (loadDefaults) {
    loadFromSystemProperties(false)
  }

  /**
   * 加载系统属性配置
   * @param silent 当给定的键(key)已经弃用，即该配置项已经过期时，是否打印警告信息
   * @return
   */
  private[spark] def loadFromSystemProperties(silent: Boolean): SparkConf = {
    //加载所有”spark.* “的system properties
    for ((key, value) <- Utils.getSystemProperties if key.startsWith("spark.")) {
      set(key, value, silent)
    }
    this
  }
```

构造方法中的参数loadDefault表示是否从系统属性中加载配置项；继承Cloneable 类主要是为了克隆SparkConf对象

### 2.2 SparkConf参数设置
Spark中的每一个组件都直接或者间接的使用SparkConf所存储的配置属性，这些属性都存储在数据结构ConcurrentHashMap中即
```
  //考虑到了并发环境下的线程安全性问题，创建一个ConcurrentHashMap，用于存储键值对形式的配置项
  //键、值类型均为String
  private val settings = new ConcurrentHashMap[String, String]()

```

设置SparkConf配置项的方法有三种：
#### (1)从系统默认属性中加载

```
  private[spark] def loadFromSystemProperties(silent: Boolean): SparkConf = {
    //加载所有”spark.* “的system properties
    for ((key, value) <- Utils.getSystemProperties if key.startsWith("spark.")) {
      set(key, value, silent)
    }
    this
  }
```
如果new SparkConf()不传入false，则默认加载系统配置

#### (2) 使用SparkConf的set*方法设置

```
/** 设置配置变量 */
  def set(key: String, value: String): SparkConf = {
    set(key, value, false)
  }

  private[spark] def set(key: String, value: String, silent: Boolean): SparkConf = {
    //键和值均不能为空
    if (key == null) {
      throw new NullPointerException("null key")
    }
    if (value == null) {
      throw new NullPointerException("null value for " + key)
    }
    //silent 当给定的键(key)已经弃用，即该配置项已经过期时，是否打印警告信息
    if (!silent) {
      logDeprecationWarning(key)
    }
    settings.put(key, value) //放到ConcurrentHashMap中
    this
  }
```

```
/**
   * 要连接的 Master URL, 例如 "local"表示在本地运行一个线程, "local[4]" 表示在本地使用4个核运行程序
   *  "spark://master:7077" 表示运行 Spark standalone cluster.
   */
  def setMaster(master: String): SparkConf = {
    set("spark.master", master)
  }

  /**  Spark web UI 中显示的程序运行名称. */
  def setAppName(name: String): SparkConf = {
    set("spark.app.name", name)
  }

  /** 发布到集群上的JAR文件 */
  def setJars(jars: Seq[String]): SparkConf = {
    for (jar <- jars if (jar == null)) logWarning("null jar passed to SparkContext constructor")
    set("spark.jars", jars.filter(_ != null).mkString(","))
  }
```
#### (3)克隆SparkConf对象
&emsp;&emsp;在某些情况下，同一个SparkConf实例中的配置信息需要被多个组件公用，而我们往往会想到的方法是将SparkConf实例定义为全局变量或者通过参数传递给其他组件，但是这样会引入并发问题，虽然settings数据结构为ConcurrentHashMap是线程安全的，而且ConcurrentHashMap也被证明是高并发下性能表现不错的数据结构，但是存在并发，就一定有性能的损失问题，也可以创建一个SparkConf实例b，并将a中的配置信息全部拷贝到b中，这样会浪费内存，导致代码散落在程序的各个部分。

　&emsp;&emsp;SparkConf继承了Cloneable物质并实现了clone方法，可以通过Cloneable物质提高代码的可利用性
```
/**
   * 复制SparkConf对象
   * Copy this object */
  override def clone: SparkConf = {
    val cloned = new SparkConf(false)
    settings.entrySet().asScala.foreach { e =>
      cloned.set(e.getKey(), e.getValue(), true)
    }
    cloned
  }
```
### 2.3 获取配置参数
&emsp;&emsp;获取配置项只有一个途径，即调用Get类方法。Get类方法同样有很多实现，基础的get()与getOption()如下所示。

```
/**
   * 获取参数配置，没有配置该参数则返回默认值
   * Get a parameter, falling back to a default if not set */
  def get(key: String, defaultValue: String): String = {
    getOption(key).getOrElse(defaultValue)
  }

  /** Get all executor environment variables set on this SparkConf */
  def getExecutorEnv: Seq[(String, String)] = {
    getAllWithPrefix("spark.executorEnv.")
  }

```

### 2.4 移除配置参数
```
/**
   * 从配置项中移除参数
   * Remove a parameter from the configuration */
  def remove(key: String): SparkConf = {
    settings.remove(key)
    this
  }

  private[spark] def remove(entry: ConfigEntry[_]): SparkConf = {
    remove(entry.key)
  }
```
### 2.5 设置序列化方式
```
/**
   * 使用Kryo序列化并且使用Kryo注册给定的类集合
   * 如果多次调用，它将把来自所有调用的类附加在一起
   * Use Kryo serialization and register the given set of classes with Kryo.
   * If called multiple times, this will append the classes from all calls together.
   */
  def registerKryoClasses(classes: Array[Class[_]]): SparkConf = {
    val allClassNames = new LinkedHashSet[String]()
    allClassNames ++= get("spark.kryo.classesToRegister", "").split(',').map(_.trim)
      .filter(!_.isEmpty)
    allClassNames ++= classes.map(_.getName)

    set("spark.kryo.classesToRegister", allClassNames.mkString(","))
    set("spark.serializer", classOf[KryoSerializer].getName)
    this
  }
```
### 2.6 配置项检查
检查非法和弃用的配置
```
private[spark] def validateSettings() {
    if (contains("spark.local.dir")) {
      val msg = "In Spark 1.0 and later spark.local.dir will be overridden by the value set by " +
        "the cluster manager (via SPARK_LOCAL_DIRS in mesos/standalone and LOCAL_DIRS in YARN)."
      logWarning(msg)
    }

    val executorOptsKey = "spark.executor.extraJavaOptions"
    val executorClasspathKey = "spark.executor.extraClassPath"
    val driverOptsKey = "spark.driver.extraJavaOptions"
    val driverClassPathKey = "spark.driver.extraClassPath"
    val driverLibraryPathKey = "spark.driver.extraLibraryPath"
    val sparkExecutorInstances = "spark.executor.instances"
}
```
![数据挖掘与机器学习笔记](https://img-blog.csdnimg.cn/20200822103248289.jpg#pic_center)
