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

## ListAdapter(ListView, GridView)

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
## 占位符 和 渐现动画
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
