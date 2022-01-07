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



# 什么时候用methods？什么时候用computed？什么时候用watch



- methods是方法和原生js没区别，大多是需要我们**主动调用**(比如事件)。

- computed是get 这个get有点特殊，**只要触发所依赖数据的set会自动触发get**。我们只关心get的return set由系统触发或者依赖的数据触发，官方说依赖缓存只是为了理解。**其实Date.now()这种只是系统不能触发set**，不能触发set get当然不会通知观察者。

- **watch 是set 由data触发**，我们可以在set里进行自己的条件封装。

  

## computed


- **computed用来监控自己定义的变量，该变量不在data里面声明，直接在computed里面定义**，然后就可以在页面上进行**双向数据绑定**展示出结果或者用作其他处理；
- computed比较适合**对多个变量或者对象进行处理后返回一个结果值，也就是数多个变量中的某一个值发生了变化则我们监控的这个值也就会发生变化**，举例：购物车里面的商品列表和总金额之间的关系，只要商品列表里面的商品数量发生变化，或减少或增多或删除商品，总金额都应该发生变化。这里的这个总金额使用computed属性来进行计算是最好的选择



**computed：**通过属性计算而得来的属性

　　**1、**computed内部的函数在调用时不加()。

　　**2、**computed是依赖vm中data的属性变化而变化的，也就是说，当data中的属性发生改变的时候，当前函数才会执行，data中的属性没有改变的时候，当前函数不会执行。

　　**3、**computed中的函数必须用return返回。

　　**4、**在computed中不要对data中的属性进行赋值操作。如果对data中的属性进行赋值操作了，就是data中的属性发生改变，从而触发computed中的函数，形成死循环了。

　　**5、**当computed中的函数所依赖的属性没有发生改变，那么调用当前函数的时候会从缓存中读取。

　　

　　使用场景：当一个值受多个属性影响的时候------------购物车商品结算





  Computed: 可以关联多个实时计算的对象，当这些对象中的其中一个改变时都会出发这个属性。具有缓存能力，**所以只有当数据再次改变时才会重新渲染，否则就会直接拿取缓存中的数据**。



在Vue中**计算属性是基于它们的依赖进行缓存的，而方法是不会基于它们的依赖进行缓存的。从而使用计算属性要比方法性能更好。**



这也同样意味着下面的计算属性将不再更新，因为Date.now()不是响应式依赖：

```
computed: {
  now() {
    return Date.now()
  }
}
```



在页面中使用大量或是复杂的表达式去处理数据，对页面的维护会有很大的影响。这个时候就需要用到computed 计算属性来处理复杂的逻辑运算，这样在页面中就可以简单的写成`{{bookmark}}`，computed一般是改变data或者props里面的值为己用。



```js
computed: {
            bookmark() {
               //这里用了es6书写方法
                return this.$store.state.bookmarks.find(bookmark => bookmark.id === this.bookmarkId);
            },
}
```



```js
<div>{{bookmark}}</div>
```



**那么计算属性的缓存有什么优势呢？**

假设我们有一个性能开销比较大的计算属性 `list`，它需要遍历一个巨大的数组并做大量的计算。然后我们可能有其他的计算属性依赖于 `list`。如果没有缓存，我们将不可避免的多次执行 `list` 的 getter！

我们为什么需要缓存？假设我们有一个性能开销比较大的计算属性 **A**，它需要遍历一个巨大的数组并做大量的计算。然后我们可能有其他的计算属性依赖于 **A** 。如果没有缓存，我们将不可避免的多次执行 **A** 的 getter！**如果你不希望有缓存，请用方法来替代。**



computed和method：

**相同之处：** `computed` 和 `methods` 将被混入到 Vue 实例中。`vm.reversedMessage/vm.reversedMessage()` 即可获取相关计算属性/方法。



### 计算属性的 Setter

计算属性默认只有 getter，不过也可以去设置一个 setter，像下面一样：

```js
// ...
computed: {
  fullName: {
    // getter
    get() {
      return this.firstName + ' ' + this.lastName
    },
    // setter
    set(newValue) {
      const names = newValue.split(' ')
      this.firstName = names[0]
      this.lastName = names[names.length - 1]
    }
  }
}
// ...

```

当我们运行 `this.fullName = '小 帅'` 时，setter 会被调用，`this.firstName` 和 `this.lastName` 也会相应地被更新。

这个setter我还很少的在项目中会使用得到，大家会在什么时候能够用到呢，欢迎提供素材。



## watch

　　刚开始总是傻傻分不清到底在什么时候使用watch，什么时候使用computed。这里大致说一下自己的理解：

- watch主要用于监控vue实例的变化，它监控的变量当然必须在data里面声明才可以，它可以监控一个变量，也可以是一个对象，但是我们不能类似这样监控，比如：

```
watch:{
goodsList.price(newVal,oldVal){
    //监控商品列表中是商品价格
}
}
```

这样会报错。只能监控整个对象或单个变量，如下所示：

```js
data(){
　　　　　　　　return {
　　　　　　　　　　example0:"",
　　　　　　　　　　example1:"",
　　　　　　　　　　example2:{
 　　　　　　　　　　　　inner0:1, 　　　　　　　　　
                        　　　innner1:2 　　　　　　　　　
                    　}
　　　　　　}
　　　　},
watch:{
　example0(newVal,oldVal){//监控单个变量
           ……
   }，example2(newVal,oldVal){//监控对象
           ……
   }，
}
```

- watch一般用于监控路由、input输入框的值特殊处理等等，它比较适合的场景是一个数据影响多个数据





**watch：**属性监听

　　**1、**watch中的函数名称必须要和data中的属性名一致，因为watch是依赖data中的属性，当data中的属性发生改变的时候，watch中的函数就会执行。

　　**2、**watch中的函数有两个参数，前者是newVal，后者是oldVal。

　　**3、**watch中的函数是不需要调用的。

　　**4、**watch只会监听数据的值是否发生改变，而不会去监听数据的地址是否发生改变。也就是说，watch想要监听引用类型数据的变化，需要进行深度监听。"obj.name"(){}------如果obj的属性太多，这种方法的效率很低，obj:{handler(newVal){},deep:true}------用handler+deep的方式进行深度监听。

　　**5、**特殊情况下，watch无法监听到数组的变化，特殊情况就是说更改数组中的数据时，数组已经更改，但是视图没有更新。更改数组必须要用splice()或者$set。this.arr.splice(0,1,100)-----修改arr中第0项开始的1个数据为100，this.$set(this.arr,0,100)-----修改arr第0项值为100。

　　**6、**immediate:true  页面首次加载的时候做一次监听。

 

　　使用场景：当一条数据的更改影响到多条数据的时候---------搜索框





watch：一个数据影响多个数据。适合监控场景，某【一个】变量改变时需要做什么操作；类似于onchange，适合耗时操作，如网络请求等。

computed：一个数据受多个数据影响。某【一些】变量发生变化时，影响的【单个】结果对应地发生改变。



根据Vue官方文档的定义

> 当需要在数据变化时执行异步或开销较大的操作时，这个方式是最有用的。
>
> 

使用 `watch` 可以让我们调用异步的API来获取数据，并用`getting`属性来控制数据状态，这些使用计算属性是无法做到的。

### 使用

> 首先我们理解下watch的使用首先确认 watch是一个对象，一定要当成对象来用。对象就有键，有值。
> 键：就是你要监控的那个家伙，比如说$route，这个就是要监控路由的变化。或者是data中的某个变量。
> **值可以是函数**：就是当你监控的家伙变化时，需要执行的函数，这个函数有两个形参，第一个是当前值，第二个是变化后的值。
> **值也可以是函数名**：不过这个函数名要用单引号来包裹。第三种情况厉害了。
> **值是包括选项的对象**：选项包括有三个。
> 1.第一个handler：其值是一个回调函数。即监听到变化时应该执行的函数。
> 2.第二个是deep：其值是true或false；确认是否深入监听。（一般监听时是不能监听到对象属性值的变化的，数组的值变化可以听到。）
> 3.第三个是immediate：其值是true或false；`确认是否以当前的初始值执行handler的函数。`

```js
  watch: {
    userInfo: {
      handler (val, oldVal) {
        this.oldUser = oldVal || val
      },
      deep: true,
      immediate: true
    }
  }
```



里面的deep设为了true，这样的话，如果修改了这个userInfo中的任何一个属性，都会执行handler这个方法。



## 其他

methods是方法和原生js没区别，大多是需要我们主动调用（比如事件）。
computed是get 这个get有点特殊，只要触发所依赖数据的set会自动触发get。我们只关心get的return set由系统触发或者依赖的数据触发，官方说依赖缓存只是为了理解。其实Date.now()这种只是系统不能触发set，不能触发set get当然不会通知观察者。
watch 是set 由data触发，我们可以在set里进行自己的条件封装。

**区别：**

　　**1、**功能上：computed是计算属性，watch是监听一个值的变化，然后执行对应的回调。

　　**2、**是否调用缓存：computed中的函数所依赖的属性没有发生变化，那么调用当前的函数的时候会从缓存中读取，而watch在每次监听的值发生变化的时候都会执行回调。

　　**3、**是否调用return：computed中的函数必须要用return返回，watch中的函数不是必须要用return。

　　**4、**使用场景：computed----当一个属性受多个属性影响的时候，使用computed-------购物车商品结算。watch----当一条数据影响多条数据的时候，使用watch-------搜索框。





## VueX中的核心内容

在VueX对象中，其实不止有`state`,还有用来操作`state`中数据的方法集，以及当我们需要对`state`中的数据需要加工的方法集等等成员。

在VueX对象中，其实不止有`state`,还有用来操作`state`中数据的方法集，以及当我们需要对`state`中的数据需要加工的方法集等等成员。

成员列表：

- state     存放状态：全局访问的state对象，存放要设置的初始状态名及值（必须要有）
- mutations   state成员操作 ：里面可以存放改变 state 的初始值的方法 ( 同步操作--必须要有 )
- getters     加工state成员给外界 ：实时监听state值的变化可对状态进行处理，返回一个新的状态，相当于store的计算属性（不是必须的）
- actions     异步操作 ：里面可以存放用来异步触发 ***\*mutations\**** 里面的方法的方法 ( 异步操作--不是必须的 )
- modules   模块化状态管理 ：存放模块化的数据（不是必须的）



<font color='red'>全局配置Vuex</font>:

  在 src 目录下创建 store 文件夹,并在里面创建一个index.js文件，然后index.js中配置如下：

```js
第一步：引入Vue、和Vuex(固定写法)
import Vue from 'vue'
import Vuex from 'vuex'
Vue.use(Vuex); 
 
第二步：声明Vuex 的五个属性，其中state,mutations 是一定要定义的，其他的三个属性对象根据实际需要。
const state = {  // 初始化状态值--一定要有该属性对象         
    ...
}
const mutations = {  // 自定义改变state初始值的方法--一定要有该属性对象
    ...
}
const getters = {  // 状态计算属性--该属性对象不是必须的            
    ...
}
const actions = { // 异步操作状态--该属性对象不是必须的
    ...
}
const modules = {  // 状态模块--该属性对象不是必须的
    ...
}
 
第三步：创建一个 store 实例，将声明的五个变量赋值赋值给 store 实例，如下：
const store = new Vuex.Store({
   state,
   mutations,
    //下面三个非必须
   getters,
   actions,
   modules
})
 
第四步：导出 store 实例，供外部访问
export default store
```

在项目的main.js中将Vuex注册到全局实例中

```
...
import store from './store'
...
 
new Vue({
  el: '#app',
  router,
  store,         //注入,组件中可以使用 this.$store 获取
  components: { App },
})
```



## VueX的工作流程

<img src=".\pics\VueX的工作流程.jpg" style="zoom:75%;" />

首先，`Vue`组件如果调用某个`VueX`的方法过程中需要向后端请求时或者说出现异步操作时，需要`dispatch` VueX中`actions`的方法，以保证数据的同步。可以说，`action`的存在就是为了让`mutations`中的方法能在异步操作中起作用。

如果没有异步操作，那么我们就可以直接在组件内提交状态中的`Mutations`中自己编写的方法来达成对`state`成员的操作。注意，`1.3.3节`中有提到，不建议在组件中直接对`state`中的成员进行操作，这是因为直接修改(例如：`this.$store.state.name = 'hello'`)的话不能被`VueDevtools`所监控到。



最后被修改后的state成员会被渲染到组件的原位置当中去。



### **state属性**

​    在配置文件store/index.js中，比如初始化设置两个状态 StudNum，StudScore

```js
import Vue from 'vue'
import Vuex from 'vuex'
Vue.use(Vuex); 
 
const state = {      
    StudNum:3, // 初始化一个状态，存放学生人数
    StudScore:[ // 初始化一个状态，存放学生的分数信息
        {name:'小敏',score:80},
        {name:'小花',score:90},
        {name:'小红',score:98}
    ]
}
 
const store = new Vuex.Store({
   state
})
export default store
```

在组件中获取两个状态的值：this.$store.state.xxx

```html
<template>
<div>
    <h4>直接使用状态值</h4>
    <p>学生人数：{{$store.state.StudNum}}</p>
    <p v-for="(item,index) in $store.state.StudScore" :key="index">
        姓名：{{item.name}} | 分数：{{item.score}}
    </p>
<!------------------------->
    <h4>通过计算属性获取</h4>
    <p>学生人数：{{StudNum}}</p>
    <p v-for="(item,index) in StudScore" :key="index">
        姓名：{{item.name}} | 分数：{{item.score}}
    </p>
</div>
</template>
<script>
export default{
    computed:{ // 计算属性
        StudNum(){
             return this.$store.state.StudNum
        },
        StudScore(){
             return this.$store.state.StudScore
        },
    }
}
</script>
```

`mapState` 辅助函数

​    当一个组件需要获取多个状态时候，将这些状态都声明为计算属性会有些重复和冗余。为了解决这个问题，我们可以使用 `mapState` 辅助函数帮助我们生成计算属性，让你少按几次键。上面计算属性获取状态值用辅助函数可写法如下：

```html
<template>
<div>
    <h4>通过计算属性获取</h4>
    <p>学生人数：{{StudNum}}</p>
    <p v-for="(item,index) in StudScore" :key="index">
        姓名：{{item.name}} | 分数：{{item.score}}
    </p>
</div>
</template>
<script>
// 在单独构建的版本中辅助函数为 Vuex.mapState
import { mapState } from 'vuex'
export default {
    data(){
        return{
              localCount:12
        }
    },
    computed: mapState({
        StudNum: state => state.StudNum,  // 箭头函数可使代码更简练
        StudScore: 'StudScore',           // 可也传字符串参数 'StudScore' 等同于 `state =>state.StudScore
        nweNum(state) {  // 为了能够使用 `this` 获取局部状态，必须使用常规函数纠正this指向
            return state.count + this.localCount
        }
    })
}
</script>
```

当映射的计算属性的名称与 state 的子节点名称相同时，我们也可以给 `mapState` 传一个字符串数组,上例中的计算属性StudNum，StudScore跟state中的子节点状态名相同，因此可简写成如下写法：

```
computed: mapState([
  'StudNum',   // 映射 this.StudNum为 this.$store.state.StudNum
  'StudScore'  // 映射 this.StudScore为 this.$store.state.StudScore
])
```

**对象展开运算符**

   mapState 函数返回的是一个对象。我们如何将它与局部计算属性混合使用呢？通常，我们需要使用一个工具函数将多个对象合并为一个，以使我们可以将最终对象传给 computed 属性。但是自从有了对象展开运算符（现处于 ECMAScript 提案 stage-4 阶段），我们可以极大地简化写法：后面讲解案例用到辅助函数时都会用这种方法，也推荐使用这种写法来使用辅助函数。当然使用辅助函数也不是必须的，对于单个组件中用到的状态比较多时，使用辅助函数是个很好的选择，能极大的简化代码。具体写法如下：

```js
computed: {
  localComputed () { // 组件中的其他计算属性
       return 23
  },
  ...mapState([  // 使用对象展开运算符将此对象混入到外部对象中
      'StudNum',   // 映射 this.StudNum为 this.$store.state.StudNum
      'StudScore'  // 映射 this.StudScore为 this.$store.state.StudScore
  ])
}
```



###  Mutations

`mutations`是操作`state`数据的方法的集合，比如对该数据的修改、增加、删除等等。更改store中的状态

####  Mutations使用方法

`mutations`方法都有默认的形参：

(**[state]** **[,payload]**)

- `state`是当前`VueX`对象中的`state`
- `payload`是该方法在被调用时传递参数使用的



例如，我们编写一个方法，当被执行时，能把下例中的name值修改为`"jack"`,我们只需要这样做

```js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

const store = new Vuex.store({
    state:{
        name:'helloVueX'
    },
    mutations:{
        //es6语法，等同edit:funcion(){...}
        edit(state){
            state.name = 'jack'
        }
    }
})

export default store
```

而在组件中，我们需要这样去调用这个`mutation`——例如在App.vue的某个`method`中:

```js
this.$store.commit('edit')
```

当需要多参提交时，推荐把他们放在一个对象中来提交:

```
this.$store.commit('edit',{age:15,sex:'男'})
```

接收挂载的参数：

```js
        edit(state,payload){
            state.name = 'jack'
            console.log(payload) // 15或{age:15,sex:'男'}
        }
```

**另一种提交方式**

```js
this.$store.commit({
    type:'edit',
    payload:{
        age:15,
        sex:'男'
    }
})
```

#### 增删state中的成员

为了配合Vue的响应式数据，我们在Mutations的方法中，应当使用Vue提供的方法来进行操作。如果使用`delete`或者`xx.xx = xx`的形式去删或增，则Vue不能对数据进行实时响应。

- Vue.set 为某个对象设置成员的值，若不存在则新增

  例如对state对象中添加一个age成员

```js
Vue.set(state,"age",15)
```

Vue.delete 删除成员

将刚刚添加的age成员删除

```js
Vue.delete(state,'age')
```



#### ==**项目中**==：

```js
const mutations={
    ChangeStudScore(state,obj) {   //自定义改变state初始值的方法，这里面的参数除了state之外还可以再传额外的参数(变量或对象);
        state.StudNum = obj.length; // 更改状态StudNum 的值
        state.StudScore = obj; // 更改状态StudScore 的值
    }
```



```js
 methods: {
        ...mapMutations([// 使用辅助函数 mapMutations
            "ChangeStudScore" // 映射 this.ChangeStudScore(obj)为 this.$store.commit("ChangeStudScore", obj)
        ]),
        add() { // 点击按钮，设置状态的值
            let obj = [
                {name: '张三',score: 93},
                {name: '李四',score: 90},
                {name: '王五',score: 98},
                {name: '赵六',score: 70},
            ]
            //this.$store.commit("ChangeStudScore", obj) // 不使用辅助函数时的写法
            this.ChangeStudScore(obj)// 使用辅助函数时的写法
        }
```



使用常量替代 Mutation 事件类型( 直接复制官网,实际开发中没用过 )



使用常量替代 mutation 事件类型在各种 Flux 实现中是很常见的模式。这样可以使 linter 之类的工具发挥作用，同时把这些常量放在单独的文件中可以让你的代码合作者对整个 app 包含的 mutation 一目了然：

```js
// mutation-types.js
export const SOME_MUTATION = 'SOME_MUTATION'
```



```js
// store.js
import Vuex from 'vuex'
import { SOME_MUTATION } from './mutation-types'
 
const store = new Vuex.Store({
  state: { ... },
  mutations: {
    // 我们可以使用 ES2015 风格的计算属性命名功能来使用一个常量作为函数名
    [SOME_MUTATION] (state) {
      // mutate state
    }
  }
})
```



### Getters（三种派生）

可以对state中的成员加工后传递给外界

Getters中的方法有两个默认参数

- state 当前VueX对象中的状态对象
- getters 当前getters对象，用于将getters下的其他getter拿来用

例如

```js
getters:{
    nameInfo(state){
        return "姓名:"+state.name
    },
    fullInfo(state,getters){
        return getters.nameInfo+'年龄:'+state.age
    }  
}
```

组件中调用

```js
this.$store.getters.fullInfo
```



前面说了，getters不是必须的，那么什么时候会用到呢？有时候我们需要从 store 中的 state 中派生出一些状态（对state进行计算过滤等操作），例如上例，我们在getters中从state的子节点StudScore里派生出两个状态：90分以上的学生以及其数量：

```js
import Vue from 'vue'
import Vuex from 'vuex'
Vue.use(Vuex); 
 
const state = {      
    StudNum:2, // 初始化一个状态，代表学生人数
    StudScore:[ // 初始化一个状态，存放学生的分数信息
        {name:'小敏',score:80},
        {name:'小花',score:90},
        {name:'小红',score:98}
    ]
}
const getters = {
    // 获取分数为90分以上的学生
    perfect: state => { // 过滤分数，获取90分及以上的学生
        return state.StudScore.filter(Stud => Stud.score>=90)
    },
    // 获取分数为90分以上的学生数量
    perfectNum: (state,getters) => { // getters 也可以接受其他 getters 作为第二个参数
        return getters.perfect.length
    }
};
 
const store = new Vuex.Store({
   state,
   getters
})
export default store
 
```

在组件中获取getters派生的两个状态的值：this.$store.getters.xxx

```html
<template>
<div>
    <h4>state原始状态</h4>
    <p>学生人数：{{$store.state.StudNum}}</p>
    <p v-for="(item,index) in $store.state.StudScore" :key="index">
        姓名：{{item.name}} | 分数：{{item.score}}
    </p>
<!------------------------->
    <h4>getters 派生的状态 通过属性访问</h4>
    <p>优秀学生人数：{{$store.getters.perfectNum}}</p>
    <p v-for="(item,index) in $store.getters.perfect" :key="index">
        姓名：{{item.name}} | 分数：{{item.score}}
    </p>
</div>
</template>
<script>
import { mapState } from 'vuex'
export default{
    computed:{
        ...mapState([  // 使用辅助函数 mapState 
            'StudNum',   // 映射 this.StudNum为 this.$store.state.StudNum
            'StudScore'  // 映射 this.StudScore为 this.$store.state.StudScore
        ])
    }
}
</script>
```

从上面例子可以看出，使用getters我们从state子节点StudScore中派生了两个新的状态，这两个派生出的新状态不用在state中进行初始化，**这是getters以属性的形式返回的情况，getters也可以返回一个函数，通过函数来访问getters，我们紧接着上面的例子，在getters中再派生出一个状态**：checkScore 通过分数找出学生信息，它返回的是一个函数，如下：

```js
...
const getters = {
    ....
    // 用分数查询学生信息
    checkScore:state=>n=>{ // 返回一个方法函数
        return state.StudScore.find(Stud=> Stud.score=== n)
    }
}
...
```

在组件中通过方法访问getters派生的状态：`this.$store.getters.xxx( val ) `

```html
<template>
<div>
<!------------------------->
    <h4>getters 派生的状态 通过方法访问</h4>
    <p>有没有人得98分</p>
    <p>{{$store.getters.checkScore(98)}}</p>
    </div>
</div>
</template>
<script>
import { mapState } from 'vuex'
export default{
    computed:{
        ...mapState([  // 使用辅助函数 mapState 
            'StudNum',   // 映射 this.StudNum为 this.$store.state.StudNum
            'StudScore'  // 映射 this.StudScore为 this.$store.state.StudScore
        ])
    }
}
</script>
```



 **注意，getter 在通过属性访问时是作为 Vue 的响应式系统的一部分缓存其中的，即只要对应的state状态不发生改变，不管执行多少次getters,都会从缓存中获取getters的状态值，不会重新计算，一旦对应的state发生改变，getters就会重新计算，并缓存起来。**



**getter 在通过方法访问时，每次都会去进行调用，而不会缓存结果。即不管对应的状态有没有发生改变，访问一次getters,就会执行一次getters返回的函数，并且不会被缓存。**



 通过上面getters的使用，我们可以看到对状态进行计算操作，我们不一定非使用getters不可，我们也可以在组件中获取状态值，再对得到的值进行过滤计算等操作也是可以的，所以说getters不是必须的。

***\*`mapGetters` 辅助函数\****

`  mapGetters` 辅助函数仅仅是将 store 中的 getter 映射到局部计算属性：

```javascript
import { mapState, mapGetters } from 'vuex'
 
export default {
  computed: {
        ...mapState([    // 使用辅助函数 mapState 
            'StudNum',   // 映射 this.StudNum为 this.$store.state.StudNum
            'StudScore'  // 映射 this.StudScore为 this.$store.state.StudScore
        ]),
        ...mapGetters([  // 使用辅助函数 mapGetters
            'perfect',   // 映射 this.perfect为 this.$store.getters.perfect
            'perfectNum',// 映射 this.perfectNum为 this.$store.getters.perfectNum
            'checkScore' // 映射 this.checkScore为 this.$store.getters.checkScore()
        ])
  }
}
```



如果你想将一个 getter 属性另取一个名字，使用对象形式：

```
computed: {
    ...mapGetters({
        P:'perfect',    // 把 `this.P` 映射为 `this.$store.getters.perfect`
        N:'perfectNum', // 把 `this.N` 映射为 `this.$store.getters.perfectNum`
        S:'checkScore'  // 把 `this.S()` 映射为 `this.$store.getters.checkScore()`
    })
}
```

### Actions

Action 类似于 mutation，不同在于：

1. Action 提交的是 mutation，而不是直接变更状态。
2. Action 可以包含任意异步操作。



由于直接在`mutation`方法中进行异步操作，将会引起数据失效。所以提供了Actions来专门进行异步操作，最终提交`mutation`方法。

`Actions`中的方法有两个默认参数

- `context` 上下文(相当于箭头函数中的this)对象
- `payload` 挂载参数

例如，我们在两秒中后执行`2.2.2`节中的`edit`方法

由于`setTimeout`是异步操作，所以需要使用`actions`

```js
actions:{
    aEdit(context,payload){
        setTimeout(()=>{
            context.commit('edit',payload)
        },2000)
    }
}

//在组件中调用:
this.$store.dispatch('aEdit',{age:15})
```



#### 项目中

```js
const actions = {
    AsyncChangeStudScore(context) {
      // 模拟异步请求，5秒后获取导数据，然后触发mutations中的方法ChangeStudScore，并传值
      setTimeout(()=>{
          let obj = {
                {name: '张三',score: 93},
                {name: '李四',score: 90},
                {name: '王五',score: 98},
                {name: '赵六',score: 70},
          }
          context.commit('ChangeStudScore',obj)
      },5000)
 
    }
}
```



```js
<script>
import { mapState, mapGetters, mapMutations, mapActions } from 'vuex'
export default{
    computed:{
  computed: {
        
        ...mapState([    // 使用辅助函数 mapState 
           'StudNum',   // 映射 this.StudNum为 this.$store.state.StudNum
           'StudScore'  // 映射 this.StudScore为 this.$store.state.StudScore
        ]),
        ...mapGetters([  // 使用辅助函数 mapGetters
            'perfect',   // 映射 this.perfect为 this.$store.getters.perfect
            'perfectNum',// 映射 this.perfectNum为 this.$store.getters.perfectNum
            'checkScore' // 映射 this.checkScore为 this.$store.getters.checkScore
        ])
  }
    },
    methods: {
        ...mapActions([  // 使用辅助函数 mapMutations
            'AsyncChangeStudScore'
        ]),
        add() { // 点击按钮，设置状态的值
            //this.$store.dispatch("AsyncChangeStudScore") // 不使用辅助函数时的写法
            this.AsyncChangeStudScore(obj)// 使用辅助函数时的写法
        }
    },
}
</script>

```

看官网

或：https://blog.csdn.net/qq_41772754/article/details/88074103



# 其他

[favicon.ico (64×64) (baidu.com)](https://www.baidu.com/favicon.ico)



## 关于vue修饰符.sync

[vue](https://so.csdn.net/so/search?from=pc_blog_highlight&q=vue)是单项数据流，所以要对他进行双向数据绑定的时候需要用到.sync修饰符，最常用的是**visible.sync**在子组件里写：`this.$emit(‘update:visible’, visible)`， 使用`update:my-prop-name `的模式触发事件父组件里：

```html
<components :visible="isVisible" @update:visible="val=>isVisible=val"></components>
//简写
<components :visible.sync="isVisible"></components>

```

子组件：

```js
this.$emit('update:visible', visible)
```

作用

：**当一个子组件改变了一个 prop 的值时，这个变化也会同步到父组件中所绑定**



## vue中 关于$emit的用法

1、父组件可以使用 props 把数据传给子组件。
2、子组件可以使用 $emit,让父组件监听到自定义事件 。

vm.$emit( event, arg ) //触发当前实例上的事件

vm.$on( event, fn );//监听event事件后运行 fn； 

## vue中this.$nextTick()的理解及使用

**页面dom中的数据渲染完之后，再执行回调函数中的方法。**

会扔到当前事件队列的最后面。

## @change

```
<el-switch v-model='inputForm[config].enable'
           @change='() => {enableJudge(config)}'></el-switch>
```

这个是elementui的组件，<font color="red">所以changge是固定的，传的是事件，然后加=>是扩大作用范围</font>



=>这里也可以写成`function(){}`一样的效果 
只是外面要保存this 因为 =》 <font color="red">是让里面的子作用域的this等于外面作用域的this </font>

如果是function的写法，this就指向function自己 而enableJodge这方法是vue对象的也就是外面的this里面的 所以如果写function(){}, 要在外面const self = this来保存外面的this在function里面调用self.enableJodge(config)

`@change = 'const self = this;(function(){self.enableJodge(config)})()'`



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

## 单元测试

`@Mock`创建一个模拟。

`@InjectMocks`**创建该类的实例**，并将使用`@Mock`（或`@Spy`）注释创建的模拟注入该实例。（不能用接口注入，因为是mock的东西注入到InjectMocks里面，是先创建这个类的实例，接口不能创建）

请注意，你必须使用`@RunWith(MockitoJUnitRunner.class)`或`Mockito.initMocks(this)`初始化这些模拟并注入它们。



虚假的数据





FixMethodOrder 按照方法的顺序

## 部署



```
@Profile(["default", "int", "pp", "prod"])
```

`application-int.properties`

`application.properties`



`application-<env>.properties 会和 application.properties合并`

相同的变量会覆盖



取名只要`application-<env>.properties`



## AOP

切点中

```
@Pointcut("execution(* com.xxx.xxx.processor..process(..))")
```



`processor..process`  中间的..这是无视多少层文件夹

# 日志



## 为什么

代码改好了怎么办？当然是删除没有用的`System.out.println()`语句了。

如果改代码又改出问题怎么办？再加上`System.out.println()`。

反复这么搞几次，很快大家就发现使用`System.out.println()`非常麻烦。

怎么办？

解决方法是使用日志。

那什么是日志？日志就是Logging，它的目的是为了取代`System.out.println()`。

输出日志，而不是用`System.out.println()`，有以下几个好处：

1. 可以设置输出样式，避免自己每次都写`"ERROR: " + var`；
2. 可以设置输出级别，禁止某些级别输出。例如，只输出错误日志；
3. 可以被重定向到文件，这样可以在程序运行结束后查看日志；
4. 可以按包名控制日志级别，只输出某些包打的日志；
5. 可以……



## 分类

`Commons Logging`和`Log4j`这一对好基友，它们一个负责充当日志API，一个负责实现日志底层，搭配使用非常便于开发。

`SLF4J`类似于`Commons Logging`，也是一个日志接口，而`Logback`类似于`Log4j`，是一个日志的实现

为什么有了Commons Logging和Log4j，又会蹦出来SLF4J和Logback？这是因为Java有着非常悠久的开源历史，不但OpenJDK本身是开源的，而且我们用到的第三方库，几乎全部都是开源的。开源生态丰富的一个特定就是，同一个功能，可以找到若干种互相竞争的开源库。

因为对Commons Logging的接口不满意，有人就搞了SLF4J。因为对Log4j的性能不满意，有人就搞了Logback。

我们先来看看SLF4J对Commons Logging的接口有何改进。在Commons Logging中，我们要打印日志，有时候得这么写：

```
int score = 99;
p.setScore(score);
log.info("Set score " + score + " for Person " + p.getName() + " ok.");
```

拼字符串是一个非常麻烦的事情，所以SLF4J的日志接口改进成这样了：

```
int score = 99;
p.setScore(score);
logger.info("Set score {} for Person {} ok.", score, p.getName());
```

从目前的趋势来看，越来越多的开源项目从Commons Logging加Log4j转向了SLF4J加Logback。



现有的日志框架
JUL（java util logging）、logback、log4j、log4j2
JCL（Jakarta Commons Logging）、slf4j（ Simple Logging Facade for Java）
日志门面
JCL、slf4j
日志实现
JUL、logback、log4j、log4j2



### 小结

日志是为了替代`System.out.println()`，可以定义格式，重定向到文件等；

日志可以存档，便于追踪问题；

日志记录可以按级别分类，便于打开或关闭某些级别；

可以根据配置文件调整日志，无需修改代码；

Java标准库提供了`java.util.logging`来实现日志功能。

## JCL 学习

全称为Jakarta Commons Logging，是Apache提供的一个通用日志API。
它是为 "所有的Java日志实现"提供一个统一的接口，它自身也提供一个日志的实现，但是功能非常常弱
（SimpleLog）。所以一般不会单独使用它。他允许开发人员使用不同的具体日志实现工具: Log4j, Jdk
自带的日志（JUL)

## 我们为什么要使用日志门面：

1. 面向接口开发，不再依赖具体的实现类。减少代码的耦合
2. 项目通过导入不同的日志实现类，可以灵活的切换日志框架
3. 统一API，方便开发者学习和使用

4. 统一配置便于项目日志的管理



为什么要使用SLF4J作为日志门面？

* 1. 使用SLF4J框架，可以在部署时迁移到所需的日志记录框架。
* 2. SLF4J提供了对所有流行的日志框架的绑定，例如log4j，JUL，Simple logging和NOP。`因此可以
     在部署时切换到任何这些流行的框架。`
* 3. 无论使用哪种绑定，SLF4J都支持参数化日志记录消息。由于SLF4J将应用程序和日志记录框架分离，
     因此可以轻松编写独立于日志记录框架的应用程序。而无需担心用于编写应用程序的日志记录框架。
* 4. SLF4J提供了一个简单的Java工具，称为迁移器。使用此工具，可以迁移现有项目，这些项目使用日志
     框架(如Jakarta Commons Logging(JCL)或log4j或Java.util.logging(JUL))到SLF4J。



**如何使用SLF4J?**

既然SLF4J只是一个接口，那么实际使用时必须要结合具体的日志系统来使用，不能单独使用



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



## swich case 和if

相同点:都可以实现多分支结构
不同点:

- switch:一般 **只能用于等值比较**
- if-else if:可以**处理范围**



通常而言大家普遍的认知里switch case的效率高于if else。根据我的理解而言switch的查找类似于二叉树，if则是线性查找。按照此逻辑推理对于对比条件数目大于3时switch更优，并且对比条件数目越多时switch的优势越为明显。



实际应用当中，我一般遵循以下编码“潜规则”：
1.凡是判断层级达到4层以上的，用switch结构。
2.凡是可能性最大的选项，放在if结构的最顶端。这个思想，也是ARM公司在ARM处理器多级流水线中加入“分支预测”功能的考量之一。



switch...case与if...else的根本区别在于，switch...case会生成一个`跳转表`来指示实际的case分支的地址，而这个跳转表的索引号与switch变量的值是相等的。从而，switch...case不用像if...else那样遍历条件分支直到命中条件，**而只需访问对应索引号的表项从而到达定位分支的目的。**



具体地说，<font color="red">switch...case会生成一份大小（表项数）为最大case常量＋1的跳表，程序首先判断switch变量是否大于最大case 常量，若大于，则跳到default分支处理；否则取得索引号为switch变量大小的跳表项的地址（即跳表的起始地址＋表项大小＊索引号），程序接着跳到此地址执行，到此完成了分支的跳转</font>。

由此看来，**switch有点以空间换时间的意思**，而事实上也的确如此。



1.switch用来根据一个整型值进行多路分支，并且编译器可以对多路分支进行优化
2.switch-case只将表达式计算一次,然后将表达式的值与每个case的值比较,进而选
 择执行哪一个case的语句块
3.if..else 的判断条件范围较广，每条语句基本上独立的，每次判断时都要条件加载
 一次。
所以在多路分支时用switch比if..else if .. else结构要效率高。



参考链接：https://www.cnblogs.com/balingybj/p/5751707.html



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



方便测试，mockito的反射是以接口来做的，应该是JDK的动态代理实现的



# 其他



早会：stand up meeting



IPM:

teration Planning Meeting, 迭代计划会议，又称Sprint计划会议：是一个开发迭代周期开始的团队活动；简单的说，它主要负责
定义和产出：哪些人（WHO）/什么时候、多长周期内（WHEN）/哪些任务（WHAT），产出INTERATION GOAL/INTERATION
PLAN / RELESE PLAN,

好处：能对迭代有一个清楚的总结，更好了解整体项目进度和迭代进度；明确迭代的整个团队目标以及个人目标，团队对于需求的理
解达到了统一；确定所有任务的点数，风险在评估中更明显的暴露出来;

下一个迭代的 Story,对下一个迭代的期望,团队的人员可用性,风险的评估总结。



PD：product design 产品的设计人   提需求的人 



story 需求



support主要包含的是回邮件和找问题  用户报告的就是

 发现的问题就会建一个bug  或者story



support case主要是解答客户疑问



HL desgin ： high level desgin ，是PD来搞的，应该是她们design完成了



版本过程：

一般来说是三个星期一个大一点的版本 =》pp
一个星期的大部分是bug fix 的小版本=》pp
三个星期，一周开发，一周内部测试，一周用户测试=》pp



内部测试就是PP的测试，我们一起的PP(DEV)， PP(PD),不是有这些review么



dev是内部人员测，pd是pd来测

用户测试是合作方的用户来PP上来测



PP的永远是release版本和prod一致，连环境都搭建的一样

PP=》prod :小版本当前星期就上了，大版本一两个星期后

