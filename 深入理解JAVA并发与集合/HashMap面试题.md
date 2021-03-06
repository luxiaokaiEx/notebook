- [HashMap面试题](#HashMap面试题)
  - [JDK7与JDK8中HashMap实现原理上的不同点](#JDK7与JDK8中HashMap实现原理上的不同点)
  - [JDK8中HashMap数组长度为何是2的幂次方](#JDK8中HashMap数组长度为何是2的幂次方)
  - [JDK8中HashMap数组什么时候扩容](#JDK8中HashMap数组什么时候扩容)
  - [JDK8中HashMap为什么要使用红黑树](#JDK8中HashMap为什么要使用红黑树)
  - [JDK8中HashMap什么时候将链表转化为红黑树](#JDK8中HashMap什么时候将链表转化为红黑树)
  - [JDK8中HashMap及TreeMap的红黑树实现原理](#JDK8中HashMap及TreeMap的红黑树实现原理)
  - [JDK7中HashMap2个线程resize时循环链表问题](#JDK7中HashMap2个线程resize时循环链表问题)
  - [JDK8中HashMap2个线程同时put会发生什么](#JDK8中HashMap2个线程同时put会发生什么)
  - [JDK8中HashMap1个线程put1个线程迭代器遍历会发生什么](#JDK8中HashMap1个线程put1个线程迭代器遍历会发生什么)
  - [JDK7与JDK8中HashMap的快速失败机制](#JDK7与JDK8中HashMap的快速失败机制)
  - [JDK7中ConcurrentHashMap与HashMap实现原理上的不同点](#JDK7中ConcurrentHashMap与HashMap实现原理上的不同点)
  - [JDK8中ConcurrentHashMap与HashMap实现原理上的不同点](#JDK8中ConcurrentHashMap与HashMap实现原理上的不同点)
  - [JDK7与JDK8中ConcurrentHashMap保证线程安全实现原理上的不同点](#JDK7与JDK8中ConcurrentHashMap保证线程安全实现原理上的不同点)
  - [JDK8中ConcurrentHashMap为何不使用JDK7中的分段锁思想了](#JDK8中ConcurrentHashMap为何不使用JDK7中的分段锁思想了)
  - [JDK7与JDK8中ConcurrentHashMap的安全失败机制](#JDK7与JDK8中ConcurrentHashMap的安全失败机制)
  - [JDK8中Hashtable与HashMap实现原理上的不同点](#JDK8中Hashtable与HashMap实现原理上的不同点)
  - [JDK8中LinkedHashMap与HashMap实现原理上的不同点](#JDK8中LinkedHashMap与HashMap实现原理上的不同点)
  
# HashMap面试题

参考
- [HashMap源码分析](应用场景与源码分析/HashMap.md)
- [JDK7.ConcurrentHashMap源码分析](应用场景与源码分析/JDK7.ConcurrentHashMap.md)
- [JDK8.ConcurrentHashMap源码分析](应用场景与源码分析/JDK8.ConcurrentHashMap.md)
- [面试问题记录汇总](https://github.com/peteryuanpan/notebook/issues/85)
- [Java集合框架常见面试题](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/collection/Java%E9%9B%86%E5%90%88%E6%A1%86%E6%9E%B6%E5%B8%B8%E8%A7%81%E9%9D%A2%E8%AF%95%E9%A2%98.md)
- [HashMap与ConcurrentHashMap面试要点](https://www.yuque.com/books/share/9f4576fb-9aa9-4965-abf3-b3a36433faa6/doh8wb)
- [HashMap相关面试题](https://www.yuque.com/books/share/9f4576fb-9aa9-4965-abf3-b3a36433faa6/hv4o3e)
- [ConcurrentHashMap相关面试题](https://www.yuque.com/books/share/9f4576fb-9aa9-4965-abf3-b3a36433faa6/biiid7)
- [HashMap的7种遍历方式与性能分析](https://mp.weixin.qq.com/s/Zz6mofCtmYpABDL1ap04ow)
- [简书：为什么HashMap线程不安全](https://www.jianshu.com/p/e2f75c8cce01)
- [掘金：快速失败机制&失败安全机制](https://juejin.im/post/6844904046617182215)

### 前提声明（重要！！）

本文内容是面试题，因此总结的比较简要，如果是第一次学习HashMap或ConcurrentHashMap的朋友，不要细钻下面的面试题，你直接看肯定是不明不白的，而应该先去看这三份源码分析：[HashMap源码分析](应用场景与源码分析/HashMap.md)、[JDK7.ConcurrentHashMap源码分析](应用场景与源码分析/JDK7.ConcurrentHashMap.md)、[JDK8.ConcurrentHashMap源码分析](应用场景与源码分析/JDK8.ConcurrentHashMap.md)。这里面包含了应用场景、类图、数据结构、关键方法的分析，有详细的注释帮助理解，如果还是看不明白，说明我表达的不够易懂，你可以去看看一看其它博主的文章结合一起理解

另外要说明的是，我更希望的是通过 源码分析 + 面试题，引导读者对HashMap及ConcurrentHashMap有一个基本的概念理解，而不是把所有的理解都灌输出来，因为我认为，对这种数据结构的理解，你必须自己去分析源码（最好能对着源码写一写注释），才能将代码的理解印刻在脑海中，而至于印刻得多深刻、多快能掌握，取决你数据结构与算法的功底了

我比较喜欢的方式是，通过关键成员属性（比如Entry数组、Segment数组） + 关键方法（比如构造方法、put方法），理解这个数据结构的核心思想，然后去写完源码分析文章，最后能够做到自己在纸上演练一遍，说清楚数据结构的原理且基本无误，就OK了，至于很细节的东西，没有必要去强记，而一些关键的点（尤其是对比不同点），我以下面面试题的形式记录出来，帮助复习回忆

### JDK7与JDK8中HashMap实现原理上的不同点

不同点，从几个层面来看
- 底层数据结构：JDK7是链表数组，JDK8是链表数组 + 红黑树
- Hash复杂度：JDK7Hash算法更复杂，Hash更散列，JDK8有红黑树保障，Hash算法更简单，计算消耗更小
- 链表插入法：JDK7是头插法（多线程扩容的情况下使用头插法会出现循环链表的问题），JDK8是尾插法
- 扩容：JDK7是将旧链表元素一个个地通过重新计算index转移到新链表上，JDK8是将每个旧链表（或红黑树）分为两个链表（或红黑树），有的元素插入新数组的低位链表（或红黑树），有的元素插入新数组的高位链表（或红黑树）
- 重新哈希：JDK7扩容时可能重新对所有key进行rehash（与哈希种子有关），JDK8无这样的逻辑
- put覆盖策略：JDK8多了一个方法 putIfAbsent(key,value)，允许put时不覆盖已存在节点

### JDK8中HashMap数组长度为何是2的幂次方

JDK7中也一样，数组长度n设计为2的幂次方，这样index=(n-1)&hash，就相当于index=hash%n，主要是降低了计算消耗，其次让数组扩容过程变得简单

每次扩容数组长度增加1倍，在JDK8的实现中利用了这个特点，扩容时将一个链表（或红黑树）分为两个链表（或红黑树）进行插入

值得说明的是，JDK7.HashMap扩容时会进行重新hash，而JDK8.HashMap、JDK7.ConcurrentHashMap、JDK8.ConcurrentHashMap在扩容时都不会重新hash，这样的话扩容就会呈现出特点了，这个特点是与每次扩容时让数组长度扩大2倍有关系的

不进行重新hash的扩容特点
- 每个旧数组的元素在新数组的位置，要不然等于旧数组位置，要不然等于旧数组位置加上旧数组长度
- 每个旧数组的两个位置上的元素一定不会出现在新数组的相同位置上

### JDK8中HashMap数组什么时候扩容

JDK7中也一样，一般是当 size > threshold 时进行扩容，size是节点个数，threshold是扩容阈值

一般情况下，threshold = capacity * loadFactor，capacity是数组长度，loadFactor是扩容引子

### JDK8中HashMap为什么要使用红黑树

JDK7中数组+链表的实现方式，可能造成一个链表过长，而查询效率低下，JDK8中链表会在特定条件下转为红黑树，提高了查询效率

### JDK8中HashMap什么时候将链表转化为红黑树

这个题很容易答错，大部分答案就是：当链表中的元素个数大于等于8时就会把链表转化为红黑树

但是其实还有另外一个限制：当发现链表中的元素个数大于等于8之后，还会判断一下当前数组的长度，如果数组长度小于64时，此时并不会转化为红黑树，而是进行扩容，扩容的原因是，如果数组长度还比较小，就先利用扩容来缩小链表的长度

因此，正确的答案是：**当链表中的元素个数大于等于8，并且数组的长度大于等于64时才会将链表转为红黑树**

### JDK8中HashMap及TreeMap的红黑树实现原理

TODO

如何理解红黑树内部维护了一个双向链表?

### JDK7中HashMap2个线程resize时循环链表问题

关键在于JDK7中HashMap使用的是头插法，会出现循环链表，而JDK8中HashMap使用的是尾插法，则不会出现

来看下JDK7中resize时的代码

```java
    void resize(int newCapacity) {
        Entry[] oldTable = table;
        int oldCapacity = oldTable.length;
        if (oldCapacity == MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return;
        }

        Entry[] newTable = new Entry[newCapacity];
        transfer(newTable, initHashSeedAsNeeded(newCapacity));
        table = newTable;
        threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
    }
    void transfer(Entry[] newTable, boolean rehash) {
        int newCapacity = newTable.length;
        for (Entry<K,V> e : table) {
            while(null != e) {
                Entry<K,V> next = e.next;
                if (rehash) {
                    e.hash = null == e.key ? 0 : hash(e.key);
                }
                int i = indexFor(e.hash, newCapacity);
                e.next = newTable[i];
                newTable[i] = e;
                e = next;
            }
        }
    }
```

会存在以下的情况

线程1执行了resize，执行到了 next = e.next;

此时，旧数组index=1位置中，存在一个链表 |->a->b，新数组还是空，然后线程1让出CPU时间片

```
旧数组     ====   新数组
线程1的初始状态
1|->a->b  ====>  |(EMPTY)
```

线程2也执行了resize，并执行完了transfer（这种情况是可能发生的，CPU执行的太快，存在不确定性）

此时，新数组中生成了新的链表，由于采用的头插法（这是关键），旧数组中链表 |->a->b 变成了 |->b->a，然后线程2让出CPU时间片

```
旧数组     ====   新数组
线程2的初始状态
1|->a->b   ====  |(EMPTY)
线程2执行两次头插法后的状态
1|(EMPTY)  ====  |->b->a
```

线程1继续执行，注意e、next都是局部变量，属于线程工作内存私有的，会存在从CPU缓存中获取数据而不直接从主内存获取数据的情况（第一次除外）

即使线程2中旧数组index=1位置的链表已经是(EMPTY)了，线程1中旧数组的链表仍可能保持原样，而新数组的链表可能会是从主内存获取的数据

此时，旧数组链表的元素a通过头插法插入新数组中，就可能会形成 |->a->b->a 这样的循环链表

```
旧数组    ====   新数组
线程1的状态（线程2执行完）
1|->a->b  ====  |->b->a
线程1执行一次头插法后的状态
1|->b     ====  |->a->b->a
```

在上面的解释中，我用了许多“可能”，这是由于并发中的不确定性导致的，一个线程执行多少行指令而让出CPU时间片，从CPU缓存还是主内存中读取数据，这些都无法有一个非常确定的结论

甚至会让人认为，那么多凑巧的情况真的会出现吗? 那我们不妨测试一下

```java
package hashmap;

import java.util.HashMap;
import java.util.Map;

public class HashMapLinkedLoopProblemTest {

    static final int N = 100000;
    static String[] str_arr = new String[N];
    static Map<String, String> map = new HashMap<>();

    static {
        for (int i = 0; i < N; i ++)
            str_arr[i] = String.valueOf(i);
    }

    public static void main(String[] args) {
        Thread t = new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < N; i ++) {
                    System.out.println(Thread.currentThread().getName() + " " + i);
                    map.put(str_arr[i], str_arr[i]);
                }
            }
        });
        Thread a = new Thread(t, "ThreadA");
        Thread b = new Thread(t, "ThreadB");
        a.start();
        b.start();
        try {
            a.join();
            b.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " end");
    }
}
```

输出结果（JDK7下运行，偶现，但复现率很高）
```
ThreadA 27034
ThreadA 27035
ThreadA 27036
...
ThreadB 21288
ThreadB 21289
ThreadB 21290
...
（不输出了，但程序在运行）
```

用jps查看进程号pid，再执行jstack pid，可以看到两个线程都在 at java.util.HashMap.put(HashMap.java:494) 这一行死循环了
```
"ThreadB" prio=6 tid=0x0000000012597800 nid=0x2fc0 runnable [0x0000000012ebf000]
   java.lang.Thread.State: RUNNABLE
        at java.util.HashMap.put(HashMap.java:494)
        at hashmap.HashMapLinkedLoopProblemTest$1.run(HashMapLinkedLoopProblemTest.java:23)
        at java.lang.Thread.run(Thread.java:745)
        at java.lang.Thread.run(Thread.java:745)

"ThreadA" prio=6 tid=0x0000000012597000 nid=0x2318 runnable [0x0000000012dbf000]
   java.lang.Thread.State: RUNNABLE
        at java.util.HashMap.put(HashMap.java:494)
        at hashmap.HashMapLinkedLoopProblemTest$1.run(HashMapLinkedLoopProblemTest.java:23)
        at java.lang.Thread.run(Thread.java:745)
        at java.lang.Thread.run(Thread.java:745)
```

查看HashMap.java:494行代码，如下图，可以看出是在for循环一直运行

加个断点后，可以看出e所表示的key->value一直是2个数字，比如21619->21619、15239->15239、21619->21619、15239->15239、...，但挺可惜，不知为何，IDEA这里debug不出e.next的信息

![image](https://user-images.githubusercontent.com/10209135/98442724-5eae3c80-2141-11eb-988b-4770557ae816.png)

这样就完了吗? 当然没有，上面的分析并没有确定一定是resize之后出现的循环链表，不妨将测试例子修改一下

```java
public class HashMapLinkedLoopProblemTest {

    static final int N = 100000;
    static String[] str_arr = new String[N];
    static Map<String, String> map = new HashMap<>(200000); // 关键!
...
```

输出结果（无论执行多少次）
```
main end
```

当把HashMap数组初始化为20万长度时，就不能复现出死循环的问题了，这是由于HashMap在这段代码中没有执行过resize方法

HashMap扩容（resize）的条件是 size > threshold，而 threshold = capacity * loadFactor，capacity是数组长度即20万，loadFactor是扩容引子默认是0.75，size是节点个数最多是10万，10万 < 20万 * 0.75，因此永远不会执行resize方法，这样我们就能证明是resize之后出现的循环链表，使得put方法中的for循环一直在执行了

总结一下，JDK7出现循环链表的关键有两点：一是采用了头插法，二是多个线程操作并执行了resize。而JDK8中使用的是尾插法，尾插法最多是使得一个线程多执行了一次另一个线程的插入活，就不会出现循环链表了

### JDK8中HashMap2个线程同时put会发生什么

会出现总元素个数变少的情况

来看下测试例子

```java
package hashmap;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.CountDownLatch;

public class HashMapConcurrencyTest2 {

    private static Map<String, String> map = new HashMap<>();
    //private static Map<String, String> map = new ConcurrentHashMap<>();
    private static CountDownLatch countDownLatch;
    private static final int threadNum = 2;

    public static void main(String[] args) {
        Thread a = new Thread(() -> {
            try {
                for (int i = 0; i < 100000; i ++) {
                    String s = Thread.currentThread().getName() + ": " + i;
                    map.put(s, s);
                }
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                countDownLatch.countDown();
            }
        });

        countDownLatch = new CountDownLatch(threadNum);

        for (int i = 0; i < threadNum; i ++) {
            Thread t = new Thread(a, "Thread" + i);
            t.start();
        }

        try {
            countDownLatch.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println(map.size());
    }
}
```

输出结果（偶现，但复现率很高）
```
196774
```

改一下测试例子
```java
    //private static Map<String, String> map = new HashMap<>();
    private static Map<String, String> map = new ConcurrentHashMap<>();
```

输出结果（无论执行多少次）
```
200000
```

用了2个线程分别向HashMap插入10万个元素，按理说结果应该是20万，可有很大概率能复现出小于20万

这是因为在HashMap.put时，如果发生了Hash碰撞，需要在一个链表上插入元素（可以反推，如果不发生Hash碰撞，总元素个数是不会变少的）

当两个线程同时在一个数组位置的链表上插入元素时，可能会出现下面的情况，这样总元素个数就变少了

```
线程1：a -> b，让出CPU时间片
线程2：a -> c，让出CPU时间片
线程1：a -> c -> d，这样 b 就丢失了
```

解决方案有多种：synchonized、ReentrantLock、ConcurrentHashMap（推荐）

### JDK8中HashMap1个线程put1个线程迭代器遍历会发生什么

会触发快速失败机制，抛 java.util.ConcurrentModificationException 异常

来看下测试例子

```java
package hashmap;

import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

public class HashMapConcurrencyTest3 {

    private static Map<String, String> map = new HashMap<>();
    //private static Map<String, String> map = new ConcurrentHashMap<>();

    public static void main(String[] args) {
        Thread a = new Thread(() -> {
            for (int i = 0; i < 1000000; i ++) {
                map.put(String.valueOf(i), String.valueOf(i));
            }
        });

        Thread b = new Thread(() -> {
            Iterator i = map.keySet().iterator();
            while (i.hasNext()) {
                i.next();
            }
        });

        a.start();
        b.start();

        try {
            a.join();
            b.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println(Thread.currentThread().getName() + " end");
    }
}
```

输出结果
```
Exception in thread "Thread-1" java.util.ConcurrentModificationException
	at java.util.HashMap$HashIterator.nextNode(HashMap.java:1445)
	at java.util.HashMap$KeyIterator.next(HashMap.java:1469)
	at hashmap.HashMapConcurrencyTest3.lambda$main$1(HashMapConcurrencyTest3.java:23)
	at java.lang.Thread.run(Thread.java:748)
main end
```

改一下测试例子
```java
    //private static Map<String, String> map = new HashMap<>();
    private static Map<String, String> map = new ConcurrentHashMap<>();
```

输出结果
```
main end
```

见下方解释

### JDK7与JDK8中HashMap的快速失败机制

快速失败机制是HashMap中对数据安全和线程安全的一种防范机制，在使用迭代器进行遍历、删除 或者 forEach等操作时，会进行 modCount != expectedModCount 校验，若 false，则会抛 ConcurrentModificationException。在JDK7与JDK8的HashMap源码中，只有两处对 expectedModCount 进行了 expectedModCount = modCount 赋值，一处是HashIterator的构造方法，一处是HashIterator的remove方法。因此多线程下，一个线程在迭代器遍历，另一个线程在remove或者put操作（会modCount++），会触发快速失败机制

在上面的例子中，我们看到了 JDK8中HashMap1个线程put1个线程迭代器遍历，会抛 java.util.ConcurrentModificationException 异常，来看 at java.util.HashMap$HashIterator.nextNode(HashMap.java:1445)

```java
    abstract class HashIterator {
        final Node<K,V> nextNode() {
            Node<K,V>[] t;
            Node<K,V> e = next;
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            if (e == null)
                throw new NoSuchElementException();
            if ((next = (current = e).next) == null && (t = table) != null) {
                do {} while (index < t.length && (next = t[index++]) == null);
            }
            return e;
        }
```

可以看出，当 modCount != expectedModCount 时，就会抛此异常，这样的判断在 JDK7 及 JDK8 中的 HashMap 十分常见，它的目的是保证数据安全，不要出现这种情况

其实快速失败机制不仅仅是多线程下会出现，在单线程下也会遇到，来看一个例子

```java
package hashmap;

import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;

public class HashMapConcurrentModificationExceptionTest1 {

    public static void main(String[] args) {
        Map<String ,String> map = new HashMap<>();
        map.put("1", "1");
        map.put("2", "2");
        Iterator i = map.keySet().iterator();
        while (i.hasNext()) {
            String key = (String) i.next();
            if ("1".equals(key)) {
                map.remove(key);
                //i.remove();
            }
        }
    }
    System.out.println("end");
}
```

输出结果
```
Exception in thread "main" java.util.ConcurrentModificationException
	at java.util.HashMap$HashIterator.remove(HashMap.java:1459)
	at hashmap.HashMapConcurrentModificationExceptionTest1.main(HashMapConcurrentModificationExceptionTest1.java:18)
```

修改一下测试例子
```java
                //map.remove(key);
                i.remove();
```

输出结果
```
end
```

在单线程下，HashMap边迭代边删除需要使用迭代器（Iterator）来进行删除，下面是源码，可以看出在 i.remove() 最后会执行 expectedModCount = modCount; 将 expectedModCount 修正

```java
    abstract class HashIterator {
        public final void remove() {
            Node<K,V> p = current;
            if (p == null)
                throw new IllegalStateException();
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            current = null;
            K key = p.key;
            removeNode(hash(key), key, null, false, false);
            expectedModCount = modCount;
        }
```

### JDK7中ConcurrentHashMap与HashMap实现原理上的不同点

TODO

### JDK8中ConcurrentHashMap与HashMap实现原理上的不同点

TODO

### JDK7与JDK8中ConcurrentHashMap保证线程安全实现原理上的不同点

从不同方面来看

要涉及到扩容

### JDK8中ConcurrentHashMap为何不使用JDK7中的分段锁思想了

TODO

### JDK7与JDK8中ConcurrentHashMap的安全失败机制

TODO

JDK8中ConcurrentHashMap的CounterCell?

### JDK8中Hashtable与HashMap实现原理上的不同点

### JDK8中LinkedHashMap与HashMap实现原理上的不同点
