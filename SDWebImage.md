# SDWebImage
This library provides a category for UIImageView with support for remote images coming from the web.  
对`UIImageView`的拓展，支持直接从网络获取图片并显示。

#### An `UIImageView` category adding web image and cache management to the Cocoa Touch framework
    @interface UIImageView (WebCache)
    
    - (NSURL *)sd_imageURL;
    
    - (void)sd_setImageWithURL:(NSURL *)url;
    
    - (void)sd_setImageWithURL:(NSURL *)url
                       options:(SDWebImageOptions)options;
    
    - (void)sd_setImageWithURL:(NSURL *)url
                     completed:(SDWebImageCompletionBlock)completedBlock;
    
    - (void)sd_setImageWithURL:(NSURL *)url
                       options:(SDWebImageOptions)options
                     completed:(SDWebImageCompletionBlock)completedBlock;
    
    - (void)sd_setImageWithURL:(NSURL *)url
                       options:(SDWebImageOptions)options
                      progress:(SDWebImageDownloaderProgressBlock)progressBlock
                     completed:(SDWebImageCompletionBlock)completedBlock;
    
    - (void)sd_cancelCurrentImageLoad;
    
    @end
    
使用 category 向`UIImageView`中添加了以上方法。

---
## SDWebImageOptions
    typedef NS_OPTIONS(NSUInteger, SDWebImageOptions) {
        /**
         *默认下载失败后，URL将被加入黑名单。
         *此选项禁用黑名单。
         */
         
        SDWebImageRetryFailed = 1 << 0,
    
        /**
         * By default, image downloads are started during UI interactions, this flags disable this feature,
         * leading to delayed download on UIScrollView deceleration for instance.
         */
        SDWebImageLowPriority = 1 << 1,
    
        /**
         * 禁用磁盘缓存
         */
        SDWebImageCacheMemoryOnly = 1 << 2,
    
        /**
         * 启动分步下载, 图片将在下在途中逐渐显示.
         * 默认情况下，图片将在加载完成后显示
         */
        SDWebImageProgressiveDownload = 1 << 3,
    
        /**
         * Even if the image is cached, respect the HTTP response cache control, and refresh the image from remote location if needed.
         * The disk caching will be handled by NSURLCache instead of SDWebImage leading to slight performance degradation.
         * This option helps deal with images changing behind the same request URL, e.g. Facebook graph api profile pics.
         * If a cached image is refreshed, the completion block is called once with the cached image and again with the final image.
         *
         * Use this flag only if you can't make your URLs static with embedded cache busting parameter.
         */
        SDWebImageRefreshCached = 1 << 4,
    
        /**
         * 向系统获取更长的后台时间用于获取图片
         */
        SDWebImageContinueInBackground = 1 << 5,
    
        /**
         * 处理储存在NSHTTPCookieStore中的cookies
         */
        SDWebImageHandleCookies = 1 << 6,
    
        /**
         * 允许不被信任的SSL连接
         * 用于测试 Use with caution in production.
         */
        SDWebImageAllowInvalidSSLCertificates = 1 << 7,
    
        /**
         * 图片默认按照顺序加载。
         * 使用此选项将此图片移动到等待队列头部
         */
        SDWebImageHighPriority = 1 << 8,
        
        /**
         * 默认情况下，placeholder优先于image加载
         * 此选项延迟加载placeholder到image加载完毕
         */
        SDWebImageDelayPlaceholder = 1 << 9,
    
        /**
         * We usually don't call transformDownloadedImage delegate method on animated images,
         * as most transformation code would mangle it.
         * Use this flag to transform them anyway.
         */
        SDWebImageTransformAnimatedImage = 1 << 10,
        
        /**
         * 默认情况下，图片在下载完成后被加载到
         * 某些情况下需要在图片下载完成后进行处理
         * 使用此选项手动在下载成功后在completedBlock中加载图片
         */
        SDWebImageAvoidAutoSetImage = 1 << 11
    };
---

    - (NSURL *)sd_imageURL {
        return objc_getAssociatedObject(self, &imageURLKey);
    }
使用了`objc_getAssociatedObject`方法获取与self关联的对象。

    - (void)sd_setImageWithURL:(NSURL *)url placeholderImage:(UIImage *)placeholder options:(SDWebImageOptions)options progress:(SDWebImageDownloaderProgressBlock)progressBlock completed:(SDWebImageCompletionBlock)completedBlock {
        [self sd_cancelCurrentImageLoad];
        objc_setAssociatedObject(self, &imageURLKey, url, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    
        if (!(options & SDWebImageDelayPlaceholder)) {
            dispatch_main_async_safe(^{
                self.image = placeholder;
            });
        }
        
        if (url) {
    
            // check if activityView is enabled or not
            if ([self showActivityIndicatorView]) {
                [self addActivityIndicator];
            }
    
            __weak __typeof(self)wself = self;
            id <SDWebImageOperation> operation = [SDWebImageManager.sharedManager downloadImageWithURL:url options:options progress:progressBlock completed:^(UIImage *image, NSError *error, SDImageCacheType cacheType, BOOL finished, NSURL *imageURL) {
                [wself removeActivityIndicator];
                if (!wself) return;
                dispatch_main_sync_safe(^{
                    if (!wself) return;
                    if (image && (options & SDWebImageAvoidAutoSetImage) && completedBlock)
                    {
                        completedBlock(image, error, cacheType, url);
                        return;
                    }
                    else if (image) {
                        wself.image = image;
                        [wself setNeedsLayout];
                    } else {
                        if ((options & SDWebImageDelayPlaceholder)) {
                            wself.image = placeholder;
                            [wself setNeedsLayout];
                        }
                    }
                    if (completedBlock && finished) {
                        completedBlock(image, error, cacheType, url);
                    }
                });
            }];
            [self sd_setImageLoadOperation:operation forKey:@"UIImageViewImageLoad"];
        } else {
            dispatch_main_async_safe(^{
                [self removeActivityIndicator];
                if (completedBlock) {
                    NSError *error = [NSError errorWithDomain:SDWebImageErrorDomain code:-1 userInfo:@{NSLocalizedDescriptionKey : @"Trying to load a nil url"}];
                    completedBlock(nil, error, SDImageCacheTypeNone, url);
                }
            });
        }
    }
    
加载图片的万能方法。

1. 取消当前加载中的image
2. 将URL绑定到当前对象上
3. 判断是否启用延迟加载placeholder选项，若否，主线程加载placeholder
4. 检查showActivityIndicatorView，若是，显示indicatorView
5. 调用`SDWebImageManager`中的`downloadImageWithURL`方法下载图片
6. 移除indicatorView
7. 切换到主线程，使用`if (!wself) return`判断下载完成后引用是否有效，否则函数退出
8. 判断`SDWebImageAvoidAutoSetImage`是否启用，若是，调用completedBlock
9. 判断图片是否存在，若是，设置imageView的图片为image。否则判断`SDWebImageDelayPlaceholder`是否启用，若是，设置imageView的图片为placeholder
10. 如果completedBlock存在，调用completedBlock