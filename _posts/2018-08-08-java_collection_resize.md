---
layout: post
title: "java中常见集合的扩容"
date: 2018-8-8 11:27:09 +0800
comments: true
---

## java中ArrayList, LinkedList, HashMap的扩容

突然发现, 北京奥运会已经过去10年了...

很久没有复习`java`方面的知识, 整理一下集合扩容的几个知识点:

* ArrayList

`ArrayList`初始化大小是`10`（如果你知道你的`ArrayList`会达到多少容量，可以在初始化的时候就指定，能节省扩容的性能开支） 
扩容点规则是，新增的时候发现容量不够用了，就去扩容
扩容大小规则是，`扩容后的大小=原始大小+原始大小/2 + 1`。（例如：原始大小是`10`，扩容后的大小就是`10+5+1 = 16`）

源码如下：

```
public void ensureCapacity(int minCapacity) {
 modCount++;
 int oldCapacity = elementData.length;
 if (minCapacity > oldCapacity) {
     Object oldData[] = elementData;
     int newCapacity = (oldCapacity * 3)/2 + 1;
     if (newCapacity < minCapacity) newCapacity = minCapacity;
     // minCapacity is usually close to size, so this is a win:
     elementData = Arrays.copyOf(elementData, newCapacity);
 }
}
```

* LinkedList

`LinkedList` 是一个双向链表，没有初始化大小，也没有扩容的机制，就是一直在前面或者后面新增就好。 

* HashMap

`HashMap`初始化大小是`16`，扩容因子(`loadFactor`)默认`0.75`（可以指定初始化大小，和扩容因子） 
扩容规则: 当前大小和当前容量的比例超过了扩容因子，就会扩容，扩容大小*2。例如：初始大小为`16`，扩容因子`0.75` ，当容量为`12`的时候，比例已经是`0.75` 。触发扩容，扩容后的大小为`32`.)

源码如下：

```
/** 
  * Adds a new entry with the specified key, value and hash code to 
  * the specified bucket.  It is the responsibility of this 
  * method to resize the table if appropriate. 
  * 
  * Subclass overrides this to alter the behavior of put method. 
  */  
 void addEntry(int hash, K key, V value, int bucketIndex) {  
     Entry<K,V> e = table[bucketIndex];  
     table[bucketIndex] = new Entry<K,V>(hash, key, value, e);  
     if (size++ >= threshold)  
         resize(2 * table.length);  
 }  
 /** 
  * Rehashes the contents of this map into a new array with a 
  * larger capacity.  This method is called automatically when the 
  * number of keys in this map reaches its threshold. 
  * 
  * If current capacity is MAXIMUM_CAPACITY, this method does not 
  * resize the map, but sets threshold to Integer.MAX_VALUE. 
  * This has the effect of preventing future calls. 
  * 
  * @param newCapacity the new capacity, MUST be a power of two; 
  *  must be greater than current capacity unless current 
  *  capacity is MAXIMUM_CAPACITY (in which case value 
  *        is irrelevant). 
  */  
 void resize(int newCapacity) {  
     Entry[] oldTable = table;  
     int oldCapacity = oldTable.length;  
     if (oldCapacity == MAXIMUM_CAPACITY) {  
         threshold = Integer.MAX_VALUE;  
         return;  
     }  

     Entry[] newTable = new Entry[newCapacity];  
     transfer(newTable);  
     table = newTable;  
     threshold = (int)(newCapacity * loadFactor);  
 }  
```

***

> 参考资料

1. [HashMap与ArrayList 的扩容问题](https://blog.csdn.net/ffj0721/article/details/54312729)
1. [ArrayList,HashMap,LinkedList 初始化大小和 扩容机制](https://blog.csdn.net/walle167/article/details/78318779)