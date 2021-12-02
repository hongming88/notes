`@Mock`创建一个模拟。

`@InjectMocks`**创建该类的实例**，并将使用`@Mock`（或`@Spy`）注释创建的模拟注入该实例。（不能用接口注入，因为是mock的东西注入到InjectMocks里面，是先创建这个类的实例，接口不能创建）

请注意，你必须使用`@RunWith(MockitoJUnitRunner.class)`或`Mockito.initMocks(this)`初始化这些模拟并注入它们。



虚假的数据





FixMethodOrder 按照方法的顺序





story 需求





support主要包含的是回邮件和找问题  用户报告的就是

 发现的问题就会建一个bug  或者story



support case主要是解答客户疑问



# mongbd

**更高的写入负载**

默认情况下，MongoDB更`侧重高数据写入性能，而非事务安全`，MongoDB很适合业务系统中`有大量“低价值”数据的场景`。但是应当避免在高事务安全性的系统中使用MongoDB，除非能从架构设计上保证事务安全。



Mongo的存储方式为`虚拟内存+持久化存储`，Mongo将数据写入内存中，再由虚拟内存管理器将其持久化到硬盘中，因此`写操作会比关系型数据库快很多`。NOSQL的存储格式是key-value形式，可以像关系型数据库那样存储基础数据类型的数据，也可以存储集合、对象等等。NoSQL虽然性能比较高，但是`并不支持事物`，也`不能进行联表查询`，`一般用于较大规模数据的存储`。



Mysql：

1）这些数据通常需要做结构化查询，比如join，这时候，关系型数据库就要胜出一筹 

2）这些数据的规模、增长的速度通常是可以预期的 

3）事务性、一致性

4）丰富的锁机制



## 什么时候用mongo

**日志系统**

系统运行过程中产生的日志信息，一般种类较多、范围较大、内容也比较杂乱。通过MongoDB可以将这些杂乱的日志进行收集管理。不仅方便了管理，查找或者导出也会变得非常容易

**地理位置存储**

MongoDB支持地理位置、二维空间索引，可以存储经纬度，因此可以很快的计算出两点之间的距离，等位置信息。如查询附近的人、或者订餐系统、配送系统等

**数据规模增长很快**

前面提到过关系型数据库数据量过大时，需要进行分库分表，这样真正操作起来可能会比较麻烦。如果选择mongo进行分库分表操作时，就会变得很简单。

**保证高可用的环境**

Mongo本身就拥有高可用及分区的解决方案，设置主从服务器非常方便，除此之外Mongo还可以快速并且安全的实现故障节点的转移。

**文件存储需求**

GridFS是MongoDB规范，用于存储和检索图片、音频、视频等大文件。GridFS虽然是文件存储的一种方式，可以存储超过16M的文件。但是它本身又是存储在MongoDB集合中的

**其他场景**

如游戏开发中我们可以通过MongoDB存储用户信息、装备、积分等，除此之外物流系统、社交系统、甚至物联网系统，Mongo都能提供完美的数据存储服务。







# postman

`application/x-www-form-urlencoded` ： 窗体数据被编码为名称/值对。这是标准的编码格式。(一般用这个)

`multipart/form-data` ： 窗体数据被编码为一条消息，页上的每个控件对应消息中的一个部分。

`text/plain` ： 窗体数据以纯文本形式进行编码，其中不含任何控件或格式字符。

补充

form的enctype属性为编码方式，常用有两种： `application/x-www-form-urlencoded` 和 `multipart/form-data` ， 默认为`application/x-www-form-urlencoded `。

当action为get时候，浏览器用`x-www-form-urlencoded`的编码方式把form数据转换成一个字串`（name1=value1&name2=value2...）`，然后把这个字串append到url后面，用?分割，加载这个新的url。



### 1、application/x-www-from-urlencoded

- 窗体数据被编码为**键值对**，不同键值对用 & 分隔，这是标准的编码格式。
- 它是 post 的默认格式，使用 js 中 URLencode 转码方法。包括将键值对中的**空格替换为加号** , 将**非 ascii 字符做百分号编码**。
  - 百分号编码：**汉字转化为 UTF-8 编码，并在每一个字节前面加 %**。以`残夜`为例，`残`的 UTF-8 编码是 `E6 AE 8B`，`夜`是 `E5 A4 9C`，拼接后是 `%E6%AE%8B%E5%A4%9C`，**拼接后变成九个 ascii 字符，占九个字节。**

**数据格式：**

![](.\pics\postman1.png)



### 2、multipart/form-data

- **数据包格式：**会将表单的数据处理为一条消息，以标签为单元，用分隔符分开。既可以上传键值对，也可以**上传文件**。当上传的字段是文件时，会有 Content-Type 来说明文件类型；
- 由于有 boundary (分割线) 隔离，所以 multipart/form-data **既可以上传文件，也可以上传键值对**，它采用了键值对的方式，所以可以上传多个文件。
- 传输非 ascii 码字符时，一个汉字占用的 3 个字节会直接以 utf-8 编码形式拼到数据包中，**因此，只占用三个字节。极大提高了效率，适合传输长字节。**

**数据格式：**

![](.\pics\postman2.png)



Controller 层

- @RequestParam：
  - GET：支持（单个参数）
  - POST：支持 x-www-form-urlencoded，不支持 Json、XML（”status”: 400）
- @RequestBody：
  - GET：不支持
  - POST：不支持 x-www-form-urlencoded（”status”: 415），支持 Json、XML（解析为 javabean）

### 3、raw

- 可以上传任意格式的文本，可以上传 text、json、xml、html 等



### 4、binary

- 相当于 `Content-Type:application/octet-stream`，从字面意思得知，只可以上传二进制数据，通常用来上传文件，由于没有键值，所以，一次只能上传一个文件。





**multipart/form-data与x-www-form-urlencoded区别**

multipart/form-data：既可以上传文件等二进制数据，也可以上传表单键值对，只是最后会转化为一条信息；

x-www-form-urlencoded：只能上传键值对，并且键值对都是间隔分开的。



# vue

>
>
>col-lg一般用于大屏设备， (min-width:1200px);
>
>col-md一般用于中屏设备， (min-width：992px);
>
>col-sm一般用于小屏设备， (min-width：768px);
>
>col-xs用于超小型设备， (max-width：768px);
>————————————————
>
>



`:xs=“4” :sm=“6” :md=“8” :lg=“9” :xl=“11”`
调整参数，就可以适应不同屏幕，而不会使页面在不同分辨率的屏幕下显得杂乱。
屏幕被分成12分，参数是几，就是占几份。
如果需要一行占1个，参数就12（12/12）
如果需要一行占2个，参数就6（6/12）
如果需要一行占3个，参数就4（4/12）
————————————————

vue @click.native 原生点击事件：

1，给vue组件绑定事件时候，必须加上native ，不然不会生效（`监听根元素的原生事件`，使用 `.native` 修饰符）

2，等同于在自组件中：

  子组件内部处理click事件然后向外发送click事件：`$emit("click".fn)`





根据Vue2.0官方文档关于父子组件通讯的原则，父组件通过prop传递数据给子组件，子组件触发事件给父组件。但父组件想在子组件上监听自己的click的话，需要加上`native`修饰符，故写法就像上面这样。



element form里面如果只有一个input框的时候 按回车会提交表单导致页面刷新 可以在form加个

```
@submit.native.prevent 
```

就可以避免这个情况



# sql

1. 脏读 :脏读就是指当一个事务正在访问数据,并且对数据进行了修改,而这种修改还没有提交到数据库中,这时,另外一个事务也访问 这个数据,然后使用了这个数据。
2. 不可重复读 :是指在一个事务内,多次读同一数据。在这个事务还没有结束时,另外一个事务也访问该同一数据。那么,在第一个事务中的两 次读数据之间,由于第二个事务的修改,那么第一个事务两次读到的的数据可能是不一样的。这样就发生了在一个事务内两次读到的数据是不一样的,因此称为是不 可重复读。例如,一个编辑人员两次读取同一文档,但在两次读取之间,作者重写了该文档。当编辑人员第二次读取文档时,文档已更改。原始读取不可重复。如果 只有在作者全部完成编写后编辑人员才可以读取文档,则可以避免该问题。
3. 幻读 : 是指当事务不是独立执行时发生的一种现象,例如第一个事务对一个表中的数据进行了修改,这种修改涉及到表中的全部数据行。 同时,第二个事务也修改这个表中的数据,这种修改是向表中插入一行新数据。那么,以后就会发生操作第一个事务的用户发现表中还有没有修改的数据行,就好象 发生了幻觉一样。例如,一个编辑人员更改作者提交的文档,但当生产部门将其更改内容合并到该文档的主复本时,发现作者已将未编辑的新材料添加到该文档中。 如果在编辑人员和生产部门完成对原始文档的处理之前,任何人都不能将新材料添加到文档中,则可以避免该问题。

补充 : 基于元数据的 Spring 声明性事务 :

Isolation 属性一共支持五种事务设置,具体介绍如下:

l DEFAULT 使用数据库设置的隔离级别 ( 默认 ) ,由 DBA 默认的设置来决定隔离级别 .

l READ_UNCOMMITTED 会出现脏读、不可重复读、幻读 ( 隔离级别最低,并发性能高 )

l READ_COMMITTED 会出现不可重复读、幻读问题(锁定正在读取的行)

l REPEATABLE_READ 会出幻读(锁定所读取的所有行)

l SERIALIZABLE 保证所有的情况不会发生(锁表)

不可重复读的重点是修改 :
同一事务,两次读取到的数据不一样。
幻读的重点在于新增或者删除
同样的条件 , 第 1 次和第 2 次读出来的记录数不一样

脏读:

强调的是第二个事务读到的不够新。

# spring

## 注入

一个接口下多个类

类上写@service（“userDaoImpl1”）

```
 @Autowired //根据类型进行注入 
 @Qualifier("userDaoImpl1") //根据名称进行注入 
 private UserDao userDao;
```

两个都要写



# java

## 静态

与类无关的方法idea会推荐静态

**如果有注入**，方法中使用了，不能使用静态



```
@Autowired 
private static JdbcTemplate jdbcTemplate;
```

单纯看这个注入过程是没有报错的,但是在接下来的`jdbcTemplate.query()`会报空指针错误.



为什么呢?静态变量/类变量不是对象的属性,而是一个类的属性,spring则是基于对象层面上的依赖注入。

意思就是：**我们注入相当于创建的是对象，不能用static修饰,类变量才能用static**



## Spring静态注入

xml方式实现；

```
<bean id="mongoFileOperationUtil" class="com.*.*.MongoFileOperationUtil" init-method="init">
	<property name="dsForRW" ref="dsForRW"/>
</bean>
```

```java
public class MongoFileOperationUtil {
    
    private static AdvancedDatastore dsForRW;
 
    private static MongoFileOperationUtil mongoFileOperationUtil;
 
    public void init() {
        mongoFileOperationUtil = this;
        mongoFileOperationUtil.dsForRW = this.dsForRW;
    }
 
}
```







`@PostConstruct`方式实现

```java
import org.mongodb.morphia.AdvancedDatastore;
import org.springframework.beans.factory.annotation.Autowired;
 
 
@Component
public class MongoFileOperationUtil {
    @Autowired
    private static AdvancedDatastore dsForRW;
 
    private static MongoFileOperationUtil mongoFileOperationUtil;
 
    @PostConstruct
    public void init() {
        mongoFileOperationUtil = this;
        mongoFileOperationUtil.dsForRW = this.dsForRW;
    }
 
}
```



@PostConstruct 注解的方法在加载类的构造函数之后执行，也就是在加载了构造函数之后，执行init方法；(@PreDestroy 注解定义容器销毁之前的所做的操作)

这种方式和在xml中配置 init-method和 destory-method方法差不多，定义spring 容器在初始化bean 和容器销毁之前的所做的操作；



3.set方法上添加@Autowired注解，类定义上添加@Component注解；

```java
import org.mongodb.morphia.AdvancedDatastore;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
 
 
@Component
public class MongoFileOperationUtil {
 
    private static AdvancedDatastore dsForRW;
    
    @Autowired
    public void setDatastore(AdvancedDatastore dsForRW) {
        MongoFileOperationUtil.dsForRW = dsForRW;
    }
}
```

首先Spring要能扫描到AdvancedDatastore的bean，然后通过setter方法注入；

然后注意：成员变量上不需要再添加@Autowired注解；





# transient

`private transient DataSource datasource = null;`
请问此中的transient起什么作用？



比如说要在网络上传输一个Class，而其中有一属性不想被传送，这一项设为transient就不会被序列化传送



`@transient `就是在给某个javabean上需要添加个属性，但是这个属性你又不希望给存到数据库中去，**仅仅是做个临时变量**，用一下。不修改已经存在数据库的数据的数据结构。



transient使用小结 

1）一旦变量被transient修饰，变量将不再是对象持久化的一部分，该变量内容在序列化后无法获得访问。

2）transient关键字只能修饰变量，而不能修饰方法和类。注意，本地变量是不能被transient关键字修饰的。变量如果是用户自定义类变量，则该类需要实现Serializable接口。

3）被transient关键字修饰的变量不再能被序列化，一个静态变量不管是否被transient修饰，均不能被序列化。

总之，java 的transient关键字为我们提供了便利，你只需要实现Serilizable接口，将不需要序列化的属性前添加关键字transient，序列化对象的时候，这个属性就不会序列化到指定的目的地中。

# 

## 序列化：

无论你是保存到本地，还是通过网线传输，信息都要转化成“00111011”的样子。注意这两个场景：

- 本地存储
- 网络传输

实在记不住的话只记住一点即可：**凡是离开内存的信息都要进行序列化**。

序列化的意思是:==将一个对象转换为可传输的数据.== 变成字节码



**java序列化后不能用python反序列化，但两种语言可以用json进行传递解析。**



也就是你Python序列化后的字节流，你自己清楚该怎么反序列回去，而我Java则是按照自己的序列化规则走，因此，就导致，双方无法通过序列化后的字节流进行交流。



https://blog.csdn.net/Appleyk/article/details/78052900?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-5.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-5.nonecase






## 为什么注入接口

通常是service层需要aop。用接口的话，AOP可以使用java自带的动态代理，但是有点麻烦要写接口。用类就要用第三方包cglib，但是简单，不用写接口
jdk规定动态代理必须用接口；当然也可以用类，用cglib可以去处理就可以了