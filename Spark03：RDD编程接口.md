<center><b><font color=#A52A2A size=5 >公众号：数据挖掘与机器学习笔记</font></b></center>

Spark中提供了通用接口来抽象每个RDD，包括：

* 分区信息：数据集的最小分片
* 依赖关系：指向其父RDD
* 函数：基于父RDD的计算方法
* 划分策略和数据位置的元数据

![image-20200902104853809](http://qfth8dccq.hn-bkt.clouddn.com/images/image-20200902104853809.png)

## 1.RDD分区

RDD的分区是一个逻辑概念，变换前后的新旧分区在物理上可能是同一块内存或存储，这种优化防止函数式不变性导致的内存需求无限扩张。在RDD操作中可以使用Partitions方法获取RDD划分的分区数，也可以设定分区数目。如果没有指定将使用默认值，而默认数值是该程序所分配到的CPU核数，如果是从HDFS文件创建，默认为文件的数据块数。

```scala
//默认两个分区
val part=sc.textFile("input/input1.txt")
println(part.partitions.size)

//显式设置为4个partitions
val part=sc.textFile("input/input1.txt",minPartitions = 4)
println(part.partitions.size)
```

## 2. RDD首选位置(PreferredLocations)

Spark在形成任务的DAG时，会尽可能把计算分配到靠近数据的位置，减少数据网络传输。当RDD产生的时候存在首选位置，如HadoopRDD分区的首选位置就是HDFS块所在的节点；当RDD分区被缓存，则计算应该发送到缓存分区所在的节点进行，再不然回溯RDD的“血统”一直找到具有首选位置属性的父RDD，并据此决定子RDD的位置。

## 3.RDD依赖关系

Spark中RDD存在两种依赖：窄依赖(Narrow Dependencies)和宽依赖(Wide Dependencies)。

<img src="http://qfth8dccq.hn-bkt.clouddn.com/images/image-20200902111336831.png" alt="image-20200902111336831" style="zoom: 50%;" />

* 窄依赖：每个父RDD的分区至多被一个子RDD的分区使用
* 宽依赖：多个子RDD的分区依赖一个父RDD的分区

**区别：**

* 窄依赖允许在单个集群节点上流水线式执行，这个节点可以计算所有父级分区；宽依赖需要所有父RDD的数据可用，并且数据已经通过类MR操作Shuffle完成
* 在窄依赖中，节点失败后的恢复更加高效。因为只有丢失的父级分区重新计算，并且这些丢失的父级分区可以并行地在不同节点上重新计算。而在宽依赖地继承关系中，单个节点地失败可能导致一个RDD的所有祖先RDD中的一些分区丢失，导致计算的重新执行。

```scala
val part = sc.textFile("input/input1.txt")
    val wordmap = part.flatMap(_.split(" ")).map(x => (x, 1))
    println(wordmap)
    //wordmap的依赖关系为OneToOneDependency，属于窄依赖
    wordmap.dependencies.foreach {
      dep =>
        println("dependency type:" + dep.getClass)
        println("dependency RDD:" + dep.rdd)
        println("dependency partitions:" + dep.rdd.partitions)
        println("dependency partitions size:" + dep.rdd.partitions.length)
    }

    val wordreduce = wordmap.reduceByKey(_ + _)
    println(wordreduce)
    wordreduce.dependencies.foreach{
      dep =>
        println("dependency type:" + dep.getClass)
        println("dependency RDD:" + dep.rdd)
        println("dependency partitions:" + dep.rdd.partitions)
        println("dependency partitions size:" + dep.rdd.partitions.length)
    }
```

<img src="http://qfth8dccq.hn-bkt.clouddn.com/images/image-20200903100355916.png" alt="image-20200903100355916" style="zoom:67%;" />

## 4.RDD分区计算

RDD的基本单位是partition,计算函数都是对迭代器进行复合，不需要保存每次计算的结果。如mapPartitions对每个分区内容作为整体来处理。

```scala
 val a = sc.parallelize(1 to 12, 3)
    a.mapPartitions {
      x =>
        var res = List[(Int, Int)]()
        var pre = x.next()
        while (x.hasNext) {
          val cur = x.next()
          res ::= (pre, cur
          )
          pre = cur
        }
        res.iterator
    }.foreach(t2 => print(t2))
```

![image-20200903101735418](http://qfth8dccq.hn-bkt.clouddn.com/images/image-20200903101735418.png)

上述代码把每个分区中的元素和下一个元素组成一个Tuple,因为分区中最后一个元素没有下一个元素，所以没有(4,5)和(8,9)

## 5. RDD分区函数

分区的划分对于Shuffle类操作很关键，决定了该操作的父RDD和子RDD之间的依赖类型。在Spark中默认提供两种分区划分器：哈希分区划分器(HashPartitioner)和范围分区划分器(RangePartitioner)，且Partitioner只存在于(K,V)类型的RDD中，对于非(K,V)类型的Partitioner值为None。

```scala
    val mapRDD = sc.textFile("input/input1.txt")
    println(mapRDD.partitioner)
    val groupRDD = mapRDD.map(x => (x, x)).groupByKey(new HashPartitioner(4))
    print(groupRDD.partitioner)
```

## **参考：**

[1]《图解Spark：核心技术与案里实战》

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200828221113544.jpg#pic_center)