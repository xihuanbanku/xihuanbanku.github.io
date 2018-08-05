---
layout: post
title: "spark 全局变量 accumulator的使用"
date: 2018-8-3 15:33:30 +0800
comments: true
---

## spark 程序中stage是如何划分的

又重新复习了一下`spark` 程序中`job, stage, task`之间的关系。首先明确基本概念:

> job由stage组成, stage由task 组成

以wordcount为例进行解析：

> sc.textFile("path/to/your/text/file").flatMap(_.split(" ")).map((_, 1)).reduceByKey(_+_).collect()

* 首先是`job`的划分

![job组成]({{ site.baseurl}}/images/2018-08-03-spark_wordcount_stages/Spark-jobs.png "wordcount job0")

从图中可以看到`wordcount`只划分了一个`Job0`, 在`spark`中, `job`的数量由该程序中的`action`算子决定,
也就是说在最后的`collect`算子触发了一个`job`, 如果没有`action`就不会触发`SparkContext`中的`runJob`方法

* `stage`的划分

![stage组成]({{ site.baseurl}}/images/2018-08-03-spark_wordcount_stages/Screen-Shot-2015-06-19-at-2.00.59-PM.png "wordcount stages")

`stage`的划分是根据`RDD`之间的宽窄依赖判断的, 如果`RDD`之间是宽依赖(后一个`RDD`的产生需要进行`shuffle`操作), 那么这里会产生一个
`stage`的分界

![宽窄依赖原理]({{ site.baseurl}}/images/2018-08-03-spark_wordcount_stages/spark-stages.jpg "宽窄依赖原理")

* Task的数量

根据类`DAGScheduler`中的`submitMissingTasks`方法可以知道，在`stage`中会为每个需要计算的`partition`生成一个`task`，
换句话说也就是每个`task`处理一个`partition`.

![task数量]({{ site.baseurl}}/images/2018-08-03-spark_wordcount_stages/submit_missingtask.png "task数量")

此外, 可以通过设置参数`spark.sql.shuffle.partitions`来指定`task`的数量

* Task的最大并行数

`task`的并行度是根据`cpu`的数量决定的. 当`task`被提交到`executor`之后, 会根据`executor`可用的`cpu`核数, 
决定一个`executor`中最多同时运行多少个`task`. 在类`TaskSchedulerImpl`的`resourceOfferSingleTaskSet`方法中, 
`CPUS_PER_TASK`的定义为`val CPUS_PER_TASK = conf.getInt("spark.task.cpus", 1)`, 也就是说默认情况下一个`task`对应`cpu`的一个核. 
如果一个`executor`可用`cpu`核数为8, 那么一个`executor`中最多同是并发执行8个`task`, 假如设置`spark.task.cpus`为2, 
那么同时就只能运行4个`task`.

![task并行度]({{ site.baseurl}}/images/2018-08-03-spark_wordcount_stages/task_parallelism.png "task并行度")

另外, `task`的并行度还与`spark.default.parallelism`参数有关

***

> 参考资料

1. [Spark executor中task的数量与最大并发数](https://www.jianshu.com/p/7c9b08a74de1)
2. [spark中tasks数量的设置](https://blog.csdn.net/mask1188/article/details/52013828)