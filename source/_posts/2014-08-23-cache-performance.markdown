---
layout: post
title: "缓存与性能"
date: 2014-08-23 23:50:00 +0800
comments: true
categories: iOS
tags: [iOS]
keywords: 缓存, iOS缓存，性能优化
description: iOS如何设计缓存，模型缓存，URL缓存
---
缓存，一词对做过web开发的开发者来说也许很熟悉（有些开发者可能没有关注过），其目的都是为了提升应用的
性能而采取的一种技术；如今已经是无缓存不架构，在从事web开发的初期我也很少关注缓存，到自己架构项目后
设计缓存框架才意识到缓存怎么才能更好的为我们的应用带来更好的体验。  
####移动客户端的缓存如何设计？  
1. #####按需缓存  
和浏览器一样，将每一次访问的请求缓存，缓存的方式根据http1.1的缓存控制头决定。  但实际开发中可能不
完全依赖于服务器，大多都是在客户端进行操作，请求从服务器获取到内容后，以请求的URL以某种格式编码后作
为名称，并将内容存储在本地文件系统，之后每次对于同样的URL请求，先检查缓存中是否存在，只有当数据不存
在（或者过期）的情况下才从服务器获取数据。

2. #####预缓存  
这种情况是缓存全部内容，以便离线浏览。预缓存最常见都是存储在sqlite数据库和core data中。
预缓存的应用场景，比如网易的新闻可以离线阅读，这样就是通过点击离线缓存，将需要看的内容预先缓存后在无
网络下进行阅读；再有很多稍后阅读类应用都是使用预缓存。（废话太多。。）
但当我想起11年开发的一款应用时，原来我们用的就是预缓存，预先缓存其实可以理解为一个数据同步过程，将远
程服务器上的数据同步到本地，当时做的就是这样一种数据更新机制。  


目前很多客户端都是按需缓存，例如QQ，微博等。  
####缓存什么？
网络应用开发，缓存的当然是耗时耗电的网络加载的数据了，只要是网络请求所获得的数据都可以考虑缓存，例如
一个典型的案例：点击某个用户的头像查看用户首页，不能每次点击都及时从网络加载吧，假如访问用户首页的
url：http://user.xxx.com/12346 那第一次访问即可将用户的信息和用户的头像通过访问的url进行缓存
（头像的url肯定不是一样的）。  
所以缓存就是在缓存网络请求的文本数据和图片数据。就目前api交互的数据格式都是以json或者xml，*我们要缓存
json格式的数据和图片即可达到我们的目的。*  

  基于前面的讨论，现在以一些代码来说明，找了一个典型的图片缓存库来把玩一下（不是教怎么用缓存库，
  不在此篇幅范围）；[SDWebImage](https://github.com/rs/SDWebImage "SDWebImage")。  
  SDWebImage 就是按需缓存的，设置一个固定的缓存过期时间，每次请求先检查缓存，如果过期或则不存在才
  从远程服务器获取。
  https://github.com/rs/SDWebImage/tree/master/SDWebImage 打开它的库一看，哇靠，这么多
  文件
  <img src="/assets/images/article/20140824-1.png"  width="800" />  
  其实从文件名都能看出个大概，类别扩展先看UIImageView+WebCache.h 其他暂时不关心，常用的方法设
  置一个图片的url和一个默认图片：  

``` objc
/**
 * Set the imageView `image` with an `url` and a placeholder.
 *
 * The download is asynchronous and cached.
 *
 * @param url         The url for the image.
 * @param placeholder The image to be set initially, until the image request finishes.
 * @see sd_setImageWithURL:placeholderImage:options:
 */
- (void)sd_setImageWithURL:(NSURL *)url placeholderImage:(UIImage *)placeholder;
```  
按他的注释我们知道它是一个异步下载并缓存，其实这样就够了；  
在看看SDImageCache.h 、SDWebImageManager.h  重点在这里，SDImageCache.h 有一个单例方法，初
始化一个默认的图片缓存路径在Library/Caches下com.hackemist.SDWebImageCache. 是默认的文件夹名
称，同时也作为内存中缓存的名字，然后还向通知中心注册了三个通知：

``` obj-c
#if TARGET_OS_IPHONE
//当内存警告时清理内存
        [[NSNotificationCenter defaultCenter] addObserver:self
                                                 selector:@selector(clearMemory)
                                                     name:UIApplicationDidReceiveMemoryWarningNotification
                                                   object:nil];
//程序将要结束时清理磁盘过期文件
        [[NSNotificationCenter defaultCenter] addObserver:self
                                                 selector:@selector(cleanDisk)
                                                     name:UIApplicationWillTerminateNotification
                                                   object:nil];
//当程序进入后台时清理磁盘缓存的过期文件
        [[NSNotificationCenter defaultCenter] addObserver:self
                                                 selector:@selector(backgroundCleanDisk)
                                                     name:UIApplicationDidEnterBackgroundNotification
                                                   object:nil];
#endif  
```

SDWebImageManager.h 最核心函数：
``` obj-c
- (id <SDWebImageOperation>)downloadImageWithURL:(NSURL *)url
                                         options:(SDWebImageOptions)options
                                        progress:(SDWebImageDownloaderProgressBlock)progressBlock
                                       completed:(SDWebImageCompletionWithFinishedBlock)completedBlock;

```
这里就是处理请求和缓存的函数，逻辑就是根据设置的缓存策略进行请求并缓存，然后将图片同步回主线程。  
好了，看了SDWebImage的缓存实现我们是不是应该参考和借鉴一下它的思路设计一个满足我们业务需求的缓存实现
当然图片缓存就可以用他的，如果图片缓存和业务数据缓存都用同一套代码就更好了。  
这里是我参考SDWebImage写的一个小demo  [点这里](https://github.com/alienjun/MyCacheTest "MyCacheTest")，欢迎拍砖，还望不吝啬的支教。


  原创文章，版权声明:自由转载-非商用-非衍生-保持署名.
