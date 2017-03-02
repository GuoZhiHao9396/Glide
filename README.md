# Glide-使用文档

* [开始](#开始) 
* [加载进阶](#加载进阶)
* [ListAdapter(ListView, GridView)使用方法](#ListAdapter(ListView, GridView)使用方法)
* [占位符和渐现动画](#占位符和渐现动画)
* [图片重设大小和缩放](#图片重设大小和缩放)
* [显示Gif和Video](#显示动态图和视频)
* [缓存基础](#缓存基础)
* [请求优先级](#请求优先级)
* [缩略图](#缩略图)
* [回调：SimpleTarget 和 ViewTarget 用于自定义视图类](#回调自定义视图类)
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

## 开始

* ### 为何使用 Glide？

有经验的 Android 开发者可以跳过这节，但对于初学者来说，你可能会问自己为什么你想要去用 Glide，而不是自己去实现。

Android 在处理图片工作的时候显得有点娘，因为它会以像素形式加载图片到内存中去，一张照片平均普通的手机摄像头尺寸是 2592x1936 像素（5百万像素）将大约会分配 19MB 内存。对于复杂的网络情况，缓存和图片处理，如果你用了一个测试完善开发完成的库，如 Glide，你会省下大量的时间，还不会让你头疼！

在这里，我们将看到 Glide 的很多特性，去看上面的提纲，并考虑你是否真的要去开发所有这些功能。

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

就像 Picasso， Glide 库是使用流接口([fluent interface](https://en.wikipedia.org/wiki/Fluent_interface))。对一个完整的功能请求，Glide 建造者要求最少有三个参数。

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

首先从Android 资源中加载，使用一个资源 id (int)，来替换之前使用字符串去指明一个网络 URL 的情况
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

## ListAdapter(ListView, GridView)使用方法

### 画廊实现示例：ListView

首先我们需要一些测试图片。我们从我们的 [pixabay](https://pixabay.com/) 网址中去拿了一些图片。
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
其次，我们需要一个 activity，它创建一个 adapter 并设置给一个 ListView。
```java
public class UsageExampleAdapter extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        setContentView(R.layout.activity_usage_example_adapter);

        listView.setAdapter(new ImageListAdapter(UsageExampleAdapter.this, eatFoodyImages));
    }
}
```
再次，看下 adapter 的布局文件。这个 ListView 的 item 的布局文件是非常简单的。
```xml
<?xml version="1.0" encoding="utf-8"?>
<ImageView xmlns:android="http://schemas.android.com/apk/res/android"
       android:layout_width="match_parent"
       android:layout_height="200dp"/>
```
这回显示一个图片列表，每个的高度是 200dp，并且填充设备的宽度。显然，这不是最好的图片画廊，不过，不要在意这些细节。

在这之前，我们需要为 ListView 实现一个 adapter。让它看起来是简单的，并绑定我们的 eatfoody 样本图片到 adapter。每个 item 会显示一个图片。
```java
public class ImageListAdapter extends ArrayAdapter {
    private Context context;
    private LayoutInflater inflater;

    private String[] imageUrls;

    public ImageListAdapter(Context context, String[] imageUrls) {
        super(context, R.layout.listview_item_image, imageUrls);

        this.context = context;
        this.imageUrls = imageUrls;

        inflater = LayoutInflater.from(context);
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        if (null == convertView) {
            convertView = inflater.inflate(R.layout.listview_item_image, parent, false);
        }

        Glide
            .with(context)
            .load(imageUrls[position])
            .into((ImageView) convertView);

        return convertView;
    }
}
```
有趣的事情发生在 ImageListAdapter 类里的 getView() 方法中。你会看到 Glide 调用方式和之前的’常规’加载图片的方式是完全一样的。不管你在应用中想要如何去加载，Glide 的使用方式总是一样的。

作为一个进阶的 Android 开发者你需要知道我们需要去重用 ListView 的布局，去创建一个快速又顺滑滚动的体验。Glide 的魅力是自动处理请求的取消，清楚 ImageView，并加载正确的图片到对应的 ImageView。

![Test](https://futurestud.io/blog/content/images/2015/09/glide-listview--1-.png)

### Glide 的一个优势：缓存

当你上下滚动很多次，你会看到图片显示的之前的快的多。在比较新的手机上，这甚至都不需要时间去等。你可以会猜测，这些图片可能是来自缓存，而不再是从网络中请求。Glide 的缓存实现是基于 Picasso，这对你来说会更加全面的而且做很多事情会更加容易。缓存实现的大小是依赖于设备的磁盘大小。

当加载图片时，Glide 使用3个来源：内存，磁盘和网络（从最快到最慢排序）。再说一次，这里你不需要做任何事情。Glide 帮你隐藏了所有复杂的情况，同时为你创建了一个智能的缓存大小。我们将在以后的博客中去了解这块缓存知识。

### 画廊实现示例：GridView

对于 GridView 来说这和 ListView 的实现并没有什么不同，你实际上可以用相同的 adapter，只需要在 activity 的布局文件改成 GridView:
```xml
<?xml version="1.0" encoding="utf-8"?>
<GridView
    android:id="@+id/usage_example_gridview"
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:numColumns="2"/>
```
这是结果：

![Test](https://futurestud.io/blog/content/images/2015/09/glide-grid--1-.png)

### 其他应用：ImageView 作为元素

目前为止，我们仅仅看了整个 adapter 的 item 是一个 ImageView。该方法仍然应用于一个或者多个 ImageView 作为 adapter item 的一部分的情况。你的 getView() 代码会有一点不同，但是 Glide 项的加载方式是完全相同的。

## 占位符和渐现动画

空 ImageView 在任何 UI 上都是不好看的。如果你用 Glide，你很可能是通过网络连接加载图像。根据你用户的环境，这可能需要花费很多的时间。一个预期的行为是一个APP 去显示一个占位符直到这张图片加载处理完成。

Glide 的流式接口让这个变得非常容易的去做到！只需要调用 .placeHolder() 用一个 drawable(resource) 引用，Glide 将会显示它作为一个占位符，直到你的实际图片准备好。
```java
Glide
    .with(context)
    .load(UsageExampleListViewAdapter.eatFoodyImages[0])
    .placeholder(R.mipmap.ic_launcher) // can also be a drawable
    .into(imageViewPlaceholder);
```
做为一个显而易见的原因，你不能设置一个网络 url 作为占位符，因为这也会被去请求加载的。App 资源和 drawable 能保证可用和可访问的。然而，作为 load() 方法的参数，Glide 接受所有值。这可能不是可加载的（没有网络连接，服务器宕机…），删除了或者不能访问。在下一节中，我们将讨论一个错误的占位符。

### 错误占位符：.error()

假设我们的 App 尝试从一个网站去加载一张图片。Glide 给我们一个选项去获取一个错误的回调并采取合适的行动。我们会在后面来讨论，对现在来说，可能太复杂了。在大多数情况下使用占位符，来指明图片不能被加载已经足够了。

调用 Glide 的流式接口和之前显示预加载占位符的例子是相同的，不同的是调用了名为 error() 的函数。
```java
Glide
    .with(context)
    .load("http://futurestud.io/non_existing_image.png")
    .placeholder(R.mipmap.ic_launcher) // can also be a drawable
    .error(R.mipmap.future_studio_launcher) // will be displayed if the image cannot be loaded
    .into(imageViewError);
```
就这样。如果你定义的 load() 值的图片不能被加载出来，Glide 会显示 R.mipmap.future_studio_launcher 作为替换。再说一次，error()接受的参数只能是已经初始化的 drawable 对象或者指明它的资源(R.drawable.drawable-keyword)。

### 使用crossFade()

无论你是在加载图片之前是否显示一个占位符，改变 ImageView 的图片在你的 UI 中有非常显著的变化。一个简单的选项是让它改变是更加平滑和养眼的，就是使用一个淡入淡出动画。Glide 使用标准的淡入淡出动画，这是(对于当前版本3.7.0)默认激活的。如果你想要如强制 Glide 显示一个淡入淡出动画，你必须调用另外一个建造者：
```java
Glide
    .with(context)
    .load(UsageExampleListViewAdapter.eatFoodyImages[0])
    .placeholder(R.mipmap.ic_launcher) // can also be a drawable
    .error(R.mipmap.future_studio_launcher) // will be displayed if the image cannot be loaded
    .crossFade()
    .into(imageViewFade);
```
crossFade() 方法还有另外重载方法 .crossFade(int duration)。如果你想要去减慢（或加快）动画，随时可以传一个毫秒的时间给这个方法。动画默认的持续时间是 300毫秒。

### 使用 dontAnimate()

如果你想直接显示图片而没有任何淡入淡出效果，在 Glide 的建造者中调用 .dontAnimate() 。
```java
Glide
    .with(context)
    .load(UsageExampleListViewAdapter.eatFoodyImages[0])
    .placeholder(R.mipmap.ic_launcher) // can also be a drawable
    .error(R.mipmap.future_studio_launcher) // will be displayed if the image cannot be loaded
    .dontAnimate()
    .into(imageViewFade);
```
这是直接显示你的图片，而不是淡入显示到 ImageView。请确保你有更好的理由来做这件事情。

需要知道的是所有这些参数都是独立的，而不需要彼此依赖的。比如，你可以设定 .error() 而不调用 .placeholder()。你可能设置 crossFade() 动画而没有占位符。任何参数的组合都是可能的。

## 图片重设大小和缩放

### 用 resize(x,y) 调整图片大小### 

通常情况下，如果你的服务器或者 API 提供的图像是你需要的精确尺寸，这时是完美的情况下，在内存小号和图像质量之间的权衡。

在和 Picasso 比较后，Glide 有更加高效的内存管理。Glide 自动限制了图片的尺寸在缓存和内存中，并给到 ImageView 需要的尺寸。Picasso 也有这样的能力，但需要调用 fit() 方法。对于 Glide，如果图片不会自动适配到 ImageView，调用 override(horizontalSize, verticalSize) 。这将在图片显示到 ImageView之前重新改变图片大小。
```java
Glide
    .with(context)
    .load(UsageExampleListViewAdapter.eatFoodyImages[0])
    .override(600, 200) // resizes the image to these dimensions (in pixel). does not respect aspect ratio
    .into(imageViewResize);
```
当你还没有目标 view 去知道尺寸的时候，这个选项也可能是有用的。比如，如果 App 想要在闪屏界面预热缓存，它还不能测量 ImageView 的尺寸。然而，如果你知道这个图片多少大，用 override 去提供明确的尺寸。

### 缩放图像

现在，对于任何图像操作，调整大小真的能让长宽比失真并且丑化图像显示。在你大多数的使用场景中，你想要避免发生这种情况。Glide 提供了一般变化去处理图像显示。提供了两个标准选项：centerCrop 和 fitCenter。

### CenterCrop

CenterCrop()是一个裁剪技术，即缩放图像让它填充到 ImageView 界限内并且裁剪额外的部分。ImageView 可能会完全填充，但图像可能不会完整显示。
```java
Glide
    .with(context)
    .load(UsageExampleListViewAdapter.eatFoodyImages[0])
    .override(600, 200) // resizes the image to these dimensions (in pixel)
    .centerCrop() // this cropping technique scales the image so that it fills the requested bounds and then crops the extra.
    .into(imageViewResizeCenterCrop);
```

### FitCenter

fitCenter() 是裁剪技术，即缩放图像让图像都测量出来等于或小于 ImageView 的边界范围。该图像将会完全显示，但可能不会填满整个 ImageView。
```java
Glide
    .with(context)
    .load(UsageExampleListViewAdapter.eatFoodyImages[0])
    .override(600, 200)
    .fitCenter() 
    .into(imageViewResizeFitCenter);
```
## 显示动态图和视频

### 显示 Gif

有很多图片加载库来去加载和显示图片。能支持 Gif 有一些特别也是非常有帮助的，如果在你的 App 需要的话。Glide 实现 Gif 是如此的特别和令人惊讶，因为它是如此的简单。如果你想显示一个 Gif，你可以只使用和过去相同的调用方式就可以了：
```java
String gifUrl = "http://i.kinja-img.com/gawker-media/image/upload/s--B7tUiM5l--/gf2r69yorbdesguga10i.gif";
Glide  
    .with( context )
    .load( gifUrl )
    .into( imageViewGif );
```

就这样！这将在 ImageView 中显示 Gif 并自动开始播放它。另外一个关于 Glide 的伟大的事情是你仍然可以使用你的标准去调用处理这个 Gif:

```java
Glide  
    .with( context )
    .load( gifUrl )
    .placeholder( R.drawable.cupcake )
    .error( R.drawable.full_cake )
    .into( imageViewGif );
```
### Gif 检查

上面的代码有一个潜在的问题是，如果提供的来源不是一个 Gif，可能只是一个常规图片，这就没有办法显示这个问题。Glide 接受 Gif 或者图片作为 load() 参数。如果你期望这个 URL 是一个 Gif，Glide 不会自动检查是否是 Gif。因此他们引入了一个额外的防区强制 Glide变成一个 Gif asGif():

```java
Glide  
    .with( context )
    .load( gifUrl )
    .asGif()
    .error( R.drawable.full_cake )
    .into( imageViewGif );
```
如果 gifUrl 是一个 git，这没什么变化。然而，不像之前那样，如果这个 gifUrl 不是一个 Gif，Glide 将会把这个 load 当成失败处理。这样做的的好处是，.error() 回调被调用并且错误占位符被显示，即使 gifUrl 是一个完美的图片（但不是一个 Gif）。

### Gif 转为 Bitmap

如果你的 App 显示一个位置的网络 URL 列表，它可能遇到常规的图片或者 Gif。在某些情况下，你可能对不想系那是整个 Gif。如果你仅仅想要显示 Gif 的第一帧，你可以调用 asBitmap() 去保证其作为一个常规的图片显示，即使这个 URL 是一个 Gif。

```java
Glide  
    .with( context )
    .load( gifUrl )
    .asBitmap()
    .into( imageViewGifAsBitmap );
```

这让你用 Glide 显示所有已知的 url 显示为图片的形式
### 显示本地视频

Glide 还能显示视频！只要他们是存储在手机上的。让我们假设你通过让用户选择一个视频后得到了一个文件路径：

```java
String filePath = "/storage/emulated/0/Pictures/example_video.mp4";
Glide  
    .with( context )
    .load( Uri.fromFile( new File( filePath ) ) )
    .into( imageViewGifAsBitmap );
```
这里需要注意的是，这仅仅对于本地视频起作用。如果没有存储在该设备上的视频（如一个网络 URL 的视频），它是不工作的！

## 缓存基础

在 Android App 中必须去做的是一个很好的实现图片加载组件，尝试去减少网络请求。Glide 在这里并没有什么不同。Glide 通过使用默认的内存和磁环缓存去避免不必要的网络请求。我们将在后面的博客中去详细的查看实现细节。如果你等不到那个时候，通过浏览[官方文档](https://github.com/bumptech/glide/wiki/Caching-and-Cache-Invalidation)这个话题。

目前最重要的是带着所有的图片请求放到内存和磁盘中。虽然缓存通常是很有用的，但在某些情况下，它可能不是像期待的行为那样。在下一节中，我们会看看如何为单个请求改变 Glide 的缓存行为。

### 使用缓存策略

如果你以前用过 Glide。你会发现不需要去做任何额外的事情来激活缓存。它直接就从盒子里取出来用了！然而，如果你知道一张图片变化很快，你可能想要避免某些缓存。

Glide 提供了方法去适配内存和磁盘缓存行为。让我们先看看内存缓存。

### 内存缓存

让我们想象一个非常简单的请求，从网络中加载图片到 ImageView。

```java
Glide  
    .with( context )
    .load( eatFoodyImages[0] )
    .skipMemoryCache( true )
    .into( imageViewInternet );
```

我们调用了 .skipMemoryCache(true) 去明确告诉 Glide 跳过内存缓存。这意味着 Glide 将不会把这张图片放到内存缓存中去。这里需要明白的是，这只是会影响内存缓存！Glide 将会仍然利用磁盘缓存来避免重复的网络请求。

这也容易知道 Glide 将会默认将所有的图片资源放到内存缓存中去。因为，指明调用 .skipMemoryCache(false) 是没有必要的。

#### 提示：注意一个事实，对于相同的 URL ，如果你的初始请求没调用 .skipMemoryCache(true) 方法，你后来又调用了 .skipMemoryCache(true) 这个方法，这个资源将会在内存中获取缓存。当你想要去调整缓存行为时，确保对同一个资源调用的一致性。

### 跳过磁盘缓存

正如你上面这部分所了解到的，即使你关闭内存缓存，请求图片将会仍然被存储在设备的磁盘缓存中。如果你有一张图片具有相同的 URL，但是变化很快，你可能想要连磁盘缓存也一起禁用。

你可以用 .diskCacheStrategy() 方法为 Glide 改变磁盘缓存的行为。不同的于 .skipMemoryCache() 方法，它需要一个枚举而不是一个简答的布尔值。如果你想要为一个请求禁用磁盘缓存。使用枚举 DiskCacheStrategy.NONE 作为参数。

```java
Glide  
    .with( context )
    .load( eatFoodyImages[0] )
    .diskCacheStrategy( DiskCacheStrategy.NONE )
    .into( imageViewInternet );
```

图片在这段代码片段中将不会被保存在磁盘缓存中。然而，默认的它将仍然使用内存缓存！为了把这里两者都禁用掉，两个方法一起调用：

```java
Glide  
    .with( context )
    .load( eatFoodyImages[0] )
    .diskCacheStrategy( DiskCacheStrategy.NONE )
    .skipMemoryCache( true )
    .into( imageViewInternet );
```
### 自定义磁盘缓存行为

正如我们之前提到的，Glide 有多个选项去配置磁盘缓存行为。在我们向你展示这些选项之前，你必须了解到 Glide 的磁盘缓存是相当复杂的。比如，Picasso 仅仅缓存了全尺寸的图像。然而 Glide 缓存了原始图像，全分辨率图像和另外小版本的图像。比如，如果你请求的一个图像是 1000x1000 像素的，但你的 ImageView 是 500x500 像素的，Glide 将会把这两个尺寸都进行缓存。

现在你将会理解对于 .diskCacheStrategy() 方法来说不同的枚举参数的意义：

      DiskCacheStrategy.NONE 什么都不缓存，就像刚讨论的那样
      DiskCacheStrategy.SOURCE 仅仅只缓存原来的全分辨率的图像。在我们上面的例子中，将会只有一个 1000x1000 像素的图片
      DiskCacheStrategy.RESULT 仅仅缓存最终的图像，即，降低分辨率后的（或者是转换后的）
      DiskCacheStrategy.ALL 缓存所有版本的图像（默认行为）
      
作为最后一个例子，如果你有一张图片，你知道你将会经常操作处理，并做了一堆不同的版本，对其有意义的仅仅是缓存原始分辨率图片。因此，我们用 DiskCacheStrategy.SOURCE 去告诉 Glide 仅仅保存原始图片：

```java
Glide  
    .with( context )
    .load( eatFoodyImages[2] )
    .diskCacheStrategy( DiskCacheStrategy.SOURCE )
    .into( imageViewFile );
```
## 请求优先级

通常，你会遇到这样的使用场景：你的 App 将会需要在同一时间内加载多个图像。让我们假设你正在构建一个信息屏幕，这里有一张很大的英雄图片在顶部，还有两个小的，在底部还有一些不那么重要的图片。对于最好的用户体验来说，应用图片元素是显示要被加载和显示的，然后才是底部不紧急的 ImageView。Glide 可以用 Priority 枚举来支持你这样的行为，调用 .priority() 方法。

但在看这个方法调用的示例代码之前，让么我看看 priority 的枚举值，它首先作为 .priority() 方法的参数的。

### 了解 Priority (优先级)枚举

这个枚举给了四个不同的选项，下面是按照递增priority(优先级)的列表：

  * Priority.LOW
  * Priority.NORMAL
  * Priority.HIGH
  * Priority.IMMEDIATE

在我们开始例子前，你应该知道的是：优先级并不是完全严格遵守的。Glide 将会用他们作为一个准则，并尽可能的处理这些请求，但是它不能保证所有的图片都会按照所要求的顺序加载。

然而，如果你有的使用场景是确定一些图片是重要的，充分利用它！

### 使用实例：英雄元素和子图像

让我们开始回到开始时的例子吧。你正在实现一个信息详情页面，有一个英雄图片在顶部，和较小的图片在底部。对于最好的用户体验来说，英雄图片首先需要被加载。因此，我们用 Priority.HIGH 来处理它。理论上说，这应该够了，但是为了让这个实例增加点趣味，我们也将底层图像分配给低优先级，用 .priority(Priority.LOW) 调用：

```java
private void loadImageWithHighPriority() {  
    Glide
        .with( context )
        .load( UsageExampleListViewAdapter.eatFoodyImages[0] )
        .priority( Priority.HIGH )
        .into( imageViewHero );
}

private void loadImagesWithLowPriority() {  
    Glide
        .with( context )
        .load( UsageExampleListViewAdapter.eatFoodyImages[1] )
        .priority( Priority.LOW )
        .into( imageViewLowPrioLeft );

    Glide
        .with( context )
        .load( UsageExampleListViewAdapter.eatFoodyImages[2] )
        .priority( Priority.LOW )
        .into( imageViewLowPrioRight );
}
```

## 缩略图

### 缩略图优势
在你要用缩略图去做优化之前，确保你理解和掌握了所有缓存的选项和请求优先级。如果你已经实现了这些，再来查看缩略图是否能帮助更好的提高你的 Android 应用。

缩略图不同于之前博客提到的占位符。占位符必须附带应用程序捆绑的资源才行。缩略图是动态占位符。它也可以从网络中加载。缩略图将会在实际请求加载完或者处理完之后才显示。如果缩略图对于任何原因，在原始图像到达之后，它不会取代原始图像。它只会被抹除。

#### 提示：另外一个流畅加载图片过程的真的很棒的方式是用色彩图像占位符的图像背景的主色彩作为图像。我们也为此写了一个指南。

### 简单的缩略图

Glide 为缩略图提供2个不同的方式。第一个是简单的选择，在原始图像被用过之后，这只需要一个较小的分辨率。这个方法在 ListView的组合和详细视图中是非常有用的。如果你已经在 ListView 中显示了图像。这么说吧，在250x250 像素的中，图像将在详细视图中需要一个更大的分辨率图像。然而，从用户的角度来看，他已经看到较小版本的图像，为什么在详情页中出现一个占位符显示了几秒，然后相同图像又再次一次显示（高分辨率的）？

在这种情况下，它有更好的意义去继续显示这张 250x250 像素版本的图像在详情视图上，并且后台去加载全分辨率的图像。Glide 的 .thumbnail() 方法让这一切成为可能。 在这样的情况下，这个参数是一个 float 作为其大小的倍数。

```java
Glide  
    .with( context )
    .load( UsageExampleGifAndVideos.gifUrl )
    .thumbnail( 0.1f )
    .into( imageView2 );
```
例如， 你传了一个 0.1f 作为参数，Glide 将会显示原始图像的10%的大小。如果原始图像有 1000x1000 像素，那么缩略图将会有 100x100 像素。因为这个图像将会明显比 ImageView 小很多，你需要确保它的 ScaleType 的设置是正确的。

请注意，将应用于演示请求的所有请求设置也应用于缩略图。比如，如果你使用了一个变换去做了一个图像灰度。这同样将发生在缩略图中。

### 用完全不同的请求去进阶缩略图

然而用 float 参数来使用 .thumbnail() 是易于设置且非常有效，但它不总是有意义的。如果缩略图是要通过网络去加载相同的全分辨率的图像，则可能不会很快。所以，Glide 提供了另一个选项去加载和显示缩略图。

第二个选择是传一个完全新的 Glide 请求作为参数。让我们来看看实例：

```java
private void loadImageThumbnailRequest() {  
    // setup Glide request without the into() method
    DrawableRequestBuilder<String> thumbnailRequest = Glide
        .with( context )
        .load( eatFoodyImages[2] );

    // pass the request as a a parameter to the thumbnail request
    Glide
        .with( context )
        .load( UsageExampleGifAndVideos.gifUrl )
        .thumbnail( thumbnailRequest )
        .into( imageView3 );
}
```
所不同的是，第一个缩略图请求是完全独立于第二个原始请求的。该缩略图可以是不同的资源或图片 URL，你可以为其应用不同的转换，等等。

## 回调自定义视图类

### Glide 中的回调：Targets

目前为止，我们很方便的使用 Glide 建造者去加载图片到 ImageView 中了。Glide 隐藏了一大堆复杂的在后台的场景。Glide 做了所有的网络请求和处理在后台线程中，一旦结果准备好了之后，切回到 UI 线程然后更新 ImageView。

在这篇博客中，我们假定 ImageView 不再是图像的最后一步。我们只要 Bitmap 本身。Glide 提供了一个用 Targets 的简单的方式去接受图片资源的 Bitmap。Targets 是没有任何别的回调，它在 Glide 做完所有的加载和处理之后返回结果。

Glide 提供了各种的 targets 并且每个都有其明确的目的。我们将在接下来的几节中通过使用它们。让我们从 SimpleTarget 开始。

### SimpleTarget

看如下代码实例：

```java
private SimpleTarget target = new SimpleTarget<Bitmap>() {  
    @Override
    public void onResourceReady(Bitmap bitmap, GlideAnimation glideAnimation) {
        // do something with the bitmap
        // for demonstration purposes, let's just set it to an ImageView
        imageView1.setImageBitmap( bitmap );
    }
};

private void loadImageSimpleTarget() {  
    Glide
        .with( context ) // could be an issue!
        .load( eatFoodyImages[0] )
        .asBitmap()
        .into( target );
}
```

这段代码的第一部分创建了一个字段对象，声明了一个方法，即一旦 Glide 已加载并处理完图像，它将被调用。这个回调方法传了 Bitmap 作为一个参数。你之后便可以使用这个 Bitmap 对象，无论你要怎样用它。

这段代码的第二部分是我们如何通过 Glide 用 targets：和 ImageView 用法完全相同的！你既可以传一个 Target 也可以传一个 ImageView 参数给 .into() 方法。Glide 自己将会处理并返回结果给任何一个。这里有一些不同的是，我们添加了 .asBitmap()，它强制 Glide 去返回一个 Bitmap 对象。记住，Glide 也可以加载 Gif 或 video 的。为了防止 target 的冲突（我们需要 Bitmap） 和未知资源在网络背后的 URL(可能是一个 Gif)，我们可以调用 .asBitmap() 告诉 Glide 我们需要一个图像。

### 关注 Targets

除了知道如何实现一个简单版本的 Glide 的 Target 回调系统，你要学会额外两件事。

首先是 SimpleTarget 对象的字段声明。从技术上来说，Java/Android 会允许你在 .into() 方法中去声明 target 的匿名内部类。然而，这大大增加了这样一个可能性：即在 Glide 做完图片请求之前， Android 垃圾回收移除了这个匿名内部类对象。最终这可能会导致一个情况，当图像加载完成了，但是回调再也不会被调用。所请确保你所声明的回调对象是作为一个字段对象的，这样你就可以保护它避免被邪恶的 Android 垃圾回收机制回收 ;-)

第二个关键部分是 Glide 建造者中这行：.with(context)。 这里的问题实际是 Glide 的功能：当你传了一个 context，例如是当前应用的 activity，Glide 将会自动停止请求当请求的 activity 已经停止的时候。这整合到了应用的生命周期中通常是非常有帮助的，但是有时工作起来是困难的，如果你的 target 是独立于应用的 activity 生命周期。这里的解决方案是用 application 的 context: .with(context.getApplicationContext))。当应用资深完全停止时，Glide 才会杀死这个图片请求。请求记住，再说一次，如果你的请求需要在 activity 生命周期之外去做时，才用下面这样的代码：

```java
private void loadImageSimpleTargetApplicationContext() {  
    Glide
        .with( context.getApplicationContext() ) // safer!
        .load( eatFoodyImages[1] 
        .asBitmap()
        .into( target2 );
}
```

### Target 指定尺寸

另一个潜在的问题是，target 没有指明大小。如果你你传一个 ImageView 作为参数给 .into()，Glide 将会用 ImageView 的大小去限制图像的大小。比如说，如果加载的图片是 1000x1000 像素的，但是 ImageView 只有 250x250 像素，Glide 将会减少图片的尺寸去节省时间和内存。很显然，在和 target 协作的时候并没有这么做，因为我们并没有已知的大小。然而，如果你有一个指定的大小，你可以提高回调。如果你知道这种图片应该要多大，你应该在你的回调声明中指定它以节省一些内存。

```java
private SimpleTarget target2 = new SimpleTarget<Bitmap>( 250, 250 ) {  
    @Override
    public void onResourceReady(Bitmap bitmap, GlideAnimation glideAnimation) {
        imageView2.setImageBitmap( bitmap );
    }
};

private void loadImageSimpleTargetApplicationContext() {  
    Glide
        .with( context.getApplicationContext() ) // safer!
        .load( eatFoodyImages[1] )
        .asBitmap()
        .into( target2 );
}
```

### ViewTarget

我们不能直接使用 ImageView 的原因可能是多种多样的。我们已经向你展示如果去接收一个 Bitmap。现在我们要更进一步。假设你有一个 Custom View。Glide 并不支持加载图片到自定义 view 中，因为并没有方法知道图片应该在哪里被设置。然而，Glide 可以用 ViewTarget 更容易实现。

让我们看一个简单的自定义 View，它继承自 FrameLayout 并内部使用了一个 ImageView 以及覆盖了一个 TextView。

```java
public class FutureStudioView extends FrameLayout {  
    ImageView iv;
    TextView tv;

    public void initialize(Context context) {
        inflate( context, R.layout.custom_view_futurestudio, this );

        iv = (ImageView) findViewById( R.id.custom_view_image );
        tv = (TextView) findViewById( R.id.custom_view_text );
    }

    public FutureStudioView(Context context, AttributeSet attrs) {
        super( context, attrs );
        initialize( context );
    }

    public FutureStudioView(Context context, AttributeSet attrs, int defStyleAttr) {
        super( context, attrs, defStyleAttr );
        initialize( context );
    }

    public void setImage(Drawable drawable) {
        iv = (ImageView) findViewById( R.id.custom_view_image );

        iv.setImageDrawable( drawable );
    }
}
```

你不能使用常规的 Glide 的方法 .into()，因为我们的自定义 view 并不继承自 ImageView。因此，我们必须创建一个 ViewTarget，并用 .into() 方法：

```java
private void loadImageViewTarget() {  
    FutureStudioView customView = (FutureStudioView) findViewById( R.id.custom_view );

    viewTarget = new ViewTarget<FutureStudioView, GlideDrawable>( customView ) {
        @Override
        public void onResourceReady(GlideDrawable resource, GlideAnimation<? super GlideDrawable> glideAnimation) {
            this.view.setImage( resource.getCurrent() );
        }
    };

    Glide
        .with( context.getApplicationContext() ) // safer!
        .load( eatFoodyImages[2] )
        .into( viewTarget );
}
```

在 target 回调方法中，我们使用我们创建的方法 setImage(Drawable drawable) 在自定义 view 类中去设置图片。另外确保你注意到我们必须在 ViewTarget 的构造函数中传递我们自定义 view 作为参数：new ViewTarget< FutureStudioView , GlideDrawable >(customView)。

这应该涵盖了所有你需要的自定义 view。你也可以在回调中添加额外的工作。如，我们可以分析传入的 Bitmap 的主要的颜色并设置十六进制值给TextView。

## 加载图片到通知栏和应用小部件中

![Test](https://futurestud.io/blog/content/images/2015/10/notification-icon-cropped.png)

通知栏图标对用户来说是重要的上下文。用 NotificationCompat.Builder 来直接设置大的通知图片，但是图像必须以 Bitmap 的形式。如果图片在手机上已经是可用的，这并没什么问题。然而，如果图片斌不在设备上并且需要从网上加载的话，使用标准的方式来处理就变得不可能了。

让 Glide 来做吧。上篇博客中，我们看了如何用 SimpleTarget 将图片以 Bitmap 的形式下载下来。理论上说，你可以利用这种方式去加载图片到你的通知栏中。但这并不是必须的，因为 Glide 提供了一个更加方便舒适的方式：NotificationTarget。

### NotificationTarget

所以，让我们来看代码。现在你知道 Glide target 是如何工作的了，因此我们不会再去用它了。为了显示一张大图片在通知栏，你可以使用 RemoteViews 并显示一个自定义的通知栏。

![Test](https://futurestud.io/blog/content/images/2015/10/custom-notification.png)

我们自定义的通知栏比较简单：

```xml
<?xml version="1.0" encoding="utf-8"?>  
<LinearLayout  
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:background="@android:color/white"
    android:orientation="vertical">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:padding="2dp">
        <ImageView
            android:id="@+id/remoteview_notification_icon"
            android:layout_width="50dp"
            android:layout_height="50dp"
            android:layout_marginRight="2dp"
            android:layout_weight="0"
            android:scaleType="centerCrop"/>
        <LinearLayout
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:orientation="vertical">
            <TextView
                android:id="@+id/remoteview_notification_headline"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:ellipsize="end"
                android:singleLine="true"
                android:textSize="12sp"/>
            <TextView
                android:id="@+id/remoteview_notification_short_message"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:ellipsize="end"
                android:paddingBottom="2dp"
                android:singleLine="true"
                android:textSize="14sp"
                android:textStyle="bold"/>
        </LinearLayout>
    </LinearLayout>
</LinearLayout>  
```

下面的代码用了上面的布局文件为我们创建了一个自定义通知。

```java
final RemoteViews rv = new RemoteViews(context.getPackageName(), R.layout.remoteview_notification);

rv.setImageViewResource(R.id.remoteview_notification_icon, R.mipmap.future_studio_launcher);

rv.setTextViewText(R.id.remoteview_notification_headline, "Headline");  
rv.setTextViewText(R.id.remoteview_notification_short_message, "Short Message");

// build notification
NotificationCompat.Builder mBuilder =  
    new NotificationCompat.Builder(context)
        .setSmallIcon(R.mipmap.future_studio_launcher)
        .setContentTitle("Content Title")
        .setContentText("Content Text")
        .setContent(rv)
        .setPriority( NotificationCompat.PRIORITY_MIN);

final Notification notification = mBuilder.build();

// set big content view for newer androids
if (android.os.Build.VERSION.SDK_INT >= 16) {  
    notification.bigContentView = rv;
}

NotificationManager mNotificationManager = (NotificationManager) context.getSystemService(Context.NOTIFICATION_SERVICE);  
mNotificationManager.notify(NOTIFICATION_ID, notification);  
```

这个代码片段为我们创建了三个重要的对象， notification 和 RemoteViews 以及常量 NOTIFICATION_ID。我们会需要这些去创建一个通知 target。

```java
private NotificationTarget notificationTarget;

...

notificationTarget = new NotificationTarget(  
    context,
    rv,
    R.id.remoteview_notification_icon,
    notification,
    NOTIFICATION_ID);
```

最后，我们要调用 Glide，正如我们之前博客所做的，将 target 作为 .into() 的参数。

```java
Glide  
    .with( context.getApplicationContext() ) // safer!
    .load( eatFoodyImages[3] )
    .asBitmap()
    .into( notificationTarget );
```

### App Widgets

让我们来看另一个 Glide target。 应用小部件一直以来都是 Android 的一部分。如果你的 App 提供了小部件并且包含图像，这部分应该会让你感兴趣的。 Glide 的AppWidgetTarget 能显著的让你非常简单的实现。

来看看一个简单的 AppWidgetProvider 实例：

```java
public class FSAppWidgetProvider extends AppWidgetProvider {

    private AppWidgetTarget appWidgetTarget;

    @Override
    public void onUpdate(Context context, AppWidgetManager appWidgetManager,
                         int[] appWidgetIds) {

        RemoteViews rv = new RemoteViews(context.getPackageName(), R.layout.custom_view_futurestudio);

        appWidgetTarget = new AppWidgetTarget( context, rv, R.id.custom_view_image, appWidgetIds );

        Glide
                .with( context.getApplicationContext() ) // safer!
                .load( GlideExampleActivity.eatFoodyImages[3] )
                .asBitmap()
                .into( appWidgetTarget );

        pushWidgetUpdate(context, rv);
    }

    public static void pushWidgetUpdate(Context context, RemoteViews rv) {
        ComponentName myWidget = new ComponentName(context, FSAppWidgetProvider.class);
        AppWidgetManager manager = AppWidgetManager.getInstance(context);
        manager.updateAppWidget(myWidget, rv);
    }
}
```

几行重要的代码声明了 appWidgetTarget 对象以及 Glide 的建造者。这里的好处是，你不需要去定制 AppWidgetTarget 并重写任何 AppWidgetTarget 方法。Glide 都自动帮你做好了。太棒了！

## 异常：调试和错误处理

Glide 的 GeneralRequest 类提供了一个方法去设置 log 的级别。不幸的是，在生产过程中，使用这个类并不容易。然而，有一个非常简单的方法去获得 Glide 的调试日志。你所要做的就是通过 adb 的 shell 来激活。打开你的终端，使用以下命令：

```text
adb shell setprop log.tag.GenericRequest DEBUG  
```

最后一个 DEBUG 来自标准的 Android 日志常量。因此，你你可以选择 debug 的优先级：

   * VERBOSE
   * DEBUG
   * INFO
   * WARN
   * ERROR
   
输出，万一图像不存在，在会像这样：

```text
io.futurestud.tutorials.glide D/GenericRequest: load failed  
io.futurestud.tutorials.glide D/GenericRequest: java.io.IOException: Request failed 404: Not Found  
...
```

可能你已猜到，这只当你的设备能接收实际的值并且你正在开发和测试你的 App。为了记录在生产中的 App，你将需要用一个不同的方式。答案是，依然用回调，我们在下一节来探讨。

### 常规异常日志记录

Glide 不能直接去访问 GenericRequest 类去设置日志，但万一一些请求出错了你是可以捕获异常的。比如，如果图片不可用，Glide 会（默默地）抛出一个异常，并且显示一个 drawable ，如果你已经指定了 .error() 的话。如果你明确想要知道这个异常，创建一个监听并传 .listener() 方法到 Glide 的建造者中。

首先，创建一个监听作为一个字段对象去避免垃圾回收（注：之前说过不要用匿名内部类的形式）：

```java
private RequestListener<String, GlideDrawable> requestListener = new RequestListener<String, GlideDrawable>() {  
    @Override
    public boolean onException(Exception e, String model, Target<GlideDrawable> target, boolean isFirstResource) {
        // todo log exception

        // important to return false so the error placeholder can be placed
        return false;
    }

    @Override
    public boolean onResourceReady(GlideDrawable resource, String model, Target<GlideDrawable> target, boolean isFromMemoryCache, boolean isFirstResource) {
        return false;
    }
};
```

在 onException 方法中， 你可以捕获错误，并且你可以决定要做什么，比如，打个 log。重要的是如果 Glide 要在后续处理的话，如显示一个错误的占位符等情况的话，你需要返回了 false 在 onException 方法中。

你可以设置一个监听在 Glide 建造者中：

```java
Glide  
    .with( context )
    .load(UsageExampleListViewAdapter.eatFoodyImages[0])
    .listener( requestListener )
    .error( R.drawable.cupcake )
    .into( imageViewPlaceholder );
```

要使日志工作正常的话，.error() 并不是必须的。然而，如果你在监听的 onException 中返回 false 的话，R.drawable.cupcake 只是显示出来而已。

## 自定义转换

### Transformations

在图片被显示之前，transformations(转换) 可以被用于图像的操作处理。比如，如果你的应用需要显示一个灰色的图像，但是我们只能访问到原始色彩的版本，你可以用 transformation 去操作 bitmap，从而将一个明亮色彩版本的图片转换成灰暗的版本。不要理解错啦，transformation 不仅限于颜色转换。你可以图片的任意属性：尺寸，范围，颜色，像素位置等等！Glide 已经包含了2个 transformation，我们之前已经看了图像重设大小，即：fitCenter 和 centerCrop。这两个选项都非常有意义，他们在 Glide 中拥有自己的实现。当然，我们这篇博客不再介绍他们。

### 实现你自己的 Transformation

为了实践自定义转换，你将需要创建一个新类，它实现了 Transformation 接口。要实现这个方法还是比较复杂的，你必须要有对 Glide 内部架构方面的洞察力才能做的比较棒。如果你只是想要对图片（不是 Gif 和 video）做常规的 bitmap 转换，我们推荐你使用抽象类 BitmapTransformation。它简化了很多的实现，这应该能覆盖 95% 的应用场景啦。

所以，来看看 BitmapTransformation 实现实例。如果你定期阅读这个博客，你会知道我们喜欢的转换是 用 Renderscript 模糊图像。我们可以将之前的所有代码重用到 Glide 的转换中。因为我们继承 BitmapTransformation 类，我们用这样的框架：

```java
public class BlurTransformation extends BitmapTransformation {

    public BlurTransformation(Context context) {
        super( context );
    }

    @Override
    protected Bitmap transform(BitmapPool pool, Bitmap toTransform, int outWidth, int outHeight) {
        return null; // todo
    }

    @Override
    public String getId() {
        return null; // todo
    }
}
```

现在我们将之前博客中用 Renderscript 来模糊图像的代码放到我们这里来：

```java
public class BlurTransformation extends BitmapTransformation {

    private RenderScript rs;

    public BlurTransformation(Context context) {
        super( context );

        rs = RenderScript.create( context );
    }

    @Override
    protected Bitmap transform(BitmapPool pool, Bitmap toTransform, int outWidth, int outHeight) {
        Bitmap blurredBitmap = toTransform.copy( Bitmap.Config.ARGB_8888, true );

        // Allocate memory for Renderscript to work with
        Allocation input = Allocation.createFromBitmap(
            rs, 
            blurredBitmap, 
            Allocation.MipmapControl.MIPMAP_FULL, 
            Allocation.USAGE_SHARED
        );
        Allocation output = Allocation.createTyped(rs, input.getType());

        // Load up an instance of the specific script that we want to use.
        ScriptIntrinsicBlur script = ScriptIntrinsicBlur.create(rs, Element.U8_4(rs));
        script.setInput(input);

        // Set the blur radius
        script.setRadius(10);

        // Start the ScriptIntrinisicBlur
        script.forEach(output);

        // Copy the output to the blurred bitmap
        output.copyTo(blurredBitmap);

        toTransform.recycle();

        return blurredBitmap;
    }

    @Override
    public String getId() {
        return "blur";
    }
}
```

再说一次，如果你对于代码块 transform() 里的实现是困惑的，去读之前的博客，getId() 方法描述了这个转换的唯一标识符。Glide 使用该键作为缓存系统的一部分，为了避免意外的问题，你要确保它是唯一的。

下一节，我们要学习如何应用我们之前创建的转换。

### 单个转换的应用

Glide 有两种方式去使用转换。首先是传一个的你的类的实例作为参数给 .transform()。你这里你可以使用任何转换，无论它是否是用于图像还是 Gif。其他选择是使用 .bitmapTransform()，它只能用于 bitmap 的转换。因为我们上面的实现是为 bitmap 设计的，这两者我们都可以用：

```java
Glide  
    .with( context )
    .load( eatFoodyImages[0] )
    .transform( new BlurTransformation( context ) )
    //.bitmapTransform( new BlurTransformation( context ) ) // this would work too!
    .into( imageView1 );
```

### 运用多种转换

通常，Glide 的流式接口允许方法以链式的形式。然而对于转换却并不在这种场景下。确保你只调用了一次 .transform() 或 .bitmapTransform()，否则，之前的配置就会被覆盖掉的！然而，你还是可以运用多种转换的，通过传递多个转换对象作为参数传给 .transform() 或 .bitmapTransform()。

```java
Glide  
    .with( context )
    .load( eatFoodyImages[1] )
    .transform( new GreyscaleTransformation( context ), new BlurTransformation( context ) )
    .into( imageView2 );
```
这个代码片段中，我们把一个图像设置了灰度，然后做了模糊。Glide 为你自动执行了这两个转换。Awesome!

提示：当你用了转换后你就不能使用 .centerCrop() 或 .fitCenter() 了。

### Glide 转换集合

如果你已经有了做什么样转换的想法，你可以会想要用到你的 App 里，花点时间看下这个库：glide-transformations。它为 Glide 转换提供了多种多样的实现。非常值得去看一下，说不定你的想法已经在它那里实现了。

这个库有两个不同的版本。扩展版本包含了更多的转换，它是通过设备的 GPU 来计算处理的。这个版本需要有额外的依赖，所以这两个版本的设置有一点不同。你应该看看所拥有的转换方法的列表，再去决定你需要使用哪个版本。

### Glide 转换设置

设置起来很简单，对于基础版本你只需要在你当前的 build.gradle 中添加一行代码就可以了。

```java
dependencies {  
    compile 'jp.wasabeef:glide-transformations:1.2.1'
}
```

如果你想要使用 GPU 转换：

```java
repositories {  
    jcenter()
    mavenCentral()
}

dependencies {  
    compile 'jp.wasabeef:glide-transformations:1.2.1'
    compile 'jp.co.cyberagent.android.gpuimage:gpuimage-library:1.3.0'
}
```

如果你想使用 BlurTransformation，你需要多一个步骤。如果你还没做的话，那就添加下面这些代码到你的 build.gradle 中。

```java
android {  
    ...
    defaultConfig {
        ...
        renderscriptTargetApi 23
        renderscriptSupportModeEnabled true
    }
}
```

### 使用 Glide 的转换

当你将 build.gradle 文件在 Android Studio 同步了之后，你可以去使用这个转换集合了。使用模式和你自己定义转换的方式相同。假设我们想要做用这个集合的模糊转换去模糊一张图片：

```java
Glide  
    .with( context )
    .load( eatFoodyImages[2] )
    .bitmapTransform( new jp.wasabeef.glide.transformations.BlurTransformation( context, 25, 2 ) )
    .into( imageView3 );
```

就像我们上面所以用的，你也可以使用一连串的转换。.bitmapTransform() 方法都接受一个或多个转换。

## 用animate()自定义动画

### 动画基础

从图像到图像的平滑过渡是非常重要的。用户不喜欢在应用中出现突然的转变。这就是 Glide 要做的。Glide 中有一个标准动画去柔软的在你的 UI 中改变。我们在之前的博客 看了 .crossFade()。

我们要去看看除了 .crossFade() 的其他选择。Glide 提供了两个选项去设置一个动画。两个版本都是在 animate() 中，但传的参数并不同。

在我们之前代码，我们指出，动画仅仅用于不从缓存中加载的情况。如果图片被缓存过了，它的显示是非常快的，因此动画是没有必要的，并且不显示的。

### 从资源中的动画

回到代码，第一个选项是传一个 Android 资源 id，即动画的资源。一个简单的例子是每个 Android 系统都提供的：slide-in-left(从左滑入)动画，android.R.anim.slide_in_left。下面这段代码是这个动画的 XML 描述：

```xml
<?xml version="1.0" encoding="utf-8"?>  
<set xmlns:android="http://schemas.android.com/apk/res/android">  
    <translate android:fromXDelta="-50%p" android:toXDelta="0"
            android:duration="@android:integer/config_mediumAnimTime"/>
    <alpha android:fromAlpha="0.0" android:toAlpha="1.0"
            android:duration="@android:integer/config_mediumAnimTime" />
</set> 
```
当然你可以创建你自己的 XML 动画。比如一个小的缩放动画，图片刚开始小的，然后逐渐增大到原尺寸。

```xml
<?xml version="1.0" encoding="utf-8"?>  
<set xmlns:android="http://schemas.android.com/apk/res/android"  
     android:fillAfter="true">

    <scale
        android:duration="@android:integer/config_longAnimTime"
        android:fromXScale="0.1"
        android:fromYScale="0.1"
        android:pivotX="50%"
        android:pivotY="50%"
        android:toXScale="1"
        android:toYScale="1"/>
</set>  
```

这两个动画都可以用到 Glide 建造者中：

```java
Glide  
    .with( context )
    .load( eatFoodyImages[0] )
    .animate( android.R.anim.slide_in_left ) // or R.anim.zoom_in
    .into( imageView1 );
```
在图片从网络加载完并准备好之后将从左边滑入。

### 通过自定义类实现动画

当你想加载到常规的 ImageView 中这是没问题的。但是如果 target 是一些自定义的呢，比如我们之前在这篇博客 里所谈论过的？所以另外一个选项就非常有用了。通过传递一个动画资源的引用，你实现的一个类有 ViewPropertyAnimation.Animator 接口。

这个很简单，你只需实现 void animate(View view) 方法。这个视图对象是整个 target 视图。如果它是一个自定义的视图，你要找到你的视图的子元素，并且做些必要的动画。

来看个简单的例子。假设你想要实现一个渐现动画，你得需要创建这样的动画对象：

```java
ViewPropertyAnimation.Animator animationObject = new ViewPropertyAnimation.Animator() {  
    @Override
    public void animate(View view) {
        // if it's a custom view class, cast it here
        // then find subviews and do the animations
        // here, we just use the entire view for the fade animation
        view.setAlpha( 0f );

        ObjectAnimator fadeAnim = ObjectAnimator.ofFloat( view, "alpha", 0f, 1f );
        fadeAnim.setDuration( 2500 );
        fadeAnim.start();
    }
};
```

接下来，你需要在 Glide 请求中去设置这个动画：

```java
Glide  
    .with( context )
    .load( eatFoodyImages[1] )
    .animate( animationObject )
    .into( imageView2 );
```

当然，在 animate(View view) 中你的动画对象方法中， 你可以做任何你想要对视图做的事情。自由的用你的动画创建吧。

如果你要在你的自定义视图中实现，你只需要创建这个视图对象，然后在你的自定义视图中创建你的自定义方法。

## 集成网络栈

通过 HTTP/HTTPS 从网络上下载图像并显示是非常重要的一块。虽然标准的 Android 网络包也能做这些工作，但在 Android 中开发了很多提升网络的模块。每个库有它自己的优势和劣势。最后，这其实需要项目的配合和开发人员自己的品位来决定的。

Glide 的开发者不强制设置网络库给你，所以 Glide 可以说和 HTTP/S 无关。理论上，它可以与任何的网络库实现，只要覆盖了基本的网络能力就行。用 Glide 集成一个网络不是完全无缝的。它需要一个 Glide 的 ModeLoader 的接口。为了让你更加易用，Glide 为2个网络库提供了实现：OkHttp 和 Volley。

### OkHttp

假定你要集成 OkHttp 作为你给 Glide 的网络库。集成可以通过声明一个 GlideModule 手动实现。如果你想要避免手动实现，只需要打开你的 build.gradle 然后在你的依赖中添加下面这两行代码：

```java
dependencies {  
    // your other dependencies
    // ...

    // Glide
    compile 'com.github.bumptech.glide:glide:3.6.1'

    // Glide's OkHttp Integration 
    compile 'com.github.bumptech.glide:okhttp-integration:1.3.1@aar'
    compile 'com.squareup.okhttp:okhttp:2.5.0'
}
```

Gradle 会自动合并必要的 GlideModule 到你的 Android.Manifest。Glide 会认可在 manifest 中的存在，然后使用 OkHttp 做到所有的网络连接。

### Volley

另一方面，如果你偏爱使用 Volley，你必须改变你的 build.gradle 依赖：

```java
dependencies {  
    // your other dependencies
    // ...

    // Glide
    compile 'com.github.bumptech.glide:glide:3.6.1'

    // Glide's Volley Integration 
    compile 'com.github.bumptech.glide:volley-integration:1.3.1@aar'
    compile 'com.mcxiaoke.volley:library:1.0.8'
}
```

这将添加 Volley 并集成该库到你的项目中。集成库添加到 GlideModule 到你的 Android.Manifest。Glide 会自动认出它，然后使用 Volley 作为网络库。并不要求做其他的配置！

#### 警告：：如果你把这两个库都在你的 build.gradle 中声明了，那这两个库都会被添加。因为 Glide 没有任何特殊的加载顺序，你将会有一个不稳定的状态，它并不明确使用哪个网络库，所以确保你只添加了一个集成库。

### 其他网络库

如果你是别的网络库的粉丝，你是不幸的。Glide 除了 Volley 和 OkHttp 外不会自动配置其他的库。然而你随时可以整合你喜欢的网络库，在 GitHub 上去开一个 pull request。为Volley 和 OkHttp 可能给你一个方向。
