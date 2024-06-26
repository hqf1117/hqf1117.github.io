##  ThreadLocal用法

ThreadLocal初始化：

```java
ThreadLocal<Integer> init = ThreadLocal.withInitial(()->0);
```

##  源码分析

### Thread,ThreadLocal,ThreadLocalMap的关系

![image-20220714214101400](https://github.com/hqf1117/hqf1117.github.io/assets/30412361/cb04a2ec-6916-489e-bd3f-1050ecab345a)


 ThreadLocalMap为k-v键值对，以ThreadLocal为key，为弱引用。

PS:

![image-20220714215703469](https://github.com/hqf1117/hqf1117.github.io/assets/30412361/b78f5c63-ba68-4524-8ce1-1f7e132b4122)


- 强引用

  当内存不足时，强引用的对象就算出现OOM也不会对该对象进行回收。

- 软引用

  SoftReference类实现，可以让对象豁免一些垃圾收集。当内存充足时不会回收，内存不足时会被回收。

- 弱引用

  WeakReference类实现，只要触发GC就会被回收。

- 虚引用

  1. 虚引用必须和ReferenceQueue联合使用。
  2. 虚引用的get方法总是返回null,==仅仅是提供了一种确保对象被finalize以后，做某些事情的通知机制==
  3. 设置虚引用的唯一目的，就是在这个对象被收集器回收的时候收到一个通知或者后续添加进一步的处理。

![image-20220714221637015](https://github.com/hqf1117/hqf1117.github.io/assets/30412361/2766e8aa-4038-49da-b921-ea39a83575bb)


### ThreadLocal为什么要使用弱引用？

![image-20220714222355953](https://github.com/hqf1117/hqf1117.github.io/assets/30412361/ab66108f-1aff-4d5d-9896-14a4890bedd7)


1. key使用弱引用可以大概率减少内存泄漏。
2. 复用线程的情况下，key作为弱引用被回收，ThreadLocalMap中就会出现key为null，值为强引用value的Entry永远无法回收，造成内存泄漏。
3. remove()方法会清理key为null的Entry。

ThreadLocal的最佳实践：

1. 建议先作初始化避免空指针。
2. ThreadLocal建议使用static,因为其没必要被多次初始化。
3. 用完需要手动remove。 

