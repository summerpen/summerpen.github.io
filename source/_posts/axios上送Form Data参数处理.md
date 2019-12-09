---
title: Axios参数处理
subtitle: mark
date: 2019-12-09
author: summerpen
categories: JS
tags: WEB
---

# Axios上送Form Data参数处理

### axios 上送 formData 格式处理的几种方式

* 后台接口需要上送 jsonString={user_id:'', pd:'', } 这种格式的参数

  
  > 在网上找了几种方法, Form Data 均上送不对, 
  > 找到的方式如下:

``` js
// 1.
this.$axios({
    url: "/api",
    method: "post",
    headers: {
        "Content-Type": "application/x-www-form-urlencoded"
    },
    data: qs.stringify(jsonString) //直接qs.stringify,上传参数不对:  如图1
    //    {"0":"t","1":"i","2":"m","3":"e","4":"F","5":"r","6":"o","7":"m","8":": (unable to decode value)
    // ","35":"t","36":"i","37":"m","38":"e","39":"T","40":"o","41":": (unable to decode value)
    // ","68":"u","69":"s","70":"e","71":"r","72":"_","73":"i","74":"d","75":": ","76":"
    // ","77":"p","78":"d","79":": "}
});
// 2.
let jsonString = new URLSearchParams();
jsonString.append("user_id", "***");
jsonString.append("pd", "***");
axios({
    url: "/api",
    method: "post",
    headers: {
        "Content-Type": "application/x-www-form-urlencoded"
    },
    // data: jsonString //参数 不带 jsonString= ,不能使用
    data: `jsonString=${jsonString}` //参数不对, Form Data 变成了{"0":"j","1":"s","2":"o","3":"n","4":"S","5":"t","6":"r","7":"i","8":"n","9":"g","10":": ","11":"[","12":"o","13":"b","14":"j","15":"e","16":"c","17":"t","18":" ","19":"O","20":"b","21":"j","22":"e","23":"c","24":"t","25":"]"}
});
```
![图1](/images/axios1.png)

> 研究后, 可以使用的其他方式:

* 方法 1: 使用 axios.create

``` js
//在script顶部引入qs,axios
import axios from "axios";
//  使用axios.create
// 先create
axios_ = axios.create({
    headers: {
        "Content-Type": "application/x-www-form-urlencoded"
    }
});
var params = {
    user_id: "***",
    pd: "***"
};
// 使用 create后的axios_
axios_({
        url: "/api",
        method: "post",
        data: "jsonString=" + JSON.stringify(params)
    })
    .then(function(response) {})
    .catch(function(err) {});
```

* 方法 2: 使用 qs 将参数拼接到 url 后边,不太推荐此方法.post请求后拼接请求参数,看着比较奇怪

``` js
// 方法2:
//在script顶部引入qs,axios
import qs from "qs";
import axios from "axios";
let params = {
    user_id: "***",
    pd: "***"
};
axios({
    url: this.$api.CustomGroupPageInfo +
        "?" +
        qs.stringify({
            jsonString: JSON.stringify(params)
        }),
    method: "post",
    headers: {
        "Content-Type": "application/x-www-form-urlencoded"
    }
});
```

* 方法 3: 使用 qs 处理 data 参数

``` js
//在script顶部引入qs,axios
import qs from "qs";
import axios from "axios";
let params = {
    user_id: "***",
    pd: "***"
};
axios({
    url: "/api",
    method: "post",
    headers: {
        "Content-Type": "application/x-www-form-urlencoded"
    },
    data: params,
    transformRequest: [
        function(data) {
            data = qs.stringify({
                jsonString: JSON.stringify(params)
            });
            return data;
        }
    ]
});
```

