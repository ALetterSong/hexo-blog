title: Android Universal Image Loader 源码分析
date: 2015-11-15 13:34:28
tags:
---
    （不保证本文所述完全正确,参考资料包括但不限于UIL的github主页和codeKK,trinea的开源项目解析）

Android上有众多的图片加载库,市面上知名的图片加载库有 [UIL](https://github.com/nostra13/Android-Universal-Image-Loader) , [Picasso](https://github.com/square/picasso)  , [Fresco](https://github.com/facebook/fresco) , [Glide](https://github.com/bumptech/glide) , [Volley](https://android.googlesource.com/platform/frameworks/volley/)  等。它们各有特色，选择合适的图片加载库非常重要。

**1. 四大图片加载库基本信息**
![流程图](http://www.trinea.cn/wp-content/uploads/2015/10/image-cache-compare-before.jpeg)

**Universal Image Loader** 是很早开源的图片缓存，在早期被很多应用使用。
**Picasso** 是 Square 开源的项目，且他的主导者是 JakeWharton，所以广为人知。
**Glide** 是 Google 员工的开源项目，被一些 Google App 使用，在去年的 Google I/O 上被推荐，不过目前国内资料不多。
**Fresco** 是 Facebook 在今年上半年开源的图片缓存，主要特点包括：
(1) 两个内存缓存加上 Native 缓存构成了三级缓存
(2) 支持流式，可以类似网页上模糊渐进式显示图片
(3) 对多帧动画图片支持更好，如 Gif、WebP

**2. Universal Image Loader基本特点**
Android Universal Image Loader 是一个强大的、可高度定制的图片缓存，本文涉及的UIL版本为1.9.4。
简单的说 UIL 就做了一件事——获取图片并显示在相应的控件上。

 - 基本特点

> * 可配置度高。支持任务线程池、下载器、解码器、内存及磁盘缓存、显示选项等等的配置。
> * 包含内存缓存和磁盘缓存两级缓存。
> * 支持多线程，支持异步和同步加载。
> * 支持多种缓存算法、下载进度监听、ListView 图片错乱解决等。

 - 总体设计图

![总体设计图](https://raw.githubusercontent.com/android-cn/android-open-project-analysis/master/tool-lib/image-cache/universal-image-loader/image/overall-design.png)

简单的讲就是`ImageLoader`收到加载及显示图片的任务，并将它交给`ImageLoaderEngine`，`ImageLoaderEngine`分发任务到具体线程池去执行，任务通过`Cache`及`ImageDownloader`获取图片，中间可能经过`BitmapProcessor`和`ImageDecoder`处理，最终转换为`Bitmap`交给`BitmapDisplayer`在`ImageAware`中显示。

 - 流程图

![流程图](https://raw.githubusercontent.com/android-cn/android-open-project-analysis/master/tool-lib/image-cache/universal-image-loader/image/uil-flow.png)


- 核心类
*1. ImageLoader.java*

主要函数：
(1) getInstance()
```java
/**
得到ImageLoader的单例。通过双层是否为 null 判断提高性能。
双重检验锁模式（double checked locking pattern），是一种使用同步块加锁的方法，
因为会有两次检查 instance == null，一次是在同步块外，一次是在同步块内。
为什么在同步块内还要再检验一次？因为可能会有多个线程一起进入同步块外的 if，如果在同步块内不进行二次检验的话就会生成多个实例了。
**/
public static ImageLoader getInstance() {
        if(instance == null) {
            Class var0 = ImageLoader.class;
            synchronized(ImageLoader.class) {
                if(instance == null) {
                    instance = new ImageLoader();
                }
            }
        }

        return instance;
    }
```
[单例模式](http://www.trinea.cn/java/singleton/)

(2) init(ImageLoaderConfiguration configuration)
初始化配置参数，参数`configuration`为`ImageLoader`的配置信息，包括图片最大尺寸、任务线程池、磁盘缓存、下载器、解码器等等。
实现中会初始化`ImageLoaderEngine engine`属性，该属性为任务分发器。
(3) displayImage(String uri, ImageAware imageAware, DisplayImageOptions options, ImageLoadingListener listener, ImageLoadingProgressListener progressListener)
加载并显示图片或加载并执行回调接口。`ImageLoader` 加载图片主要分为三类接口：
`displayImage(…)` 表示异步加载并显示图片到对应的ImageAware上。
`loadImage(…)` 表示异步加载图片并执行回调接口。
`loadImageSync(…)` 表示同步加载图片。
以上三类接口最终都会调用到这个函数进行图片加载。函数参数解释如下：
**uri**: 图片的 uri。uri 支持多种来源的图片，包括 http、https、file、content、assets、drawable 及自定义，具体介绍可见ImageDownloader。
**imageAware**: 一个接口，表示需要加载图片的对象，可包装 View。
**options**: 图片显示的配置项。比如加载前、加载中、加载失败应该显示的占位图片，图片是否需要在磁盘缓存，是否需要在内存缓存等。
**listener**: 图片加载各种时刻的回调接口，包括开始加载、加载失败、加载成功、取消加载四个时刻的回调函数。
**progressListener**: 图片加载进度的回调接口。
```java
public void displayImage(String uri, ImageAware imageAware, DisplayImageOptions options, ImageLoadingListener listener, ImageLoadingProgressListener progressListener) {
        this.checkConfiguration();
        if(imageAware == null) {
            throw new IllegalArgumentException("Wrong arguments were passed to displayImage() method (ImageView reference must not be null)");
        } else {
            if(listener == null) {
                listener = this.defaultListener;
            }

            if(options == null) {
                options = this.configuration.defaultDisplayImageOptions;
            }
//url为空
            if(TextUtils.isEmpty(uri)) {
                this.engine.cancelDisplayTaskFor(imageAware);
                listener.onLoadingStarted(uri, imageAware.getWrappedView());
                if(options.shouldShowImageForEmptyUri()) {
                    imageAware.setImageDrawable(options.getImageForEmptyUri(this.configuration.resources));
                } else {
                    imageAware.setImageDrawable((Drawable)null);
                }

                listener.onLoadingComplete(uri, imageAware.getWrappedView(), (Bitmap)null);
            } else {
            //计算Bitmap的大小，以便后面解析图片时用
                ImageSize targetSize = ImageSizeUtils.defineTargetSizeForView(imageAware, this.configuration.getMaxImageSize());
                String memoryCacheKey = MemoryCacheUtils.generateKey(uri, targetSize);
                this.engine.prepareDisplayTaskFor(imageAware, memoryCacheKey);
                listener.onLoadingStarted(uri, imageAware.getWrappedView());
                //在内存缓存中是否存在
                Bitmap bmp = this.configuration.memoryCache.get(memoryCacheKey);
                ImageLoadingInfo imageLoadingInfo;
                //如果bitmap不为空并且没被回收
                if(bmp != null && !bmp.isRecycled()) {
                    L.d("Load image from memory cache [%s]", new Object[]{memoryCacheKey});
                    //是否需要后续处理
                    if(options.shouldPostProcess()) {
                    //新建ProcessAndDisplayImageTask任务
                        imageLoadingInfo = new ImageLoadingInfo(uri, imageAware, targetSize, memoryCacheKey, options, listener, progressListener, this.engine.getLockForUri(uri));
                        ProcessAndDisplayImageTask displayTask1 = new ProcessAndDisplayImageTask(this.engine, bmp, imageLoadingInfo, defineHandler(options));
                        //是否同步加载
                        if(options.isSyncLoading()) {
                        //任务直接运行
                            displayTask1.run();
                        } else {
                        //任务提交到engine中
                            this.engine.submit(displayTask1);
                        }
                    } else {
    //否则直接调用显示模块显示
    options.getDisplayer().display(bmp, imageAware, LoadedFrom.MEMORY_CACHE);
                        listener.onLoadingComplete(uri, imageAware.getWrappedView(), bmp);
                    }
                } else {
           //显示默认值
           if(options.shouldShowImageOnLoading()) {
                        imageAware.setImageDrawable(options.getImageOnLoading(this.configuration.resources));
                    } else if(options.isResetViewBeforeLoading()) {
                        imageAware.setImageDrawable((Drawable)null);
                    }
//新建LoadAndDisplayImageTask任务
                    imageLoadingInfo = new ImageLoadingInfo(uri, imageAware, targetSize, memoryCacheKey, options, listener, progressListener, this.engine.getLockForUri(uri));
                    LoadAndDisplayImageTask displayTask = new LoadAndDisplayImageTask(this.engine, imageLoadingInfo, defineHandler(options));
                     //是否同步加载
                    if(options.isSyncLoading()) {
                     //任务直接运行
                        displayTask.run();
                    } else {
                     //任务提交到engine中
                        this.engine.submit(displayTask);
                    }
                }

            }
        }
    }
    
    

```
*2.ImageAware.java*
需要显示图片的对象的接口，可包装 View 表示某个需要显示图片的 View。
主要函数：
(1) View getWrappedView()
得到被包装的 View，图片在该 View 上显示。
(2) getWidth() 与 getHeight()
得到宽度高度，在计算图片缩放比例时会用到。
(3) getId()
得到唯一标识 id。`ImageLoaderEngine`中用这个 id 标识正在加载图片的`ImageAware`和图片内存缓存 key 的对应关系，图片请求前会将内存缓存 key 与新的内存缓存 key 进行比较，如果不相等，则之前的图片请求会被取消。这样当`ImageAware`被复用时就不会因异步加载(前面任务未取消)而造成错乱了。

*3.ViewAware.java*
封装Android View来显示图片的抽象类，实现了ImageAware接口，利用Reference来 Warp View 防止内存泄露。
主要函数：
(1) ViewAware(View view, boolean checkActualViewSize)
构造函数。
`view`表示需要显示图片的对象。
`checkActualViewSize`表示通过`getWidth()`和`getHeight()`获取图片宽高时返回真实的宽和高，还是`LayoutParams`的宽高，`true` 表示返回真实宽和高。
如果为`true`会导致一个问题，`View`在还没有初始化完成时加载图片，这时它的真实宽高为 `0`，会取它`LayoutParams`的宽高，而图片缓存的 `key` 与这个宽高有关，所以当`View`初始化完成再次需要加载该图片时，`getWidth()`和`getHeight()`返回的宽高都已经变化，缓存`key`不一样，从而导致缓存命中失败会再次从网络下载一次图片。可通过`.denyCacheImageMultipleSizesInMemory()`设置不允许内存缓存缓存一张图片的多个尺寸。

(2) setImageDrawable(Drawable drawable)
如果当前操作在主线程并且`View`没有被回收，则调用抽象函数`setImageDrawableInto(Drawable drawable, View view)`去向`View`设置图片。

(3) setImageBitmap(Bitmap bitmap)
如果当前操作在主线程并且`View`没有被回收，则调用抽象函数`setImageBitmapInto(Bitmap bitmap, View view)`去向`View`设置图片。

*4.ImageViewAware.java*
封装Android ImageView来显示图片的`ImageAware`，继承了`ViewAware`，利用`Reference`来Warp View防止内存泄露。
如果`getWidth()`函数小于等于 0，会利用反射获取`mMaxWidth`的值作为宽。
如果`getHeight()`函数小于等于 0，会利用反射获取`mMaxHeight`的值作为高。

*5.ProcessAndDisplayImageTask.java*
处理并显示图片的Task，实现了Runnable接口。
主要函数：
(1) run()
主要通过`imageLoadingInfo`得到`BitmapProcessor`处理图片，并用处理后的图片和配置新建一个`DisplayBitmapTask`在`ImageAware`中显示图片。

*6.LoadAndDisplayImageTask.java*
加载并显示图片的Task，实现了`Runnable`接口，用于从网络、文件系统或内存获取图片并解析，然后调用`DisplayBitmapTask`在`ImageAware`中显示图片。
主要函数：
(1) run()
获取图片并显示，核心代码如下：
```java
bmp = configuration.memoryCache.get(memoryCacheKey);
if (bmp == null || bmp.isRecycled()) {
    bmp = tryLoadBitmap();
    ...
    ...
    ...
    if (bmp != null && options.isCacheInMemory()) {
        L.d(LOG_CACHE_IMAGE_IN_MEMORY, memoryCacheKey);
        configuration.memoryCache.put(memoryCacheKey, bmp);
    }
}
……
DisplayBitmapTask displayBitmapTask = new DisplayBitmapTask(bmp, imageLoadingInfo, engine, loadedFrom);
runTask(displayBitmapTask, syncLoading, handler, engine);
```
从上面代码段中可以看到先是从内存缓存中去读取 `bitmap` 对象，若 `bitmap`对象不存在，则调用`tryLoadBitmap()`函数获取`bitmap`对象，获取成功后若在`DisplayImageOptions.Builder`中设置了`cacheInMemory(true)`, 同时将`bitmap`对象缓存到内存中。最后新建DisplayBitmapTask显示图片。

*7.DisplayBitmapTask.java*
显示图片的`Task`，实现了`Runnable`接口，必须在主线程调用。

- LRU
`UIL`的内存缓存默认使用了`LRU`算法。LRU: Least Recently Used近期最少使用算法, 选用了基于链表结构的`LinkedHashMap` 作为存储结构。假设情景：内存缓存设置的阈值只够存储两个`bitmap` 对象，当`put`第三个`bitmap`对象时，将近期最少使用的`bitmap`对象移除。
- 工具类
 
```java
/**
 * ImageLoader工具类
 * String imageUri = "http://site.com/image.png"; // from Web
 * String imageUri = "file:///mnt/sdcard/image.png"; // from SD card
 * String imageUri = "content://media/external/audio/albumart/13"; // from content provider
 * String imageUri = "assets://image.png"; // from assets
 * String imageUri = "drawable://" + R.drawable.image; // from drawables (only images, non-9patch)
 * Created by XP on 2015/8/12.
 */
public class ImageLoaderUtils {
    public static void displaySimpleImage(String url, ImageView view) {
        ImageLoader imageLoader = ImageLoader.getInstance();
        DisplayImageOptions options = getSimpleOptions();
        // displayImage将ImageView转换成ImageViewAware,将ImageView的强引用变成弱引用，
        // 当内存不足的时候，可以更好的回收ImageView对象，并获取ImageView的宽度和高度。
        // 这使得我们可以根据ImageView的宽高去对图片进行一个裁剪，减少内存的使用。
        imageLoader.displayImage(url, view, options);

    }

    public static void displayUserImage(String url, ImageView view) {
        ImageLoader imageLoader = ImageLoader.getInstance();
        imageLoader.clearDiskCache();
        imageLoader.clearMemoryCache();
        DisplayImageOptions options = getSimpleOptions();
        imageLoader.displayImage(url, view, options);
    }

    public static void displayWholeImage(String url, ImageView view) {
        ImageLoader imageLoader = ImageLoader.getInstance();
        DisplayImageOptions options = getWholeOptions();
        imageLoader.displayImage(url, view, options);
    }

    /**
     * 显示背景
     *
     * @param uri
     * @param layout
     */
    public static void displayBackground(String uri, final View layout) {
        ImageLoader imageLoader = ImageLoader.getInstance();
        DisplayImageOptions options = getSimpleOptions();//不加options会导致图片错位，不显示
        imageLoader.loadImage(uri, options, new SimpleImageLoadingListener() {
            @Override
            public void onLoadingComplete(String imageUri, View view, Bitmap loadedImage) {
                super.onLoadingComplete(imageUri, view, loadedImage);
                layout.setBackgroundDrawable(new BitmapDrawable(loadedImage));
            }

        });
    }

    /**
     * 设置常用的设置项
     *
     * @return options
     */
    private static DisplayImageOptions getSimpleOptions() {
        DisplayImageOptions options = new DisplayImageOptions.Builder()
//                .showImageOnLoading(R.mipmap.loading) //设置图片在下载期间显示的图片
                .showImageForEmptyUri(R.mipmap.loading)//设置图片Uri为空或是错误的时候显示的图片
                .showImageOnFail(R.mipmap.loading)  //设置图片加载/解码过程中错误时候显示的图片
                .cacheInMemory(true)//设置下载的图片是否缓存在内存中
                .cacheOnDisk(true)//设置下载的图片是否缓存在SD卡中
                .imageScaleType(ImageScaleType.IN_SAMPLE_INT)//设置图片以如何的编码方式显示
                .bitmapConfig(Bitmap.Config.RGB_565)//设置图片的解码类型
                .resetViewBeforeLoading(true)
                .build();
        return options;
    }

    /**
     * 显示图片的所有配置
     *
     * @return options
     */
    private static DisplayImageOptions getWholeOptions() {
        DisplayImageOptions options = new DisplayImageOptions.Builder()
                .showImageOnLoading(R.mipmap.loading) //设置图片在下载期间显示的图片
                .showImageForEmptyUri(R.mipmap.ic_launcher)//设置图片Uri为空或是错误的时候显示的图片
                .showImageOnFail(R.mipmap.loading)  //设置图片加载/解码过程中错误时候显示的图片
                .cacheInMemory(true)//设置下载的图片是否缓存在内存中
                .cacheOnDisk(true)//设置下载的图片是否缓存在SD卡中
                .considerExifParams(true)  //是否考虑JPEG图像EXIF参数（旋转，翻转）
                .imageScaleType(ImageScaleType.IN_SAMPLE_INT)//设置图片以如何的编码方式显示
                .bitmapConfig(Bitmap.Config.RGB_565)//设置图片的解码类型
                        //.decodingOptions(BitmapFactory.Options decodingOptions)//设置图片的解码配置
                .delayBeforeLoading(0)//int delayInMillis为你设置的下载前的延迟时间
                        //设置图片加入缓存前，对bitmap进行设置
                        //.preProcessor(BitmapProcessor preProcessor)
                .resetViewBeforeLoading(true)//设置图片在下载前是否重置，复位
                .displayer(new RoundedBitmapDisplayer(20))//不推荐用！！！！是否设置为圆角，弧度为多少
                .displayer(new FadeInBitmapDisplayer(100))//是否图片加载好后渐入的动画时间，可能会出现闪动
                .build();//构建完成
        return options;
    }
}
```





