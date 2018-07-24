---
title: spark RDD 操作
date: 2016-08-04 20:06:29
tags:
  - Spark
categories:
  - Spark
---

# 2.1 Spark RDD操作

## 2.1.1 RDD是什么？

弹性分布式数据集RDD是Spark中的抽象数据结构类型，任何数据在Spark中都被表示为RDD。

```scala
abstract class RDD[T: ClassTag](
    @transient private var _sc: SparkContext,
    @transient private var deps: Seq[Dependency[_]]
  ) extends Serializable with Logging 
```
RDD还提供了一组丰富的操作来操作这些数据。在这些操作中，诸如map、flatMap、filter等转换操作实现了monad模式，很好地契合了Scala的集合操作。除此之外，RDD还提供了诸如join、groupBy、reduceByKey等更为方便的操作（注意，reduceByKey是action，而非transformation），以支持常见的数据运算。RDD可以简单看成是一个数组。和普通数组的区别是，RDD中的数据是分区存储的，这样不同分区的数据就可以分布在不同的机器上，同时可以被并行处理。因此，Spark应用程序所做的就是把需要处理的数据转换为RDD，然后对RDD进行一系列的变换和操作从而得到结果。RDD的接口只支持粗粒度的操作，一个操作会被应用到RDD上所有的数据上。

## 2.1.2 RDD基本转换操作

- map操作将RDD中类型为T的元素一对一的映射为类型为U的元素

```scala
def map[U: ClassTag](f: T => U): RDD[U] = withScope {
  val cleanF = sc.clean(f)
  new MapPartitionsRDD[U, T](this, (context, pid, iter) => iter.map(cleanF))
}
```
举例

```shell
scala> var rdd = sc.makeRDD(1 to 5, 1)
rdd: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[1] at makeRDD at <console>:21
scala> rdd.collect()
res2: Array[Int] = Array(1, 2, 3, 4, 5)
scala> val mapRDD = rdd.map(x => x.toFloat)
mapRDD: org.apache.spark.rdd.RDD[Float] = MapPartitionsRDD[2] at map at <console>:23
scala> mapRDD.collect()
res3: Array[Float] = Array(1.0, 2.0, 3.0, 4.0, 5.0)
```

- distinct操作返回RDD中所有不一样的元素

```scala
def distinct(numPartitions: Int)(implicit ord: Ordering[T] = null): RDD[T] = withScope {
  map(x => (x, null)).reduceByKey((x, y) => x, numPartitions).map(_._1)
}
```

- flatMap操作将RDD中每一个元素进行一对多转换

```scala
def flatMap[U: ClassTag](f: T => TraversableOnce[U]): RDD[U] = withScope {
  val cleanF = sc.clean(f)
  new MapPartitionsRDD[U, T](this, (context, pid, iter) => iter.flatMap(cleanF))
}
```
  举例
```shell
scala> val flatMapRDD = rdd.flatMap(x => 1 to x)
flatMapRDD: org.apache.spark.rdd.RDD[Int] = MapPartitionsRDD[3] at flatMap at <console>:23
scala> flatMapRDD.collect()
res4: Array[Int] = Array(1, 1, 2, 1, 2, 3, 1, 2, 3, 4, 1, 2, 3, 4, 5)
```

- filter操作对RDD中的元素进行过滤

```scala
def filter(f: T => Boolean): RDD[T] = withScope {
  val cleanF = sc.clean(f)
  new MapPartitionsRDD[T, T](
    this,
    (context, pid, iter) => iter.filter(cleanF),
    preservesPartitioning = true)
}
```

## 2.1.3 键值RDD转换操作

- combineByKey、foldByKey、reduceByKey、groupByKey这四种操作类型，都是针对RDD[K, V]进行，最终会归结为combineByKey的操作上。
  combineByKey的内部实现分成三部分来完成：

1. 首先，根据是否需要在map端进行combine操作决定是否对RDD进行一次mapPartitions操作，从而减少shuffle的数据量。

2. 第2步根据partitioner函数对MapPartitionsRDD进行shuffle操作。

3. 最后对suffle的结果进行combine操作。


- join、leftOuterJoin、rightOuterJoin针对RDD[K, V]中K值相等的连接操作，最终归结为cogroup来实现。

```scala
def cogroup[W](other: RDD[(K, W)], partitioner: Partitioner)
    : RDD[(K, (Iterable[V], Iterable[W]))] = self.withScope {
  if (partitioner.isInstanceOf[HashPartitioner] && keyClass.isArray) {
    throw new SparkException("Default partitioner cannot partition array keys.")
  }
  val cg = new CoGroupedRDD[K](Seq(self, other), partitioner)
  cg.mapValues { case Array(vs, w1s) =>
    (vs.asInstanceOf[Iterable[V]], w1s.asInstanceOf[Iterable[W]])
  }
}
```

一个join操作会产生CoGroupRDD、MapValuesRDD、FlatMapValuesRDD三个RDD。

```scala
def join[W](other: RDD[(K, W)], partitioner: Partitioner): RDD[(K, (V, W))] = self.withScope {
  this.cogroup(other, partitioner).flatMapValues( pair =>
    for (v <- pair._1.iterator; w <- pair._2.iterator) yield (v, w)
  )
}
```

## 2.1.4 RDD依赖关系

因为RDD操作是粗粒度的，每一个转换操作都会产生一个新有RDD，所以前后的RDD就会形成前后依赖关系。Spark中有两种依赖类型，窄依赖（Narrow Dependenceis）和宽依赖（Wide Dependencies）。

- 窄依赖，每一个父RDD的分区最多只被子RDD的一个分区所依赖，map、filter、union操作就会形成一个窄依赖

- 宽依赖，多个子RDD的分区会依赖于同一个父RDD的分区。两个RDD数据集之间进行join操作就会形成宽依赖。


## 2.1.5 使用Spark实现PageRank算法

PageRank，网页排名，又称网页级别、Google左侧排名或佩奇排名，是一种由搜索引擎根据网页之间相互的超链接计算的技术，而作为网页排名的要素之一，以Google公司创办人拉里·佩奇（Larry Page）之姓来命名。Google用它来体现网页的相关性和重要性，在搜索引擎优化操作中是经常被用来评估网页优化的成效因素之一。

PageRank通过网络浩瀚的超链接关系来确定一个页面的等级。Google把从A页面到B页面的链接解释为A页面给B页面投票，Google根据投票来源（甚至来源的来源，即链接到A页面的页面）和投票目标的等级来决定新的等级。简单的说，一个高等级的页面可以使其他低等级页面的等级提升。[来源[维基百科](https://zh.wikipedia.org/wiki/PageRank)]

其算法原理如下：

1. 通过链接关系将网页构建成Web图，每个页面设置相同的PageRank值，初始一般为1。
2. 将每一个网页的权重贡献发送给相邻的网页，权重贡献=权重/指向的URL个数
3. 对天每一个网页，将收到的权重贡献相加记为contribs，重新计算权重rank=0.15 + contribs*0.85
4. 迭代1、2、3，直到收敛。

利用Spark的各种操作方法，可以方便的实现PageRank算法，实现如下：

```scala
val MAX_ITERATION = 20
val links = sc.parallelize(Array(('A', Array('D')), ('B', Array('A')), ('C', Array('A', 'B')), ('D', Array('A', 'C'))), 2).map(x => (x._1,x._2)).cache()
var ranks = sc.parallelize(Array(('A', 1.0), ('B', 1.0), ('C', 1.0), ('D', 1.0)), 2)
for (i <- 1 to MAX_ITERATION) {
  val contribs = links.join(ranks, 2).flatMap {
    case (url, (links, rank)) => links.map(dest => (dest, rank/links.size))
  }
  ranks = contribs.reduceByKey(_ + _, 2).mapValues(0.15 + 0.85 * _)
}
ranks.collect()
```

最终结果：

```shell
res15: Array[(Char, Double)] = Array((B,0.4613200524321036), (D,1.3705281840649928), (A,1.4357617405523626), (C,0.7323900229505396))
```

