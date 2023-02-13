
### SDWebImageManager核心类介绍
**SDWebImageManager**是一个单例, **SDWebImageManager** 用于支撑 **WebCache** 分类加载图片,连接 **SDwebImageDownloader** 异步加载器和 **SDWebImageCache** 缓存模块
主要包含
 - id<SDWebImageManagerDelegate> delegate
 - SDwebImageDownloader //异步下载器
 - SDWebImageCache // 缓存模块

SDWebImageManagerDelegate中包含了三个方法
```Objective-C
- (BOOL)imageManager:(nonnull SDWebImageManager *)imageManager shouldDownloadImageForURL:(nullable NSURL *)imageURL;
- (BOOL)imageManager:(nonnull SDWebImageManager *)imageManager shouldBlockFailedURL:(nonnull NSURL *)imageURL withError:(nonnull NSError *)error;
- (nullable UIImage *)imageManager:(nonnull SDWebImageManager *)imageManager transformDownloadedImage:(nullable UIImage *)image withURL:(nullable NSURL *)imageURL;
```
第一个方法用于询问代理在找不到缓存情况下,是否根据url下载图片.默认返回为yes.如果返回为NO,缓存中没有查找到也不下载
第二个方法用于询问代理是当下载出现错误的时候是否将URL添加到failedURLs中,默认为yes
第三个在下载之后，缓存之前转换图片。在全局队列中操作，不阻塞主线程

### 下载模块SDWebImageDownloader
包含了 **SDWebImageDownloaderOperation** 和 **SDWebImageDownloader**
**SDWebImageDownloaderOperation** 配置最大并发(默认6个)、超时时间(15s)
```Objective-C
- (nonnull instancetype)initWithSessionConfiguration:(nullable NSURLSessionConfiguration *)sessionConfiguration {
    if ((self = [super init])) {
        _operationClass = [SDWebImageDownloaderOperation class];
        _shouldDecompressImages = YES;
        _executionOrder = SDWebImageDownloaderFIFOExecutionOrder;
        _downloadQueue = [NSOperationQueue new];
        _downloadQueue.maxConcurrentOperationCount = 6;
        _downloadQueue.name = @"com.hackemist.SDWebImageDownloader";
        _URLOperations = [NSMutableDictionary new];
        _operationsLock = dispatch_semaphore_create(1);
        _headersLock = dispatch_semaphore_create(1);
        _downloadTimeout = 15.0;

        [self createNewSessionWithConfiguration:sessionConfiguration];
    }
    return self;
}
```
**SDWebImageDownloader** 下载

```Objective-C
- (nullable SDWebImageDownloadToken *)downloadImageWithURL:(nullable NSURL *)url
                                                   options:(SDWebImageDownloaderOptions)options
                                                  progress:(nullable SDWebImageDownloaderProgressBlock)progressBlock
                                                 completed:(nullable SDWebImageDownloaderCompletedBlock)completedBlock;
```

### 图片缓存
SDImageCache单例中主包含是 **SDImageCacheConfig**、**SDMemoryCache** 、**SDDiskCache**三个模块

 - SDImageCacheConfig（单利）:用来设置缓存的策略。如磁盘缓存大小（无限制）、内存缓存大小（无限制）、磁盘缓存时间（默认7天）、是否允许内存缓存、是否进入后台自动清除过期磁盘数据等
 - SDMemoryCache: SDMemoryCache继承自NSCache，对内存缓存进行增、删、查。
 - SDDiskCache: 磁盘缓存。diskCache中可以根据key来设置和读取磁盘数据，删除过期数据等。key是url+其他image额外的信息进行16位的MD5加密。

### 缓存清理时机
 - UIApplicationWillTerminateNotification程序将要杀死时候，会调用deleteOldFilesWithCompletionBlock，通过diskCache来清理磁盘缓存
 - UIApplicationDidEnterBackgroundNotification程序进入后台，也会调用deleteOldFilesWithCompletionBlock清理磁盘缓存
 - didReceiveMemoryWarning监听内存警告，收到警告后清理内存缓存

### 清理磁盘的策略
 - 先获取当前磁盘目录，遍历当前磁盘目录URL，查找磁盘数据中所有修改日期超过最大磁盘缓存时间的数据，清除超时数据
 - 在遍历磁盘数据的时候，更新未超时的本地磁盘占用大小；如果最后超过了规定的磁盘占用大小，对磁盘文件的修改时间进行排序，然后for循环中移除最老的数据，直到占用磁盘总大小小于设置的磁盘最大占用


### SDWebImage图片加载流程 
简单来说：
1. 调用setImageWithURL，会先显示默认的图片（placeholderImage），然后根据URL开始处理图片
2. 交给 SDImageCache 从缓存查找图片是否已经下载，如果存在，则直接显示出来
3. 如果内存缓存中没有，生成 NSInvocationOperation 添加到队列开始从硬盘查找图片是否已经缓存。根据 URLKey 在硬盘缓存目录下尝试读取图片文件，如果读取到了图片，将图片添加到内存缓存中，再显示出来
4. 如果内存和硬盘都没有该图片，则需要下载图片，共享或重新生成一个下载器 SDWebImageDownloader 开始下载图片
5. 数据下载完成后交给 SDWebImageDecoder 做图片解码处理。图片解码处理在一个 NSOperationQueue 完成，不会拖慢主线程 UI
6. 回调给 SDWebImageManager 告知图片下载完成，通知所有的 downloadDelegates 下载完成，回调给需要的地方展示图片
7. 将图片保存到 SDImageCache 中，内存缓存和硬盘缓存同时保存。写文件到硬盘也在以单独 NSInvocationOperation 完成，避免拖慢主线程
