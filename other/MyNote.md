

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



![](pics\Strong.png)

 假设在业务代码中使用完ThreadLocal ，threadLocal Ref被回收了。

 但是因为threadLocalMap的Entry强引用了threadLocal，造成threadLocal无法被回收。

 在没有手动删除这个Entry以及CurrentThread依然运行的前提下，始终有强引用链` threadRef->currentThread->threadLocalMap->entry`，Entry就不会被回收（Entry中包括了ThreadLocal实例和value），导致Entry内存泄漏。

 也就是说，ThreadLocalMap中的key使用了强引用， 是无法完全避免内存泄漏的。


**（5）如果key使用弱引用**

 那么ThreadLocalMap中的key使用了弱引用，会出现内存泄漏吗？

 此时ThreadLocal的内存图（实线表示强引用，虚线表示弱引用）如下：

![](pics\weak.png)

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

`preHandle`

调用时间：Controller方法处理之前

执行顺序：链式Intercepter情况下，Intercepter按照声明的顺序一个接一个执行

若返回false，则中断执行，**注意：不会进入afterCompletion**

 

`postHandle`

调用前提：preHandle返回true

调用时间：Controller方法处理完之后，DispatcherServlet进行视图的渲染之前，也就是说在这个方法中你可以对ModelAndView进行操作

执行顺序：链式Intercepter情况下，Intercepter**按照声明的顺序倒着执行**。

备注：postHandle虽然post打头，但post、get方法都能处理

 

`afterCompletion`

调用前提：preHandle返回true

调用时间：DispatcherServlet进行视图的渲染之后

多用于清理资源



# spring事务

## 三个接口



PlatformTransactionManager、TransactionDefinition和TransactionStatus



```
1. PlatformTransactionManager
PlatformTransactionManager 接口是 Spring 提供的平台事务管理器，用于管理事务。该接口中提供了三个事务操作方法，具体如下。
TransactionStatus getTransaction（TransactionDefinition definition）：用于获取事务状态信息。
void commit（TransactionStatus status）：用于提交事务。
void rollback（TransactionStatus status）：用于回滚事务。
 
在项目中，Spring 将 xml 中配置的事务详细信息封装到对象 TransactionDefinition 中，然后通过事务管理器的 getTransaction() 方法获得事务的状态（TransactionStatus），并对事务进行下一步的操作。
```



```
2. TransactionDefinition
TransactionDefinition 接口是事务定义（描述）的对象，它提供了事务相关信息获取的方法，其中包括五个操作，具体如下。
String getName()：获取事务对象名称。
int getIsolationLevel()：获取事务的隔离级别。
int getPropagationBehavior()：获取事务的传播行为。
int getTimeout()：获取事务的超时时间。
boolean isReadOnly()：获取事务是否只读。
 
在上述五个方法的描述中，事务的传播行为是指在同一个方法中，不同操作前后所使用的事务。
PROPAGATION_REQUIRED    required    支持当前事务。如果 A 方法已经在事务中，则 B 事务将直接使用。否则将创建新事务
PROPAGATION_SUPPORTS    supports    支持当前事务。如果 A 方法已经在事务中，则 B 事务将直接使用。否则将以非事务状态执行
PROPAGATION_MANDATORY    mandatory    支持当前事务。如果 A 方法没有事务，则抛出异常
PROPAGATION_REQUIRES_NEW    requires_new    将创建新的事务，如果 A 方法已经在事务中，则将 A 事务挂起
PROPAGATION_NOT_SUPPORTED    not_supported    不支持当前事务，总是以非事务状态执行。如果 A 方法已经在事务中，则将其挂起
PROPAGATION_NEVER    never    不支持当前事务，如果 A 方法在事务中，则抛出异常
PROPAGATION.NESTED    nested    嵌套事务，底层将使用 Savepoint 形成嵌套事务
在事务管理过程中，传播行为可以控制是否需要创建事务以及如何创建事务。
通常情况下，数据的查询不会改变原数据，所以不需要进行事务管理，而对于数据的增加、修改和删除等操作，必须进行事务管理。如果没有指定事务的传播行为，则 Spring3 默认的传播行为是 required。
```



```
3. TransactionStatus
TransactionStatus 接口是事务的状态，它描述了某一时间点上事务的状态信息。其中包含六个操作
void flush()    刷新事务
boolean hasSavepoint()    获取是否存在保存点
boolean isCompleted()    获取事务是否完成
boolean isNewTransaction()    获取是否是新事务
boolean isRollbackOnly()    获取是否回滚
void setRollbackOnly()    设置事务回滚
```

# vue

## 各种存储

**cookie**

- 浏览器储存

- 最大4KB

- document.cookie = cname + "=" + cvalue + "; " + expires;

- document.cookie.split(';');

- 有个数限制（随浏览不同）一般不能超过20个；

- 与服务端通信，每次都会携带在HTTP头中

- 如果使用cookie保存过多数据会带来性能问题

-   (尽量不用)

**localstorage**

- 本地永久存储

- 以文件的方式存储在本地  

-  字符串    

-  适合存储用户登录信息  

-  不同页面之间的传值
-    需手动清除：localstorage.clear()
-    localStorage.getItem(key);
-    localStorage.setItem(key,value);

-   localStorage.removeItem(key);

**sessionStorage** 

- (临时存储)

- 关闭页面后自动清除，页面刷新不会清除)  

-  字符串(json对象可序列化成字符串存储)    

-    不同页面之间的传值

-    sessionStorage.getItem(key)

-    sessionStorage.setItem(key,value)

-    sessionStorage.removeItem(key)

-    sessionStorage.clear()

  

  **store (vuex)** 

- （临时存储） 

-  F5刷新vuex存储的值会全部丢失

-  字符串/对象

- vuex用于组件之间的传值

-   多个组件共用一个数据源（对象或数组）时，一个组件改变了该数据源，其他组件响应该变化 

  

  **url传值**

- vue单页面分享时会附带（移动端），造成信息读取错误
  



## {{}}

<!-- 可以直接写变量-->
        {{userName}}
        <!-- 可以写三元表达式 -->
        {{true?'男':'女'}}
        <!-- 可以调用函数  功能是倒叙-->
        {{userName.split("").reverse().join("")}}
