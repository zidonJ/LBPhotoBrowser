# LBPhotoBrowser

一行代码即可搞定下面效果的浏览器.  简书地址:[http://www.jianshu.com/p/00f4b7b20dc4](http://www.jianshu.com/p/00f4b7b20dc4)(详细说明)
```
LBPhotoBrowser对gif的播放提供了两种方式(详细见简书):

(1)采用系统的 + (nullableUIImage*)animatedImageWithImages:(NSArray *)images duration:(NSTimeInterval)durationNS_AVAILABLE_IOS(5_0);

(2)自定义gif的播放,具体步骤如下：

   * 获取当前手机可以利用的内存和当前展示的gif图片每帧图片加载到内存占用的大小,以取得当前内存可以加载gif的最大帧数.
     最大加载帧数 = 可利用内存 /  每帧图片的大小.
     
   * 使用CADisplayLink作为定时器,开始展示当前帧的图片
   
   * 获取当前帧的展示时间,展示完毕,切换下一帧图片.当在展示当前帧的图片的时候, 异步线程(自定义NSOperation)去取下一帧的图片,以供当前帧的图片展示
     完毕后,直接从缓存的buffer（字典）中读取.
     
   * 当gif图片的帧数大于当前内存适合加载的帧数的时候,buffer(字典)会不断的移除已展示过的图片,来确保加载到内存中的图片数稳定.
     如果小于可加载的最大帧数,直接全部加载到内存,节省CPU.
     
   * LBPhotoBrowser为了保证较低的CPU消耗,即使在图片浏览器加载多张gif的时候,也会保证同一时间内,只会对一张gif进行处理,不会同时去解压多张gif图片.
   
     demo中（采用方式2）加载的gif包含了144张图,仍然保持较低内存和cpu,借鉴了YYImage的播放方式
```
通过`LBPhotoBrowserManager`的`lowGifMemory`属性控制.

效果图如下: 

![](https://github.com/tianliangyihou/zhuxian/blob/master/2306467-14a8a6771dad3b5c.gif?raw=true)


# 使用(usage)

实现上面的效果,只需调用下面这行代码:
```obj-c

/**
 展示图片
 @param urls 需要加载的图片的URL数组
 @param imageViews 传入需要大图显示的imageViews 因为将来需要在对应的地方imageView用动画消除掉,主要是取imageView的frame
 @param index 点击图片的index
 @param superView 当前View的父View
 */
- (void)showImageWithURLArray:(NSArray *)urls fromImageViews:(NSArray *)imageViews andSelectedIndex:(int)index andImageViewSuperView:(UIView *)superView;

Example:

   [[LBPhotoBrowserManager defaultManager] showImageWithURLArray:_urls fromImageViews: _imageViews andSelectedIndex:(int)tap.view.tag andImageViewSuperView:self.view];
   
 ```
如果需要添加长按手势,可以采用默认的(类似微信的)

```obj-c

// 添加默认的长按控件
- (instancetype)addLongPressShowTitles:(NSArray <NSString *>*)titles;
// 默认长按控件的回调
- (instancetype)addTitleClickCallbackBlock:(void(^)(UIImage *image,NSIndexPath *indexPath,NSString *title))titleClickCallBackBlock;

Example:
// 添加长按手势的效果 
    [[[LBPhotoBrowserManager defaultManager] addLongPressShowTitles:self.titles] addTitleClickCallbackBlock:^(UIImage *image, NSIndexPath *indexPath, NSString *title) {
        LBPhotoBrowserLog(@"%@ %@ %@",image,indexPath,title);
    }]
   
 ```
 如果需要自定义长按手势的弹出框,实现下面这个Bock即可
 ```obj-c
// 添加自定义的长按控件
- (instancetype)addLongPressCustomViewBlock:(UIView *(^)(UIImage *image, NSIndexPath *indexPath))longPressBlock;
 ```
 
 如果需要自定义每张图片加载的占位图,实现下面这个Block,否则采用默认的站位图LBLoading.png
  ```obj-c
// 为每张图片添加占位图
- (instancetype)addPlaceHoldImageCallBackBlock:(UIImage *(^)(NSIndexPath * indexPath))placeHoldImageCallBackBlock;

Example:
     // 给每张图片添加占位图
    [[LBPhotoBrowserManager defaultManager] addPlaceHoldImageCallBackBlock:^UIImage *(NSIndexPath *indexPath) {
        LBPhotoBrowserLog(@"%@",indexPath);
        return [UIImage imageNamed:@"LBLoading.png"];
    }];
 ```
 
 有时候你也可以这么写:
 ```obj-c
   
  [[[[LBPhotoBrowserManager defaultManager] addLongPressShowTitles:self.titles] addTitleClickCallbackBlock:^(UIImage *image, NSIndexPath *indexPath, NSString *title) {
        LBPhotoBrowserLog(@"%@ %@ %@",image,indexPath,title);
    }]addPlaceHoldImageCallBackBlock:^UIImage *(NSIndexPath *indexPath) {
        return [UIImage imageNamed:@"LBLoading.png"];
    }].lowGifMemory = YES;
 
```

除此之外:LBPhotoBrowser对图片放大到超过屏幕尺寸时候 拖动的消失方式,也提供了两种方式`LBMaximalImageViewOnDragDismmissStyle`默认的`LBMaximalImageViewOnDragDismmissStyleOne`

