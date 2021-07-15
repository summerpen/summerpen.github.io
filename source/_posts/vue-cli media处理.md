---
title: vue-cli3+ media处理
subtitle: mark
date: 2021-7-15
author: summerpen
categories: JS
tags: WEB
---

# vue-cli3+ media处理

### 项目中需要对mp4文件进行处理, 记录使用到的问题

* 原先的处理方式 `处理后不能显示视频`

```js 
  // vue.config.js
  chainWebpack: config => {

      // 解析文件
      config.module
          .rule(/\.(mp4|webm|ogg|mp3|wav|flac|aac|mov)(\?.*)?$/)
          .test(/\.(mp4|webm|ogg|mp3|wav|flac|aac|mov)(\?.*)?$/)
          .use("url-loader")
          .loader("url-loader")
          .end()

  }, 

```
* 修改后的处理方式
```js
 chainWebpack: (config) => {
    config.plugins.delete("prefetch");

    /* const urlLoader = config.module.rule("images");
    // 清除已有的所有 loader。
    // 如果你不这样做，接下来的 loader 会附加在该规则现有的 loader 之后。
    urlLoader.uses.clear();
    // 添加要替换的 loader
    urlLoader
      .test(/\.(png|jpe?g|gif|svg)(\?.*)?$/)
      .use("url-loader")
      .loader("url-loader")
      .options({
        limit: 1024 * 8, // 8k
        name: "static/img/[name].[hash:8].[ext]", // 利用[path]路径获取文件夹名称并设置文件名
        fallback: "file-loader", // 当超过8192byte时，会回退使用file-loader
        context: path.resolve(__dirname, "./src"), //过滤掉[path]的相对路径
        // publicPath: "", //采用根路径
        esModule: false,
      }); */

    const mediaUrlLoader = config.module.rule("media");
    mediaUrlLoader.uses.clear();
    // 添加要替换的 loader
    mediaUrlLoader
      .test(/\.(mp4|webm|ogg|mp3|wav|flac|aac)(\?.*)?$/)
      .use("url-loader")
      .loader("url-loader")
      .options({
        limit: 1024 * 8, // 8k
        name: "static/media/[name].[hash:8].[ext]", // 设置文件名  ---------需要修改成你自己项目的路径,名称 
        // name: "[path][name].[hash:8].[ext]", // 设置文件名
        fallback: "file-loader", // 当超过8192byte时，会回退使用file-loader
        context: path.resolve(__dirname, "./src"), //过滤掉[path]的相对路径
        esModule: false,
      });
  },
```
