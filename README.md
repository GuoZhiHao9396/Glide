# Glide-使用文档


* [开始！](#开始！)
* [加载进阶](#加载进阶)
* [ListAdapter(ListView, GridView)](#ListAdapter(ListView, GridView))
* [占位符 和 渐现动画](#占位符 和 渐现动画)
* [图片重设大小 和 缩放](#图片重设大小 和 缩放)
* [显示 Gif 和 Video](#显示 Gif 和 Video)
* [缓存基础](#缓存基础)
* [请求优先级](#请求优先级)
* [缩略图](#缩略图)
* [回调：SimpleTarget 和 ViewTarget 用于自定义视图类](#回调：SimpleTarget 和 ViewTarget 用于自定义视图类)
* [加载图片到通知栏和应用小部件中](#加载图片到通知栏和应用小部件中)
* [异常：调试和错误处理](#异常：调试和错误处理)
* [自定义转换](#自定义转换)
* [用 animate() 自定义动画](#用 animate() 自定义动画)
* [集成网络栈](#集成网络栈)
* [用 Module 自定义 Glide](#用 Module 自定义 Glide)
* [Module 实例：接受自签名证书的 HTTPS](#Module 实例：接受自签名证书的 HTTPS)
* [Module 实例：自定义缓存](#Module 实例：自定义缓存)
* [Module 实例：用自定义尺寸优化加载的图片](#Module 实例：用自定义尺寸优化加载的图片)
* [动态使用 Model Loader](#动态使用 Model Loader)
* [如何旋转图像](#如何旋转图像)
* [系列综述](#系列综述)

## 开始！

### 为何使用 Glide？

有经验的 Android 开发者可以跳过这节，但对于初学者来说，你可能会问自己为什么你想要去用 Glide，而不是自己去实现。

Android 在处理图片工作的时候显得有点娘，因为它会以像素形式加载图片到内存中去，一张照片平均普通的手机摄像头尺寸是 2592x1936 像素（5百万像素）将大约会分配 19MB 内存。对于复杂的网络情况，缓存和图片处理，如果你用了一个测试完善开发完成的库，如 Glide，你会省下大量的时间，还不会让你头疼！

在这个系列，我们将看到 Glide 的很多特性，去看下这篇博客的提纲，并考虑你是否真的要去开发所有这些功能。

### 添加 Glide

首先，添加 Glide 到你的依赖中，最新的版本是 Glide 是 3.7.0。
```text
Gradle：
       compile 'com.github.bumptech.glide:glide:3.7.0'
Maven：
       <dependency>
           <groupId>com.github.bumptech.glide</groupId>
           <artifactId>glide</artifactId>
           <version>3.7.0</version>
           <type>aar</type>
       </dependency>
```

### 第一次：从一个 URL 中加载图片

就像 Picasso， Glide 库是使用流接口(fluent interface)。对一个完整的功能请求，Glide 建造者要求最少有三个参数。

 * with(Context context) - 对于很多 Android API 调用，Context 是必须的。Glide 在这里也一样
 * load(String imageUrl) - 这里你可以指定哪个图片应该被加载，同上它会是一个字符串的形式表示一个网络图片的 URL
 * into(ImageView targetImageView) 你的图片会显示到对应的 ImageView 中。
 
理论解释总是苍白的，所以，看一下实际的例子吧：
   
```java
ImageView targetImageView = (ImageView) findViewById(R.id.imageView);
String internetUrl = "http://i.imgur.com/DvpvklR.png";

Glide
    .with(context)
    .load(internetUrl)
    .into(targetImageView);
```

## 加载进阶

### 从资源中加载

首先从Android 资源中加载，使用一个资源 id (int)，来替换之前使用字符串去指明一个网络 URL 的情况。

```java
int resourceId = R.mipmap.ic_launcher;

Glide
    .with(context)
    .load(resourceId)
    .into(imageViewResource);
```

### 从文件中加载

其次是从文件中加载，当你让用户选择一张照片去显示图像（比如画廊）这可能会比较有用。该参数只是一个文件对象。我们看一个例子：

```java
//这个文件可能不存在于你的设备中。然而你可以用任何文件路径，去指定一个图片路径。
File file = new File(Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES), "Running.jpg");

Glide
    .with(context)
    .load(file)
    .into(imageViewFile);
```

### 从 Uri 中加载

最后，你也指定一个 Uri 来加载图片。该请求和之前的没有什么不同。

```java
//这可能是任何 Uri。为了演示的目的我们只是用一个 launcher icon 去创建了一个 Uri 
Uri uri = resourceIdToUri(context, R.mipmap.future_studio_launcher);

Glide
    .with(context)
    .load(uri)
    .into(imageViewUri);
```

一个小助手功能：简单的从资源 id 转换成 Uri。

```java
public static final String ANDROID_RESOURCE = "android.resource://";
public static final String FOREWARD_SLASH = "/";

private static Uri resourceIdToUri(Context context, int resourceId) {
    return Uri.parse(ANDROID_RESOURCE + context.getPackageName() + FOREWARD_SLASH + resourceId);
}
```
然而， Uri 不必从资源中去生成，它可以是任何 Uri。

## ListAdapter(ListView, GridView)

### 画廊实现示例：ListView

首先我们需要一些测试图片。我们从我们的 [pixabay](https://pixabay.com/) 项目中去拿了一些图片。

```java
public static String[] eatFoodyImages = {
        "http://i.imgur.com/rFLNqWI.jpg",
        "http://i.imgur.com/C9pBVt7.jpg",
        "http://i.imgur.com/rT5vXE1.jpg",
        "http://i.imgur.com/aIy5R2k.jpg",
        "http://i.imgur.com/MoJs9pT.jpg",
        "http://i.imgur.com/S963yEM.jpg",
        "http://i.imgur.com/rLR2cyc.jpg",
        "http://i.imgur.com/SEPdUIx.jpg",
        "http://i.imgur.com/aC9OjaM.jpg",
        "http://i.imgur.com/76Jfv9b.jpg",
        "http://i.imgur.com/fUX7EIB.jpg",
        "http://i.imgur.com/syELajx.jpg",
        "http://i.imgur.com/COzBnru.jpg",
        "http://i.imgur.com/Z3QjilA.jpg",
};
```








