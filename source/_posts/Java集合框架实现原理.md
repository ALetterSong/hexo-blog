title: Java集合框架实现原理
date: 2016-01-12 13:25:05
tags:
---


> 大部分内容总结自博客园的[这篇文章][1]，访问作者的[博客][2]

### 一、Java集合框架

Java集合是java提供的工具包，包含了常用的数据结构：集合、链表、队列、栈、数组、映射等。Java集合工具包位置是`java.util.`*
Java集合主要可以划分为4个部分：List列表、Set集合、Map映射、工具类(Iterator迭代器、Enumeration枚举类、Arrays和Collections)、。
Java集合工具包框架图(如下)：
![Java集合][3]


**Collection**

![Collection1][4]

`Collection`是一个接口，是高度抽象出来的集合，它包含了集合的基本操作：添加、删除、清空、遍历(读取)、是否为空、获取大小、是否包含某元素等等。它主要的两个分支是：`List`和`Set`。
**List**是一个继承于`Collection`的接口，即`List`是集合中的一种。`List`是有序的队列，`List`中的每一个元素都有一个索引；第一个元素的索引值是0，往后的元素的索引值依次+1。和`Set`不同，`List`中允许有重复的元素。
**Set**是一个继承于`Collection`的接口，即`Set`也是集合中的一种。`Set`是没有重复元素的集合。
**Iterator**是一个接口，它是集合的迭代器。集合可以通过`Iterator`去遍历集合中的元素。`Iterator`提供的API接口，包括：是否存在下一个元素、获取下一个元素、删除当前元素。
**Map** 
`Map`是一个键值对(key-value)映射接口。`Map`映射中不能包含重复的键；每个键最多只能映射到一个值。
**AbstractMap**是继承于`Map`的抽象类，它实现了`Map`中的大部分API。其它`Map`的实现类可以通过继承`AbstractMap`来减少重复编码。

### 二、List实现分析

**ArrayList**

 - `ArrayList`是一个数组列表，相当于动态数组。与Java中的数组相比，它的容量能动态增长。它继承于`AbstractList`，实现了`List`、`RandomAccess`、`Cloneable`、`java.io.Serializable`这些接口。
 - `ArrayList`实际上是通过一个数组去保存数据的。当我们构造`ArrayList`时；若使用默认构造函数，则`ArrayList`的默认容量大小是10。
 - 当`ArrayList`容量不足以容纳全部元素时，`ArrayList`会重新设置容量：新的容量=“(原始容量x3)/2 + 1”。
 - 遍历方式
 
(1)通过迭代器遍历

    Integer value = null;
    Iterator iter = list.iterator();
    while (iter.hasNext()) {
        value = (Integer)iter.next();
    }
        
(2)随机访问，通过索引值去遍历

    Integer value = null;
    int size = list.size();
    for (int i=0; i<size; i++) {
        value = (Integer)list.get(i);        
    }
(3)for循环遍历

    Integer value = null;
    for (Integer integ:list) {
        value = integ;
    }
使用随机访问(即，通过索引序号访问)效率最高，而使用迭代器的效率最低。

> 和ArrayList不同，Vector中的操作是线程安全的，它的函数都是synchronized的，即都是支持同步的。


**LinkedList**
![链表][7]

 - `LinkedList`的本质是双向链表。
 - `LinkedList`继承于`AbstractSequentialList`，并且实现了Dequeue接口。
 - `LinkedList`包含两个重要的成员：`header`和`size`。
　　`header`是双向链表的表头，它是*双向链表节点*所对应的类Entry的实例。Entry中包含成员变量：previous,next,element。其中，`previous`是该节点的上一个节点，`next`是该节点的下一个节点，`element`是该节点所包含的值。 
　　`size`是双向链表中节点的个数。
 - `LinkedList`支持多种遍历方式。不要采用随机访问的方式去遍历`LinkedList`，而采用foreach逐个遍历的方式。
 - 对元素的插入、添加、移除操作，与`ArrayList`相比，`LinkedList` 更快，因为，当一个元素被添加到集合内部的任意位置时，`LinkedList` 不需要重新调整数组大小或者更新索引。
 
**List总结**

 - `ArrayList`是一个数组队列，相当于动态数组。它由数组实现，随机访问效率高，插入、删除效率低。
 - `LinkedList`是一个双向链表。它也可以被当作堆栈、队列或双端队列进行操作。`LinkedList`随机访问效率低，但插入、删除效率高。
 - `Vector`是矢量队列，和`ArrayList`一样，它也是一个动态数组，由数组实现。但是`ArrayList`是非线程安全的，而`Vector`是线程安全的。
 - `Stack`是栈，它继承于`Vector`。它的特性是：先进后出(FILO, First In Last Out)。
 - 对于需要快速插入，删除元素，应该使用`LinkedList`。因为通过`add(int index, E element)`向`LinkedList`插入元素时。先是在双向链表中找到要插入节点的位置`index`；找到之后，再插入一个新节点。双向链表查找`index`位置的节点时，有一个加速动作：若`index`<双向链表长度的1/2，则从前向后查找;否则，从后向前查找。而在`ArrayList`中，`System.arraycopy`较为耗时，它会移动index之后所有元素即可。所以,`ArrayList`的`add(int index, E element)`函数，会引起index之后所有元素的改变。
 - 对于需要快速随机访问元素，应该使用`ArrayList`。因为`ArrayList`的`get(int index)`获取`ArrayList`第`index`个元素时。直接返回数组中index位置的元素，而不需要像`LinkedList`一样进行查找。
 - 对于单线程环境或者多线程环境，但List仅仅只会被单个线程操作，此时应该使用非同步的类(如ArrayList)。

### 三、Map实现分析
**HashMap**
![hashmap][8]

 - `HashMap`基于哈希表的`Map`接口的实现。，它存储的内容是键值对(key-value)映射。
 - `HashMap`继承于`AbstractMap`，实现了`Map`、`Cloneable`、`java.io.Serializable`接口。
 - `HashMap`实际上是一个“链表散列”的数据结构，即数组和链表的结合体。
 - `HashMap`的实现不是同步的，这意味着它不是线程安全的。它的key、value都可以为`null`。此外，`HashMap`中的映射不是有序的。
 - `HashMap`是通过"拉链法"实现的哈希表。它包括几个重要的成员变量：
　　`table`是一个Entry[]数组类型，而Entry实际上就是一个单向链表。哈希表的"key-value键值对"都是存储在Entry数组中的。 
　　`size`是HashMap的大小，它是HashMap保存的键值对的数量。 
　　`threshold`是HashMap的阈值，用于判断是否需要调整HashMap的容量。threshold的值="容量*加载因子"，当HashMap中存储数据的数量达到threshold时，就需要将HashMap的容量加倍。
　　`loadFactor`就是加载因子。 
　　`modCount`是用来实现fail-fast机制的。
 - 遍历方式

        /*
         * 通过entry set遍历HashMap
         * 效率高!
         */
        private static void iteratorHashMapByEntryset(HashMap map){
            if (map == null)
                return ;
            System.out.println("\niterator HashMap By entryset");
            String key = null;
            Integer integ = null;
            Iterator iter = map.entrySet().iterator();
            while(iter.hasNext()) {
                Map.Entry entry = (Map.Entry)iter.next();
                key = (String)entry.getKey();
                integ = (Integer)entry.getValue();
                System.out.println(key+" -- "+integ.intValue());
            }
        }
    
        /*
         * 通过keyset来遍历HashMap
         * 效率低!
         */
        private static void iteratorHashMapByKeyset(HashMap map) {
            if (map == null)
                return ;
            System.out.println("\niterator HashMap By keyset");
            String key = null;
            Integer integ = null;
            Iterator iter = map.keySet().iterator();
            while (iter.hasNext()) {
                key = (String)iter.next();
                integ = (Integer)map.get(key);
                System.out.println(key+" -- "+integ.intValue());
            }
        }
        
        /*
         * 遍历HashMap的values
         */
        private static void iteratorHashMapJustValues(HashMap map) {
            if (map == null)
                return ;
            Collection c = map.values();
            Iterator iter= c.iterator();
            while (iter.hasNext()) {
                System.out.println(iter.next());
           }
        }
**Map总结**

 - `HashMap`是基于“拉链法”实现的散列表。一般用于单线程程序中。对`HashMap`的同步处理可以使用`Collections`类提供的`synchronizedMap`静态方法。
 - `HashMap`的`key`、`value`都可以为`null`。当`HashMap`的`key`为`null`时，`HashMap`会将其固定的插入`table[0]`位置(即`HashMap`散列表的第一个位置)；而且`table[0]`处只会容纳一个`key`为`null`的值，当有多个`key`为`null`的值插入的时候，`table[0]`会保留最后插入的`value`。
 - `Hashtable`也是基于“拉链法”实现的散列表。`Hashtable`的几乎所有函数都是同步的，即它是线程安全的，支持多线程。
 - `Hashtable`的`key`、`value`都不可以为`null`，否则，会抛出异常`NullPointerException`。
 - `WeakHashMap`也是基于“拉链法”实现的散列表，它一般也用于单线程程序中。相比`HashMap`，`WeakHashMap`中的键是“弱键”，当“弱键”被GC回收时，它对应的键值对也会被从`WeakHashMap`中删除；而`HashMap`中的键是强键。
 - `TreeMap`是有序的散列表，它是通过红黑树实现的。它一般用于单线程中存储有序的映射。


  [1]: http://www.cnblogs.com/skywang12345/p/3323085.html
  [2]: http://www.cnblogs.com/skywang12345/category/455711.html
  [3]: http://images.cnitblog.com/blog/497634/201309/08171028-a5e372741b18431591bb577b1e1c95e6.jpg
  [4]: http://1.bigggge.sinaapp.com/images/3751779964676785446.png
  [7]: http://pic002.cnblogs.com/images/2012/381041/2012092516312023.gif
  [8]: http://1.bigggge.sinaapp.com/images/7d17f3cct79999972a70e&690.png