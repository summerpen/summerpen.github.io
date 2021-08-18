---
title: 关闭页面时向后台发送消息(sendBeacon+koa2)
subtitle: 使用navigator.sendBeacon, 在页面关闭时向后台发送请求，并使用node.js处理请求
date: 2021-8-18
author: summerpen
categories: Node.js
tags: Node.js
---

### 在用户关闭页面的时候，前端要给后台发一条请求，释放该页面的授权占用

### 使用 navigator.sendBeacon()

* 这个方法主要用于满足统计和诊断代码的需要，这些代码通常尝试在卸载（unload）文档之前向web服务器发送数据
* 用法
 `navigator.sendBeacon(url, data); `

`url` 表明 `data` 将要被发送到的网络地址
`data` 参数是将要发送的 `ArrayBufferView` 或 `Blob` 、 `DOMString` (js中即string) 或者 `FormData` 类型的数据。
* 返回值
当用户代理成功把数据加入传输队列时，sendBeacon() 方法将会返回 `true` ，否则返回 `false` 。

### 前端代码

```js
window.beginTime = 0; //执行onbeforeunload的开始时间
window.differTime = 0; //时间差
window.onbeforeunload = function() {
    window.beginTime = new Date().getTime();
};
window.onunload = function() {
    window.differTime = new Date().getTime() - window.beginTime;
    if (window.differTime <= 5) {
        console.log("浏览器关闭");
        const data = {
            name: "loginout关闭页面",
        };
        window.navigator.sendBeacon(
            "http://127.0.0.1:3000/loginout",
            JSON.stringify(data)
        );
    } else {
        console.log("浏览器刷新页面操作");
    }
};
```

### node.js代码

* 在一个空白目录下 `npm init -y` 生成package.json 然后安装依赖 `npm i koa koa-router koa-bodyparser co-body -S`

*  在该目录下新建app.js 

```js
const Koa = require("koa");
const Router = require("koa-router");
const fs = require("fs");
const bodyParser = require("koa-bodyparser");
const coBody = require("co-body");
const app = new Koa();
const router = new Router();
app.use(bodyParser());
// 此处接口做了兼容, 可以处理中正常post请求, 以及 navigator.sendBeacon的调用
router.post("/loginout", async function(ctx, res) {
    // url路径传入 ?json=true ( http://127.0.0.1:3000/loginout?isJson=true),代表传入json格式 (可用postman 使用post请求)
    if (!ctx.query.isJson) {
        let params = await coBody.json(ctx.req);
        console.log(params);
        ctx.body = params;
    } else {
        // text/plain格式 (可用postman 使用post请求, 或者在页面中执行 前端代码中的navigator.sendBeacon)
        //http://127.0.0.1:3000/api/loginout
        // navigator.sendBeacon('http://localhost:3000/loginout','{"a":9}')
        console.log(ctx.request.body);
        ctx.body = ctx.request.body;
    }
});
app.use(router.routes()).use(router.allowedMethods());
app.listen(3000);
console.log("server started at http:localhost:3000");
```

* 启动服务 `node app.js`, 如果全局安装了pm2, 也可使用 `pm2 start app.js`

### 进行测试该功能

1. 在项目中执行前端代码,观察node服务日志
2. 也可以在postman中执行接口调用,查看node服务日志
* 测试效果
![](/images/koa2+sendBeacon.gif)

### 参考链接

1. [js判断浏览器窗口的关闭与刷新](https://blog.wangpengpeng.site/2020/06/19/js%E5%88%A4%E6%96%AD%E6%B5%8F%E8%A7%88%E5%99%A8%E7%AA%97%E5%8F%A3%E7%9A%84%E5%85%B3%E9%97%AD%E4%B8%8E%E5%88%B7%E6%96%B0/#%E6%9C%80%E7%BB%88%E8%A7%A3%E5%86%B3%E6%96%B9%E6%B3%95)
2. [你知道关闭页面时怎么向后台发送消息吗？
](https://juejin.cn/post/6997016317635084319)
3. [stack overflow: How to receive data posted by “navigator.sendbeacon” on node.js server?](https://stackoverflow.com/questions/31355128/how-to-receive-data-posted-by-navigator-sendbeacon-on-node-js-server)
