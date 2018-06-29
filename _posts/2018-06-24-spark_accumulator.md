---
layout: post
title: "spark 全局变量 accumulator的使用"
date: 2018-6-24 21:29:50 +0800
comments: true
---

## spark 全局变量 accumulator的使用

今天下午使用spark sql 操作 DataFrame，forEachPartition 入库的时候, 因为需要动态的生成ID一个数据库表中。
因为多个partition的操作， 所以变量不能直接共享， 就想到了 全局变量accumulator, 代码如下
```
    val conf = new SparkConf().setMaster("local[*]").setAppName("accumulo")
    val sc = new SparkContext(conf)

    var accum = sc.longAccumulator
    accum.add(624000) // 断点1

    sc.textFile("d:/log.txt", 5).foreachPartition(x => {

      accum.add(1) //断点2
      println(accum.value) //输出为 1
    })
    println(accum.value)//结果 624005
```

但是在使用debug的时候, 发现虽然设置了初始值624000, 但是在 foreachPartition 中的accum 在断点2处查看结果是0, add以后是1, 且hashcode也与刚初始化时的不一样,
最后的结果却是正确的。
后来查找答案, 有一个很重要的前提. accumulator的使用场景适合于对各个partition的累加计数，也就是说想要在 foreachPartition
处理的过程中使用accum是看不到正确结果的, **在tast中只能对accumulator进行操作, 但是不能读取它的值. 只有driver端才可以读取.** 而且只有最后触发了action操作, 才会进行accumulator的累加, 否则也不会有结果
