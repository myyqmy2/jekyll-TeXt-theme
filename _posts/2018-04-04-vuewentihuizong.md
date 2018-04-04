---
layout: post
title: 关于vue2.0单页面应用搭建中遇到的问题
key: 20180403
tags: 总结 分享
---

学vue.js也有一阵子了,上周完成了第一个基于vue全家桶构建的简单的电商demo,实现了分类页切换以及购物车相关交互,比较简易,以后有机会会再改进一下,具体代码可以去我的github查看。当然过程中间也也遇到了很多的坑，下面就来梳理一下顺便对自己的vue学习做一个简单总结。<!--more-->

页面布局采用的是饿了么的element组件库，用法类似Bootstrap尤其是栅格式布局的思想，其实我也只应用了它的栅格化做响应式导航组件以及它的跑马灯组件，所以对于element的样式改写还有些僵硬，还需要更加多的练习才行。还有一点就是绑定事件的时候不要直接绑定到它的栅格的标签上，可能会失效。

项目中，关于数据这方面采用的是本地数据虚拟后端接口，数据传输采用axios传输，mock数据采用axios-mock-adapter插件实现，遇见的第一个问题就是axios-mock-adapter插件的配置，只mock数据的话配置其实很简单。
### axios-mock-adapter MOCK本地数据方法：
>1. 安装插件
\$ npm install axios
$ npm install axios-mock-adapter –save-dev 
>2. 新建mock文件夹，内含data文件夹和index.js文件
>3. 定义一些数据存在js中，导出，存到/mock/data 
>4. index.js文件加入下边代码 
>5. 在页面上发起请求的方法 eg: app.vue

``` javascript
>> index.js
// 引入axios
var axios = require('axios')
// 引入插件
var MockAdapter = require('axios-mock-adapter')
// 实例化对象
var mock = new MockAdapter(axios)
// 模拟请求接口和返回数据
// arguments (status == 响应状态, data == 响应数据, headers == 响应头部信息)
// /users即为请求接口，res.data.users即为数据内容
mock.onGet('/users').reply(200, {
  users: [
    { id: 1, name: 'John Smith' }
  ]
})
// 导出
export default mock
```  
``` javascript
>> main.js
// 引入mock模块
import mock from 'mock';
// 引用
Vue.use(mock);
```
``` javascript
>> app.vue
// 引入axios
import axios from 'axios'
// 发送请求
axios.get('/users').then(res => {
    console.log(res.data.users)
})
```
关于其他更多参数，可以去官网查看详细设置。


----------

数据的问题解决了以后，就开始建立页面路由了，紧接着第二个遇到的问题也出现了，就是关于img元素中src的动态绑定。
###关于图片的src动态绑定：
开始的时候图片我是放在src目录下assets文件夹中的，然后本地数据中的imgSrc属性的值为图片的相对路径。
>// start之后找不到图片
>imgSrc: '../../assets/images/xxx.jpg'

因为webpack编译的时候对于assets文件夹中的内容也会编译，所以编译之后的路径会有所改变，有两种方法可以防止该情况：
>1.先对路径进行require('{path}')引用转换，然后再进行:src动态绑定
>2.将图片放入src同级static文件夹下，该文件夹为静态文件夹，不会被webpack编译

其中，第一种方法require的参数不支持变量引入，所以我采用的是第二种将images文件夹放入static目录下的方法，然后将本地数据中的imgSrc改为相对根目录的绝对路径即'/static/images/xxx.jpg',图片成功加载。

不过这里遗留了一个问题我还没有解决，就是在我dev的时候图片加载是没有问题的，但当我准备上传代码时删掉依赖包然后重新install以后发现，同样的文件，同样的路径，却又找不到图片？

----------

接下来，想说一说关于使用vuex时遇到的问题
关于vuex，我只在购物车的数据使用，为了方便交互，参考的是官方的shopping-cart实例
state.added中放入购物车实时数据，state.checkoutOk为表示checkout状态的boolean值
getters用于返回state.added和相关计算
mutations中封装了所有关于state的同步操作
actions中封装所有调用mutations的异步操作
这些基本的内容和交互都比较容易理解，唯一比较搞不明白的一点是关于购物车中商品数量改变而却不能触发相关getters数据重载的情况：当第一次进入购物车时数据没有问题，当切换到别的页面添加已有商品再重新进入购物车页面时发现数量改变却未触发结算金额的变化。本来我认为是因为检测不到数组项中对象属性的改变，现在我认为问题应该出现在vue的数据重用上，**所以当商品数量改变时，应进行购物车路由页面的重加载才能更新相应getters数据。**由于当时做项目时并未想到这一点，我会抽时间将demo再改进一下。


----------

以上就是这次vue项目搭建中遇到的主要问题，关于发布这方面的相关问题由于我还没有实战项目经验所以就先不总结了，等以后遇上了会再写相关文章总结。
这是我第一篇技术文章，写的不好请多见谅。