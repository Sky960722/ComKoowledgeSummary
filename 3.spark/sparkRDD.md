# SparkRDD

## RDD定义

RDD（Resilient Distributed Dataset）叫做弹性分布式数据集，是Spark中最基本的数据处理模型。代码中是一个抽象类，它代表一个弹性的、不可变、可分区、里面的元素可并行计算的集合。

- 弹性
  - 存储的弹性：内存与磁盘的自动切换；
  - 容错的弹性：数据丢失可以自动恢复；
  - 计算的弹性：计算出错重试机制；
  - 分片的弹性：可根据需要重新分片。
- 分布式：数据存储在大数据集群不同节点上
- 数据集：RDD 封装了计算逻辑，并不保存数据
- 数据抽象：RDD 是一个抽象类，需要子类具体实现
- 不可变：RDD 封装了计算逻辑，是不可以改变的，想要改变，只能产生新的RDD，在新的RDD 里面封装计算逻辑
- 可分区、并行计算

## RDD 转换算子

1. RDD根据数据处理方式的不同将算子整体上分为Value类型、双Value类型和Key-Value类型
### Value类型
   1. map
      1. 作用：将处理的数据逐条进行映射转换，这里的转换可以是类型的转换，也可以是值的转换。

   ~~~scala
   //函数签名
   def map[U: ClassTag](f: T => U): RDD[U]
       
       val sparkConf = new SparkConf().setMaster("local[*]").setAppName("Operator")
       val sc = new SparkContext(sparkConf)
   
   	val rdd = sc.makeRDD(List(1, 2, 3, 4))
    	val mapRDD = rdd.map(_ * 2)
   	mapRDD.collect().foreach(println)
       sc.stop()
       
   ~~~

   2. mapPartitions

      	1. 作用：将待处理的数据以分区为单位发送到计算节点进行处理，这里的处理是指可以进行任意的处理，哪怕是过滤数据。

      ~~~scala
      //函数签名
      def mapPartitions[U: ClassTag]( 
          f: Iterator[T] => Iterator[U],
      	preservesPartitioning: Boolean = false): RDD[U]
      
      val rdd: RDD[Int] = sc.makeRDD(List(1, 2, 3, 4), 2)
      
      val mapRDD = rdd.mapPartitions(
            iter => {
              println(">>>>>>>>>>>>>>>>")
              iter.map(_ * 2)
            }
          )
      
      ~~~

      

   3. mapPartitionsWithIndex

      	1. 作用：将待处理的数据以分区为单位发送到计算节点进行处理，这里的处理是指可以进行任意的处理，哪怕是过滤数据，在处理时同时可以获取当前分区索引。

      ~~~scala
      //函数签名
      def mapPartitionsWithIndex[U: ClassTag](
      	f: (Int, Iterator[T]) => Iterator[U],
      	preservesPartitioning: Boolean = false): RDD[U]
      
      val rdd = sc.makeRDD(List(1, 2, 3, 4), 2)
      
      //获得第二个分区的数据
          val mpiRDD = rdd.mapPartitionsWithIndex(
            (index, iter) => {
              if (index == 1) {
                iter
              } else {
                Nil.iterator
              }
            }
          )
      ~~~

      

   4. flatMap

      	1. 作用：将处理的数据进行扁平化后再进行映射处理，所以算子也称之为扁平映射

      ~~~scala
      //函数签名
      def flatMap[U: ClassTag](f: T => TraversableOnce[U]): RDD[U]
      
      val rdd = sc.makeRDD(List(List(1,2),List(3,4)))
      
          val flatRDD: RDD[Int] = rdd.flatMap(
            list => {
              list
            }
          )
      ~~~

   5. glom

      	1. 作用：将同一个分区的数据直接转换为相同类型的内存数组进行处理，分区不变

      ~~~scala
      //函数签名
      def glom(): RDD[Array[T]]
      
      val rdd: RDD[Int] = sc.makeRDD(List(1, 2, 3, 4), 2)
      
      val glomRDD: RDD[Array[Int]] = rdd.glom()
      
      glomRDD.collect().foreach(data => println(data.mkString(",")))
      ~~~

   6. groupBy

      	1. 作用：将数据根据指定的规则进行分组, 分区默认不变，但是数据会被打乱重新组合，我们将这样的操作称之为shuffle。极限情况下，数据可能被分在同一个分区中
          一个组的数据在一个分区中，但是并不是说一个分区中只有一个组

      ~~~scala
      //函数签名
      def groupBy[K](f: T => K)(implicit kt: ClassTag[K]): RDD[(K, Iterable[T])]
      
      val rdd = sc.makeRDD(List(1, 2, 3, 4), 2)
      
          def groupFunction(num:Int):Int = {
            num % 2
          }
      
          val groupRDD:RDD[(Int,Iterable[Int])] = rdd.groupBy(groupFunction)
      
          groupRDD.collect().foreach(println)
      ~~~

   7. filter

      	1. 作用：将数据根据指定的规则进行筛选过滤，符合规则的数据保留，不符合规则的数据丢弃。
          当数据进行筛选过滤后，分区不变，但是分区内的数据可能不均衡，生产环境下，可能会出现数据倾斜。

      ~~~scala
      //函数签名
      def filter(f: T => Boolean): RDD[T]
      
      val rdd: RDD[Int] = sc.makeRDD(List(1, 2, 3, 4))
          val filterRDD: RDD[Int] = rdd.filter(num => num % 2 != 0)
          filterRDD.collect.foreach(println)
      ~~~

   8. sample

      	1. 根据指定的规则从数据集中抽取数据

      ~~~scala
      //函数签名
      def sample(
      	withReplacement: Boolean,
      	fraction: Double,
      	seed: Long = Utils.random.nextLong): RDD[T]
      
      val rdd = sc.makeRDD(List(1, 2, 3, 4, 5, 6, 7, 8, 9, 10))
      
          // sample 算子需要传递三个参数
          //1. 第一个参数表示，抽取数据后是否将数据返回 true(返回),false(丢弃)
          //2. 第二个参数表示，
          //                如果抽取不放回的场合：数据源中每条数据被抽取的概念，基准值的概念
          //                如果抽取放回的场合：表示数据源中的每条数据被抽取的可能次数
          //3. 第三个参数表示，抽取数据时随机算法的种子
          //                  如果不传递三个参数，那么使用的是当前系统时间
      
          println(rdd.sample(
            false,
            0.4
          ).collect().mkString(","))
      
          println(rdd.sample(
            true,
            2
          ).collect().mkString(","))
      
      ~~~

   9. distinct

      	1. 将数据集中重复的数据去重

      ~~~scala
      //函数签名
      def distinct()(implicit ord: Ordering[T] = null): RDD[T]
      def distinct(numPartitions: Int)(implicit ord: Ordering[T] = null): RDD[T]
      
      val rdd = sc.makeRDD(List(1, 2, 3, 4, 1, 2, 3, 4))
      
      val rdd1: RDD[Int] = rdd.distinct()
      
      rdd1.collect().foreach(println)
      ~~~

   10. coalesce

       	1. 根据数据量缩减分区，用于大数据集过滤后，提高小数据集的执行效率。当spark程序中，存在过多的小任务的时候，可以通过coalesce方法，收缩合并分区，减少分区的个数，减小任务调度成本

       ~~~scala
       //函数签名
       def coalesce(
           numPartitions: Int, 
           shuffle: Boolean = false,
       	partitionCoalescer: Option[PartitionCoalescer] = Option.empty)
       	(implicit ord: Ordering[T] = null): RDD[T]
       
       val rdd = sc.makeRDD(List(1, 2, 3, 4, 5, 6), 3)
       
       val newRDD: RDD[Int] = rdd.coalesce(2, true)
       
       newRDD.saveAsTextFile("output")
       ~~~

   11. repartition

       	1. 该操作内部其实执行的是coalesce操作，参数shuffle的默认值为true。无论是将分区数多的RDD转换为分区数少的RDD，还是将分区数少的RDD转换为分区数多的RDD，repartition操作都可以完成，因为无论如何都会经shuffle过程。

       ~~~scala
       //函数签名
       def repartition(numPartitions: Int)(implicit ord: Ordering[T] = null): RDD[T]
       
       // TODO 算子 - mapPartitions
       val rdd = sc.makeRDD(List(1, 2, 3, 4, 5, 6), 2)
       
       // val newRDD: RDD[Int] = rdd.coalesce(3,true)
       val newRDD: RDD[Int] = rdd.repartition(3)
       newRDD.saveAsTextFile("output")
       ~~~

   12. sortBy

          1. 该操作用于排序数据。在排序之前，可以将数据通过f函数进行处理，之后按照f函数处理的结果进行排序，默认为升序排列。排序后新产生的RDD的分区数与原RDD的分区数一致。中间存在shuffle的过程

          ~~~scala
          //函数签名
          def sortBy[K](
          f: (T) => K,
          ascending: Boolean = true,  
          numPartitions: Int = this.partitions.length)
          (implicit ord: Ordering[K], ctag: ClassTag[K]): RDD[T]
          
          val rdd = sc.makeRDD(List(6, 2, 4, 5, 3, 1), 2)
          //val newRDD: RDD[Int] = rdd.coalesce(3,true)
          val newRDD: RDD[Int] = rdd.sortBy(num => num)
          newRDD.saveAsTextFile("output")
          
          ~~~

### 双Value类型

1. intersection
   1. 对源RDD和参数RDD求交集后返回一个新的RDD
2. union
   1. 对源RDD和参数RDD求并集后返回一个新的RDD
3. subtract
   1. 以一个RDD元素为主，去除两个RDD中重复元素，将其他元素保留下来。求差集
4. zip
   1. 将两个RDD中的元素，以键值对的形式进行合并。其中，键值对中的Key为第1个RDD中的元素，Value为第2个RDD中的相同位置的元素。

~~~scala
//函数签名
def intersection(other: RDD[T]): RDD[T]
def union(other: RDD[T]): RDD[T]
def subtract(other: RDD[T]): RDD[T]
def zip[U: ClassTag](other: RDD[U]): RDD[(T, U)]

// TODO 算子 - 双Value类型
val rdd1: RDD[Int] = sc.makeRDD(List(1, 2, 3, 4))
val rdd2: RDD[Int] = sc.makeRDD(List(3, 4, 5, 6))

//交集
val rdd3: RDD[Int] = rdd1.intersection(rdd2)
println(rdd3.collect().mkString(","))

//并集
val rdd4: RDD[Int] = rdd1.union(rdd2)
println(rdd4.collect().mkString(","))

//差集
val rdd5: RDD[Int] = rdd1.subtract(rdd2)
println(rdd5.collect().mkString(","))

//拉链
val rdd6: RDD[(Int, Int)] = rdd1.zip(rdd2)
println(rdd6.collect().mkString(","))
~~~

### Key - Value类型

1. partitionBy

   1. 将数据按照指定Partitioner重新进行分区。Spark默认的分区器是HashPartitioner

   ~~~scala
   //函数签名
   def partitionBy(partitioner: Partitioner): RDD[(K, V)]
   
       val rdd = sc.makeRDD(List(1,2,3,4))
   
       val mapRdd:RDD[(Int,Int)] = rdd.map((_,1));
   
       mapRdd.partitionBy(new HashPartitioner(2)).saveAsTextFile("output")
   ~~~

2. reduceByKey

   1. 可以将数据按照相同的Key对Value进行聚合

   ~~~scala
   //函数签名
   def reduceByKey(func: (V, V) => V): RDD[(K, V)]
   def reduceByKey(func: (V, V) => V, numPartitions: Int): RDD[(K, V)]
   
   val rdd: RDD[(String, Int)] = sc.makeRDD(List(
         ("a", 1), ("a", 2), ("a", 3), ("b", 4)
       ))
   
       val reduceRdd: RDD[(String, Int)] = rdd.reduceByKey((x: Int, y: Int) => {
         println(s"x = ${x},y = ${y}")
         x + y
       })
   
       reduceRdd.collect().foreach(println)
   ~~~

3. groupByKey

   1. 将数据源的数据根据key对value进行分组

   ~~~scala
   //函数签名
   def groupByKey(): RDD[(K, Iterable[V])]
   def groupByKey(numPartitions: Int): RDD[(K, Iterable[V])]
   def groupByKey(partitioner: Partitioner): RDD[(K, Iterable[V])]
   
   val rdd: RDD[(String, Int)] = sc.makeRDD(List(
         ("a", 1), ("a", 2), ("a", 3), ("b", 4)
       ))
   
       val groupRdd: RDD[(String, Iterable[Int])] = rdd.groupByKey()
   
       groupRdd.collect().foreach(println)
   ~~~

   从shuffle的角度：reduceByKey和groupByKey都存在shuffle的操作，但是reduceByKey可以在shuffle前对分区内相同key的数据进行预聚合（combine）功能，这样会减少落盘的数据量，而groupByKey只是进行分组，不存在数据量减少的问题，reduceByKey性能比较高。
   从功能的角度：reduceByKey其实包含分组和聚合的功能。GroupByKey只能分组，不能聚合，所以在分组聚合的场合下，推荐使用reduceByKey，如果仅仅是分组而不需要聚合。那么还是只能使用groupByKey

4. aggregateByKey

   1. 将数据根据不同的规则进行分区内计算和分区间计算

   ~~~scala
   def aggregateByKey[U: ClassTag](zeroValue: U)(seqOp: (U, V) => U,
                                                 combOp: (U, U) => U): RDD[(K, U)]
   
   val rdd: RDD[(String, Int)] = sc.makeRDD(List(
         ("a", 1), ("a", 2), ("a", 3), ("b", 4)
       ),2)
   
       rdd.aggregateByKey(0)(
         (x,y) => math.max(x,y),
         (x,y) => x+y
       )
   ~~~

5. foldByKey

   1. 当分区内计算规则和分区间计算规则相同时，aggregateByKey就可以简化为foldByKey

   ~~~scala
   def foldByKey(zeroValue: V)(func: (V, V) => V): RDD[(K, V)]
   
   val rdd: RDD[(String, Int)] = sc.makeRDD(List(
         ("a", 1), ("a", 2), ("b", 3), ("b", 4),("b",5),("a",6)
       ),2)
   
        rdd.foldByKey(0)(_+_).collect().foreach(println)
   ~~~

6. combineByKey

   1. 最通用的对key-value型rdd进行聚集操作的聚集函数（aggregation function）。类似于aggregate()，combineByKey()允许用户返回值的类型与输入不一致。

   ~~~scala
   def combineByKey[C](
   	createCombiner: V => C,
   	mergeValue: (C, V) => C,
   	mergeCombiners: (C, C) => C): RDD[(K, C)]
   val rdd: RDD[(String, Int)] = sc.makeRDD(List(
         ("a", 1), ("a", 2), ("b", 3)
         , ("b", 4), ("b", 5), ("a", 6)
       ), 2)
   
       //获取相同key的数据的平均值 => (a,3) , (b,4)
       val newRDD: RDD[(String, (Int, Int))] = rdd.combineByKey(
         v => (v, 1),
         (t: (Int, Int), v) => {
           (t._1 + v, t._2 + 1)
         },
         (t1: (Int, Int), t2: (Int, Int)) => {
           (t1._1 + t2._1, t1._2 + t2._2)
         }
       )
   
   val resultRDD: RDD[(String, Int)] = newRDD.mapValues({
         case (num, cnt) => {
           num / cnt
         }
       })
   
       resultRDD.collect().foreach(println)	
   ~~~

7. sortByKey

   1. 在一个(K,V)的RDD上调用，K必须实现Ordered接口(特质)，返回一个按照key进行排序的

   ~~~scala
   def sortByKey(ascending: Boolean = true, numPartitions: Int = self.partitions.length)
   : RDD[(K, V)]
   
   
   val dataRDD1 = sparkContext.makeRDD(List(("a",1),("b",2),("c",3)))
   val sortRDD1: RDD[(String, Int)] = dataRDD1.sortByKey(true)
   val sortRDD1: RDD[(String, Int)] = dataRDD1.sortByKey(false)
   ~~~

8. join

   1. 在类型为(K,V)和(K,W)的RDD上调用，返回一个相同key对应的所有元素连接在一起的(K,(V,W))的RDD

   ~~~scala
   def join[W](other: RDD[(K, W)]): RDD[(K, (V, W))]
   
   val rdd1: RDD[(String, Int)] = sc.makeRDD(List(
         ("a", 1), ("b", 2), ("c", 3)
       ), 2)
   val rdd2: RDD[(String, Int)] = sc.makeRDD(List(
         ("a", 4), ("b", 5), ("c", 6)
       ), 2)
   
   val joinRDD:RDD[(String,(Int,Int))] = rdd1.join(rdd2)
   ~~~

9. leftOuterJoin

   1. 类似于SQL语句的左外连接

   ~~~scala
   def leftOuterJoin[W](other: RDD[(K, W)]): RDD[(K, (V, Option[W]))]
   
   val rdd1: RDD[(String, Int)] = sc.makeRDD(List(
         ("a", 1), ("b", 2), ("c", 3)
       ), 2)
   
       val rdd2: RDD[(String, Int)] = sc.makeRDD(List(
         ("a", 4), ("b", 5)//, ("c", 6)
       ), 2)
   
       val leftRDD: RDD[(String, (Int, Option[Int]))] = rdd1.leftOuterJoin(rdd2)
   
       leftRDD.collect().foreach(println)
   ~~~

10. cogroup

    1. 在类型为(K,V)和(K,W)的RDD上调用，返回一个(K,(Iterable<V>,Iterable<W>))类型的RDD

    ~~~scala
    def cogroup[W](other: RDD[(K, W)]): RDD[(K, (Iterable[V], Iterable[W]))]
    
    val dataRDD1 = sparkContext.makeRDD(List(("a",1),("a",2),("c",3)))
    val dataRDD2 = sparkContext.makeRDD(List(("a",1),("c",2),("c",3)))
    val value: RDD[( String, (Iterable[Int], Iterable[Int]))] =
    dataRDD1.cogroup(dataRDD2)
    ~~~

    ### RDD行动算子
    
    1. reduce：聚集RDD中的所有元素，先聚合分区内数据，再聚合分区间数据
    2. collect：在驱动程序中，以数组Array的形式返回数据集的所有元素
    3. count：返回RDD中元素的个数
    4. first：返回RDD中的第一个元素
    5. take：返回一个由RDD的前n个元素组成的数组
    6. takeOrdered：返回该RDD排序后的前n个元素组成的数组
    7. aggregate：分区的数据通过初始值和分区内的数据进行聚合，然后再和初始值进行分区间的数据聚合
    8. fold：折叠操作，aggregate的简化版操作
    9. countByKey：统计每种key的个数
    10. save 相关算子：将数据保存到不同格式的文件中
    11. foreach：分布式遍历RDD中的每一个元素，调用指定函数
