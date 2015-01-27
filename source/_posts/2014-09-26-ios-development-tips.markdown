---
layout: post
title: "开发tips"
date: 2014-09-26 14:43:13 +0800
comments: true
categories: iOS
tags: [iOS]
keywords: iOS技巧，iOS兼容
description: iOS6兼容
---
1.uitableview 在ios6和ios7以上布局的问题，frame.y要么多出20pt要么少了20pt，然后就是到处用到UITableView
的地方都去判断 if(ios7_Or_Later) 之类的，这样可以解决只是同样的代码写的到处都是了。与此同时我发现
了一个很好解决此问题的办法，就是反射，如果uitableview都是从xib或者storyboard中实例化出来的，在
viewDidLoad执行之前已经实例化好了，这时候如果我们有一个共有的BaseViewController,那就回先执行
BaseViewController中的ViewDidLoad,那这样就可以拿到当前实例中的属性，（注意uitableview 作为IBOutlet
进行于界面关联。）拿到了uitableview，那就可以对它进行一些布局的调整，以修正偏移的问题。
代码片段：


``` obj-c
-(void)fixTableView
{
    if (!IOS7_OR_LATER) {
        unsigned int outCount, i;
        objc_property_t *properties = class_copyPropertyList([self class], &outCount);
        for (i = 0; i < outCount; i++)
        {
            objc_property_t property = properties[i];
            NSString *propName = [NSString stringWithUTF8String:property_getName(property)];
            //正对uitableview对ios7以下版本适配，controller中的tableview必须命名为tableView
            if ([propName isEqualToString:@"tableView"]) {
                //fprintf(stdout, "%s %s\n", property_getName(property), property_getAttributes(property));
                id value=[self valueForKey:propName];
                UITableView *t=value;
                CGRect f=t.frame;
                f.origin.y=f.origin.y+20;
                f.size.height=f.size.height-20;
                t.frame=f;
            }
        }
    }
}
```
