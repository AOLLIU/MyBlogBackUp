---
title: SDWebImage学习
date: 2015-3-10 19:36:16
tags:
    - iOS开发
    - OC
---


## SDWebImage的简单介绍:
- 优秀的图片缓存,下载框架,没有警告

## SDWebImage的详细介绍:
### 首先SDWebImage的实现原理
- SDWebImager核心是SDWebImageManger类
- SDWebImageManger  是两个类的集合体:
 - 一个负责下载网络图片,SDWebImageDownloader
 - 一个处理缓存的类SDImageCache

<!--more-->

- SDWebImage提供了如下三个category来进行缓存。
 - MKAnnotationView + WebCache            地图大头针
 - UIButton + WebCache                         给按钮设置图片
 - UIImageView + WebCache                     imageView的图片

### SDWebImage的常用方法介绍

- 在给tableView设置图片时可以用到SDWebImage中UIImageView + WebCache分类中下面一个重要的方法
`- (void)sd_setImageWithURL:(NSURL *)url placeholderImage:(UIImage *)placeholder options:(SDWebImageOptions)options progress:(SDWebImageDownloaderProgressBlock)progressBlock completed:(SDWebImageCompletionBlock)completedBlock;`
- 这个方法在以前是没有sd_前缀的,这个方法里面的参数很容易理解,这里重点介绍一下options这个参数:
这个参数是SDWebImageOptions类型的枚举类型,里面有10个选项,下面一一进行介绍
 -   ` SDWebImageRetryFailed = 1 << 0,  `    默认情况下，当一个 URL 下载失败，会将该 URL 放在黑名单中，不再尝试下载,当使用这个标记时就会 ,取消黑名单意思是即使下载失败也不会被加入黑名单
  - `  SDWebImageLowPriority = 1 << 1,  `    默认情况下，在 UI 交互时也会启动图像下载, 当时用这个标记时, 用户拖动tableView时不会进行图片的下载,当没有交互时才会下载
  - `SDWebImageCacheMemoryOnly = 1 << 2,`   此标记取消磁盘缓存,仅做内存缓存 
  - `  SDWebImageProgressiveDownload = 1 << 3,` 默认情况下，图像会在下载完成后一次性显示,  此标记允许渐进式下载，就像浏览器中那样，下载过程中，图像会逐步显示出来
  - `  SDWebImageRefreshCached = 1 << 4,  `    磁盘缓存将由 NSURLCache 处理，而不是 SDWebImage，这会对性能有轻微的影响, 此选项有助于处理同一个请求 URL 的图像发生变化,  如果缓存的图像被刷新，会调用一次 completion block，并传递最终的图像,  仅在无法使用嵌入式缓存清理参数确定图像 URL 时，使用此标记
  - `SDWebImageContinueInBackground = 1 << 5, ` 在 iOS 4+，当 App 进入后台后仍然会继续下载图像。这是向系统请求额外的后台时间以保证下载请求完成的, 如果后台任务过期，请求将会被取消
 - ` SDWebImageHandleCookies = 1 << 6, `     处理保存在 NSHTTPCookieStore 中的 cookies
 - ` SDWebImageAllowInvalidSSLCertificates = 1 << 7,`  允许不信任的 SSL 证书,  可以出于测试目的使用，在正式产品中慎用
  - `SDWebImageHighPriority = 1 << 8, `     默认情况下，图像会按照在队列中的顺序被加载，此标记会将它们移动到队列前部立即被加载,  而不是等待当前队列被加载，等待队列加载会需要一段时间
 - `  SDWebImageDelayPlaceholder = 1 << 9, `   默认情况下，在加载图像时，占位图像已经会被加载。而此标记会延迟加载占位图像，直到图像已经完成加载
  - ` SDWebImageTransformAnimatedImage = 1 << 10, ` 通常不会在可动画的图像上调用 transformDownloadedImage 代理方法，因为大多数转换代码会破坏动画文件,  使用此标记尝试转换

- 以最为常用的UIImageView为例:
 -  UIImageView+WebCache分类里面常用的方法:
`setImageWithURL :placeholderImage: options:`
这个方法默认情况下先显示 placeholderImage ，同时由SDWebImageManager 根据 URL 来在本地查找图片。

 - SDWebImageManager: `downloadWithURL:delegate:options:userInfo:`SDWebImageManager是将UIImageView+WebCache同SDImageCache链接起来的类，图片缓存是在内存缓存一份,在磁盘缓存一份.

 - SDImageCache：
  - ` queryDiskCacheForKey:delegate:userInfo:`用来从缓存根据`CacheKey`查找图片是否已经在内存缓存中,
  - 如果内存中已经有图片缓存`SDWebImageManager`会回调`SDImageCacheDelegate : imageCache:didFindImage:forKey:userInfo:`
而` UIImageView+WebCache `则回调`SDWebImageManagerDelegate: webImageManager:didFinishWithImage:`来显示图片。
  - 如果内存中没有图片缓存，那么生成 `NSInvocationOperation` 添加到队列，从硬盘查找图片是否已被下载缓存。根据 `URLKey` 在硬盘缓存目录下尝试读取图片文件。这一步是在 `NSOperation` 进行的操作，所以回主线程进行结果回调 `notifyDelegate:`。
  - 如果上一操作从硬盘读取到了图片，将图片添加到内存缓存中（如果空闲内存过小，会先清空内存缓存）。然后`SDImageCacheDelegate` 回调 `imageCache:didFindImage:forKey:userInfo:`。进而回调展示图片。
  - 如果从硬盘缓存目录没有读取到图片,说明所有缓存都不存在该图片，需要下载图片，回调 `imageCache:didNotFindImageForKey:userInfo:`。共享或重新生成一个下载器` SDWebImageDownloader` 开始下载图片。
  - 图片下载由 `NSURLConnection `来做，实现相关 `delegate` 来判断图片下载中、下载完成和下载失败。
  - `connection:didReceiveData: `中利用 `ImageIO `做了按图片下载进度加载效果。
  - `connectionDidFinishLoading:` 数据下载完成后交给 `SDWebImageDecoder` 做图片解码处理。
  - 图片解码处理在一个 `NSOperationQueue` 完成，不会拖慢主线程 UI。如果有需要对下载的图片进行二次处理，最好也在这里完成，效率会好很多。
 - 在主线程 `notifyDelegateOnMainThreadWithInfo: `宣告解码完成，`imageDecoder:didFinishDecodingImage:userInfo: `回调给 `SDWebImageDownloader`。
 - `imageDownloader:didFinishWithImage:` 回调给 `SDWebImageManager` 告知图片下载完成。
 - 通知所有的 `downloadDelegates `下载完成，回调给需要的地方展示图片。
 - 将图片保存到` SDImageCache `中，内存缓存和磁盘缓存同时进行保存。
 - 写文件到磁盘在单独 `NSInvocationOperation` 中完成，避免拖慢主线程。
 - 如果是在iOS上运行，`SDImageCache `在初始化的时候会注册`notification` 到 `UIApplicationDidReceiveMemoryWarningNotification` 以及 `UIApplicationWillTerminateNotification`,在内存警告的时候清理内存图片缓存，应用结束的时候清理过期图片。

 - ` SDWebImagePrefetcher `可以预先下载图片，方便后续使用。

- 下面是SDWebImage在清理图片缓存时的原理
 - 使用 SDWebImage 设置cell图片(网上下载)的注意点
 - 当程序收到内存警告的时候就会调用下面的方法
![Snip20160529_2.png](http://upload-images.jianshu.io/upload_images/1494773-603db80825b5c6f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- SDWebImage在cleanMemary(清除缓存)时的运行原理
![Snip20160529_3.png](http://upload-images.jianshu.io/upload_images/1494773-41f2b5a726ec7dd0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### SDWebImage可以做的事情:
- 1,给imageView设置图片
- 2,单独下载图片,可以监听下载进度
- 3,管理图片缓存
- 4,设置gif图片
- 5,判断图片是什么类型的

## 细节:
- 1.clear和clean的区别
 - clear:直接把之前的缓存文件删除,然后重新创建一个空的
 - clean:删除过期的缓存,然后计算剩余的缓存大小,和设置的最大缓存大小比较,如果发现超过那么就会继续删除,按照文件的 创建时间来删除(由远到近)
- 2.默认的缓存时间:KDefaultCacheMaxCacheAge 1week
- 3.播放gif `#import<ImgeIO/ImageIO.h>`
- 4.判断图片的类型:得到图片的二进制数据,然后判断第一个字节
- 5.请求的超时时间:downloadTimeOut 15s
- 6.内部缓存处理模式:二级缓存处理,内存缓存和磁盘缓存
- 7.磁盘缓存的路径:`~/Library/Caches/default/com.hackemist.SDWebImageCache.default/文件的名称`
- 8.得到图片的URL地址,然后对URL进行加密处理(MD5)

```objc
    //  abe7c97cjw8ermn0v2x7nj20k00k0jrz.jpg
    //  36c0b4da16d5ee121d4adf6c3b4d93b6.jpg
    //  $ echo -n "abe7c97cjw8ermn0v2x7nj20k00k0jrz" | md5
```
- 9.最大并发数量: 6
- 10.队列中任务默认的处理方式是:`SDWebImageDownloaderFIFOExecutionOrder`先进先出,可以通过给图片现在设置依赖,来实现LIFO
- 11.内存的处理方式:NSCache
- 12.下载图片的方式:NSURLConnection来发送请求,设置代理下载图片
- 13,内存警告:该框架内部采用监听系统通知的方式来处理内存警告问题,会自动的清空缓存
- 14.key--URL(如何优化),黑名单(当一个URL请求事变后,会被添加到黑名单中,可以有效的防止一个错误的URL被多次尝试下载)



