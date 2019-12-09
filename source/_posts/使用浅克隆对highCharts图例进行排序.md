---
title: 使用浅克隆对highCharts图例进行排序
subtitle: 可视化
date: 2019-08-13
author: summerpen
categories: Blog
tags: JS, HTML
---

### highcharts图例排序 

> 工作中有需求, 需要pie图图例进行排序, 但图表的顺序不能改变
> 看了下highcharts文档API找到了legendIndex, 可以实现对legend图例进行排序
> 主要代码如下  

``` js
var data = [

    {
        name: 'Edge',
        y: 4.67
    },
    {
        name: 'Internet Explorer',
        y: 11.84
    },
    {
        name: 'Firefox',
        y: 10.85
    },
    {
        name: 'Safari',
        y: 4.18
    },
    {
        name: 'Chrome',
        y: 61.41,
        sliced: true,
        selected: true
    },
    {
        name: 'Other',
        y: 7.05
    }
];
/* -------------- 核心操作  -------------- */
//对 对象数组进行浅克隆
var tmp = data.slice();
//对克隆后数组进行排序
tmp.sort((a, b) => {
    return b.y - a.y
})
// 对克隆后数组添加legendIndex属性, 
// tmp中每个元素与data中对相应的元素,均为对象,执行同一个地址
// 因而data中每个对象也会有legendIndex属性,并按其y属性进行了排序
tmp.map((item, index) => {
    item.legendIndex = index;
    return item;
})

Highcharts.chart('container', {
    chart: {
        plotBackgroundColor: null,
        plotBorderWidth: null,
        plotShadow: false,
        type: 'pie'
    },
    credits: {
        enabled: false
    },
    title: {
        text: '2018年浏览器市场份额'
    },
    tooltip: {
        pointFormat: '{series.name}: <b>{point.percentage:.1f}%</b>'
    },
    legend: {
        layout: 'vertical',
        align: 'right',
        floating: true,
    },
    plotOptions: {
        pie: {
            allowPointSelect: true,
            cursor: 'pointer',
            dataLabels: {
                enabled: false
            },
            showInLegend: true
        }
    },
    series: [{
        name: 'Brands',
        colorByPoint: true,
        data: data
    }]
})
```

* 图示如下

> 浅克隆及后续处理后, data与tmp的对比

![](/images/highCharts3.png)

* 未对data进行浅克隆处理前的图表, 图例

![](/images/highCharts1.png)

* 对data进行浅克隆处理后的图表, 图例已按照legendIndex进行了排序

![](/images/highCharts2.png)

* THE END

