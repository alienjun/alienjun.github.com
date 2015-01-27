---
layout: post
title: "CLLocationManager定位在iOS8上该注意的"
date: 2014-09-21 21:45:06 +0800
comments: true
categories: iOS
tags: [iOS]
keywords: 缓存, iOS缓存，性能优化
description: iOS如何设计缓存，模型缓存，URL缓存
---
今天突然发现之前一直正常的定位服务在iOS8上不能用了，我用的是一个开源的[组件](https://github.com/smallqiang/MMLocationManager "MMLocationManager")。
此组件实现的是一个MKMapView号称定位很准，也没有仔细去究是否定位精确。

现在iOS8上对地理位置处理重新多了一些小的设计，估计是为了兼用一些穿戴的智能设备，看到有两个函数的介绍

``` obj-c
//使用时可以使用定位
– requestWhenInUseAuthorization
//后台总是可以使用定位
– requestAlwaysAuthorization
```

很明显就是为了协处理准备的，iOS8使用apple自己的定位服务：
plist文件中增加key: NSLocationWhenInUseUsageDescription
value:定位的时候弹出提示权限附带显示的message。 所以在代码中就需要根据plist配置的方式进行调用api
[_locationManager requestWhenInUseAuthorization];

demo：[点这里](https://github.com/alienjun/CLLocationManager-iOS8 "CLLocationManager-iOS8")

原创文章，版权声明:自由转载-非商用-非衍生-保持署名.
