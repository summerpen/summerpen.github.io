---
title: less中~的用法
subtitle: ~的两种用法
date: 2021-8-25
author: summerpen
categories: Less
tags: Less
---

### `~` 用法

1. 动态路径

* 在 less 中，使用 `~` 为前缀就会变成动态的，他默认会去寻找 `node_modules` 下的模块，也就是以 `node_modules` 为原点，所以你可以这么使用：

```less
@import "~element-ui/lib/theme-chalk/index.css";
```

不加 `~` 时, 会从当前路径寻找资源, `./element-ui/lib/theme-chalk/index.css` 从而找不到对应资源
![](/images/less引入.png)
* 在vue中 如果使用了别名`assets` 引用静态资源(如图片等), 在vue `template`中直接引入图片时, 也是需要加`~`, 才能正确访问到图片资源

> 错误写法 `<img src="assets/image/camera.png"/>`
> 正确写法 `<img src="~assets/image/camera.png"/>`

别名配置
```js
module.exports = {
    configureWebpack: {
        resolve: {
            alias: {
                assets: "@/assets",
            }
        }
    }
};
```

2. 延迟计算

```less
   height: ~"-moz-calc(100% - 54px)";
   height: ~"-webkit-calc(100% - 54px)";
   height: ~"calc(100% - 54px)";
```

此时less不会计算出结果，而是到了用户浏览器再由实际浏览器进行计算。
