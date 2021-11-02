`@Mock`创建一个模拟。`@InjectMocks`创建该类的实例，并将使用`@Mock`（或`@Spy`）注释创建的模拟注入该实例。

请注意，你必须使用`@RunWith(MockitoJUnitRunner.class)`或`Mockito.initMocks(this)`初始化这些模拟并注入它们。



虚假的数据





FixMethodOrder 按照方法的顺序





story 需求





support主要包含的是回邮件和找问题  用户报告的就是

 发现的问题就会建一个bug  或者story



support case主要是解答客户疑问



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