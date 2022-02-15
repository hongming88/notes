

# ThreadLocal

==作用就是**让线程自己独立保存一份自己的变量副本**。每个线程独立使用自己的变量副本， 不会影响到其他线程。==

我愿称其为线程域

这个变量一直存在这个线程里





用户连接项目后就开启一个线程直到用户断开连接



## 运用：

1. 事务：

    **开启事务的注意点:**

   - 为了保证所有的操作在一个事务中,案例中使用的连接必须是同一个: ==service层开启事务的connection需要跟dao层访问数据库的connection保持一致==
   - 线程并发情况下, 每个线程只能操作各自的 connection

   

   注意了，每一次访问代表一个线程，若没用ThreadLocal的时候，多个浏览器可是共用一个数据库连接的，那会出现问题，比如说一个浏览器提交完之后顺带把其他客户端的还没执行完的事物提交了（用 ThreadLocal 保存 connection）

2. 获取用户信息

用户连接后保存用户到ThreadLocal，需要的时候就从ThreadLocal取。

- ThreadLocal可以把用户信息保存在线程中，用户发来的每一次请求启动的线程到保存了用户信息，当请求结束，我们会把保存的用户信息清除掉，这样就方便我们在开发中获取用户登录信息
- 实现思路：
  - 我们需要创建一个ThreadLocal类，创建一个ThreadLocal对象，设置ThreadLocal的set，remove，get方法，
  - 我们定义一个登录的拦截器类，实现HandlerInterceptor ，重写 preHandle() 和afterCompletion()方法 ，preHandle ()方法把登录信息写入ThreadLocal，afterCompletion()方法清除登录信息
  - 我们需要设置一些配置信息，创建一个类实现 WebMvcConfigurer ，重写addInterceptors()方法，创建一个登录拦截器类的对象，给他添加到配置中，我们就实现了ThreadLocal保存用户信息
    

## 内部

1、每个Thread对象内部都维护了一个ThreadLocalMap这样一个ThreadLocal的Map，可以存放若干个ThreadLocal。

```java
/* ThreadLocal values pertaining to this thread. This map is maintained
 * by the ThreadLocal class. */
ThreadLocal.ThreadLocalMap threadLocals = null;
```

　2、当我们在调用get()方法的时候，先获取当前线程，然后获取到当前线程的`ThreadLocalMap`对象，如果非空，那么取出`ThreadLocal的value`，否则进行初始化，初始化就是将initialValue的值set到ThreadLocal中。

```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null)
            return (T)e.value;
    }
    return setInitialValue();
}
```

## 弱引用和内存泄漏
 有些程序员在使用ThreadLocal的过程中会发现有内存泄漏的情况发生，就猜测这个内存泄漏跟Entry中使用了弱引用的key有关系。这个理解其实是不对的。

 我们先来回顾这个问题中涉及的几个名词概念，再来分析问题。

（1） 内存泄漏相关概念

- Memory overflow:内存溢出，没有足够的内存提供申请者使用。
- Memory leak: 内存泄漏是指程序中已动态分配的堆内存由于某种原因程序未释放或无法释放，造成系统内存的浪费，导致程序运行速度减慢甚至系统崩溃等严重后果。内存泄漏的堆积终将导致内存溢出。

（2） 弱引用相关概念

 Java中的引用有4种类型： 强、软、弱、虚。当前这个问题主要涉及到强引用和弱引用：

 **强引用（“Strong” Reference）**，就是我们最常见的普通对象引用，只要还有强引用指向一个对象，就能表明对象还“活着”，垃圾回收器就不会回收这种对象。

 **弱引用（WeakReference）**，垃圾回收器一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。

**（3） 如果key使用强引用**

 假设ThreadLocalMap中的key使用了强引用，那么会出现内存泄漏吗？

 此时ThreadLocal的内存图（实线表示强引用）如下：



![](\pics\Strong.png)

 假设在业务代码中使用完ThreadLocal ，threadLocal Ref被回收了。

 但是因为threadLocalMap的Entry强引用了threadLocal，造成threadLocal无法被回收。

 在没有手动删除这个Entry以及CurrentThread依然运行的前提下，始终有强引用链` threadRef->currentThread->threadLocalMap->entry`，Entry就不会被回收（Entry中包括了ThreadLocal实例和value），导致Entry内存泄漏。

 也就是说，ThreadLocalMap中的key使用了强引用， 是无法完全避免内存泄漏的。


**（5）如果key使用弱引用**

 那么ThreadLocalMap中的key使用了弱引用，会出现内存泄漏吗？

 此时ThreadLocal的内存图（实线表示强引用，虚线表示弱引用）如下：

![](\pics\weak.png)

 同样假设在业务代码中使用完ThreadLocal ，threadLocal Ref被回收了。

 由于ThreadLocalMap只持有ThreadLocal的弱引用，没有任何强引用指向threadlocal实例, 所以threadlocal就可以顺利被gc回收，此时Entry中的key=null。

 但是在没有手动删除这个Entry以及CurrentThread依然运行的前提下，也存在有强引用链 threadRef->currentThread->threadLocalMap->entry -> value ，value不会被回收， 而这块value永远不会被访问到了，导致value内存泄漏。

 也就是说，ThreadLocalMap中的key使用了弱引用， 也有可能内存泄漏。

**（6）出现内存泄漏的真实原因**

 比较以上两种情况，我们就会发现，内存泄漏的发生跟ThreadLocalMap中的key是否使用弱引用是没有关系的。那么内存泄漏的的真正原因是什么呢？

 细心的同学会发现，在以上两种内存泄漏的情况中，都有两个前提：

>1. 没有手动删除这个Entry
>2. CurrentThread依然运行



第一点很好理解，只要在使用完ThreadLocal，调用其remove方法删除对应的Entry，就能避免内存泄漏。

 第二点稍微复杂一点， 由于ThreadLocalMap是Thread的一个属性，被当前线程所引用，所以它的生命周期跟Thread一样长。那么在使用完ThreadLocal之后，如果当前Thread也随之执行结束，ThreadLocalMap自然也会被gc回收，从根源上避免了内存泄漏。

 综上，**ThreadLocal内存泄漏的根源是**：由于ThreadLocalMap的生命周期跟Thread一样长，如果没有手动删除对应key就会导致内存泄漏。


**（7） 为什么使用弱引用**

 根据刚才的分析, 我们知道了： 无论ThreadLocalMap中的key使用哪种类型引用都无法完全避免内存泄漏，跟使用弱引用没有关系。

 要避免内存泄漏有两种方式：

1. 使用完ThreadLocal，调用其remove方法删除对应的Entry
2. 使用完ThreadLocal，当前Thread也随之运行结束

>  相对第一种方式，第二种方式显然更不好控制，特别是使用线程池的时候，线程结束是不会销毁的。

 也就是说，只要记得在使用完ThreadLocal及时的调用remove，无论key是强引用还是弱引用都不会有问题。那么为什么key要用弱引用呢？

 事实上，在ThreadLocalMap中的set/getEntry方法中，会对key为null（也即是ThreadLocal为null）进行判断，如果为null的话，那么是会对value置为null的。

==这就意味着使用完ThreadLocal，CurrentThread依然运行的前提下，就算忘记调用remove方法，弱引用比强引用可以多一层保障：弱引用的ThreadLocal会被回收，对应的value在下一次ThreadLocalMap调用set,get,remove中的任一方法的时候会被清除，从而避免内存泄漏。==

链接：https://blog.csdn.net/weixin_44050144/article/details/113061884

## 其他

preHandle

调用时间：Controller方法处理之前

执行顺序：链式Intercepter情况下，Intercepter按照声明的顺序一个接一个执行

若返回false，则中断执行，**注意：不会进入afterCompletion**

 

postHandle

调用前提：preHandle返回true

调用时间：Controller方法处理完之后，DispatcherServlet进行视图的渲染之前，也就是说在这个方法中你可以对ModelAndView进行操作

执行顺序：链式Intercepter情况下，Intercepter**按照声明的顺序倒着执行**。

备注：postHandle虽然post打头，但post、get方法都能处理

 

afterCompletion

调用前提：preHandle返回true

调用时间：DispatcherServlet进行视图的渲染之后

多用于清理资源