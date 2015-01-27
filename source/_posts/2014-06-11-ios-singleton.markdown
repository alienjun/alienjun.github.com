---
layout: post
title: "iOS中多种单例写法"
date: 2014-06-11 20:27:06 +0800
comments: true
categories: iOS
tags: [iOS]
keywords: 单例模式, iOS单例模式
description: iOS中使用单例模式
---
Singleton
单例模式在整个程序生命周期内，该对象只有一份存在内存中，就是C中的全局变量
在实际开发中经常会有用到单例模式，记录一下oc中使用单例模式的几种写法。
在oc中通常默认以default、shared、current开头命名的方法。
``` obj-c
static Broadcast *b;

//第一种
+(Broadcast *)sharedBroadcast{
    if (b==nil) {
       b=[[self alloc]init];
    }
    return b;
}

//第二种，线程完全
+(Broadcast *)sharedBroadcast{
    @synchronized(self){
        if (b==nil) {
            b=[[self alloc]init];
        }
    }
    return b;
}
//第三种，线程安全，使用GCD
+(Broadcast *)sharedBroadcast{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        if (b==nil) {
            b=[[self alloc]init];
        }
    });
    return b;
}
//第四种，重写initialize方法
+(void) initialize{
    static BOOL initalized=NO;
    if (!initalized) {
        initalized=YES;
        b=[[self alloc]init];
    }
}
```
