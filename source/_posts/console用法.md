---
title: console用法
slug:
date: 2019-04-30
author: summerpen
# cover: true
categories: JS
tags: 调试技巧
---
## Console 对象提供对浏览器控制台的接入,可以在任何全局对象中访问

### 1.显示信息的命令
congsole.log()、console.error()、console.warn()、console.info()
```js
let json = {a: 1, b: 2}
 console.log("log ==> ", json, new Date())
 console.error("error ==> ", json, new Date())
 console.warn("warn ==> ", json, new Date())
 console.info("info ==> ", json, new Date())
```
![](/images/console/1.png)

每一个结果在日志中都有不同的样式，可以使用浏览器控制台的日志筛选功能筛选出感兴趣的日志信息。
有两种途径使用这些方法，你可以简单的传入一组对象，其中的字符串对象会被连接到一起，输出到控制台。或者你可以传入包含零个或多个的替换的字符串，后面跟着被替换的对象列表。

 `%o` , `%O`  打印 JavaScript 对象，可以是整数、字符串或是 JSON 数据 
 `%d` , `%i`  打印整数                                               
 `%s`         打印字符串                                             
 `%f`         打印浮点数                                             
- 还可以使用`"%c"`为打印内容定义样式:
```js
console.log("%cMy stylish message", "color: red; font-style: italic");
```
![](/images/console/2.png)


### 2.console.dir()
`console.dir()`可以显示一个对象所有的属性和方法。
```js
var dog={
    name:'dog',
    bark : function(){alert("汪汪汪")}
}
console.dir(dog)
```
![](/images/console/3.png)

### 3.分组显示
如果信息太多，可以分组显示，用到的方法是`console.group()`和`console.groupEnd()`。
```js
console.log("This is the outer level");
console.group();
console.log("Level 2");
console.group();
console.log("Level 3");
console.warn("More of level 3");
console.groupEnd();
console.log("Back to level 2");
console.groupEnd();
console.debug("Back to the outer level");
```
![](/images/console/4.png)

### 4.console.table()
console.table()非常美观打印数组和对象
```js
 const typeOfConsole = [
   {name: 'log', type: 'standard'},
   {name: 'info', type: 'standard'},
   {name: 'table', type: 'standard'}
 ]

 console.table(typeOfConsole)

 const mySocial = {
   faceboo: true,
   linkedin: true,
   instagram: true,
   twitter: false
 }

 console.table(mySocial)
 ```
![](/images/console/5.png)

 ### 5.计次计时功能
 - `console.count()` 计算并输出相同的类型的次数
 - `console.time()`和`console.timeEnd()`，用来计算代码的运行时间。
 ```js
 for(let i=0;i<5;i++){
     console.count('added')
 }
 console.time("计时器一");
　　for(var i=0;i<1000;i++){
　　　　for(var j=0;j<1000;j++){}
　　}
　　console.timeEnd("计时器一");
```

![](/images/console/6.png)

### 6.断言
`console.assert()`用来判断一个表达式或变量是否为真。如果结果为否，则在控制台输出一条相应信息，并且抛出一个异常。
```js
var result = 0;
console.assert( result );
var year = 2019;
console.assert(year == 9102 );
```
![](/images/console/7.png)

### 7.追踪
`console.trace()`用来追踪函数的调用轨迹
```js
function foo() {
  function bar() {
    console.trace();
  }
  bar();
}

foo();
```
![](/images/console/8.png)

## 移动端调试方案 `vConsole`
一个轻量、可拓展、针对手机网页的前端开发者调试面板。
## 特性
- 查看 console 日志
- 查看网络请求
- 查看页面 element 结构
- 查看 Cookies、localStorage 和 SessionStorage
- 手动执行 JS 命令行
- 自定义插件
## 使用
从vConsole GitHub仓库下载 [链接](https://github.com/Tencent/vConsole) 或者使用 npm 安装:
```js
npm install vconsole
```
引入 dist/vconsole.min.js 到项目中：
```js   
<script src="path/to/vconsole.min.js"></script>
<script>
  // 初始化
  var vConsole = new VConsole();
  console.log('Hello world');
</script>
```
使用详情见[https://github.com/Tencent/vConsole]((https://github.com/Tencent/vConsole))