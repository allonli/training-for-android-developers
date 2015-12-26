# chapter-03-bitmap-and-background-threads

Tags:Training-for-Android-developers

---

## 位图（Bitmap）
和Drawable的区别：一个Bitmap对象是一张bitmap格式图片的表示(类似于Java.awt.image）。一个Drawable对象是“那些能够在其上面画的任意对象”，它也许是一个bitmap对象，也可能是一个solid color、一个其他Drawable对象的集合。 大多数Android UI框架喜欢用Drawable对象，而不是Bitmap对象。一个View可以接受任何Drawable对象作为background。

>*    Mobile devices typically have constrained system resources. Android devices can have as little as 16MB of memory available to a single application. The Android Compatibility Definition Document (CDD), Section 3.7. Virtual Machine Compatibility gives the required minimum application memory for various screen sizes and densities. Applications should be optimized to perform under this minimum memory limit. However, keep in mind many devices are configured with higher limits.

Android设备为每个app分配16MB的内存空间，虽然厂商会更改rom，定制更高的单个App内存空间，但是开发者应该依据最小的内存限制即16MB控制内存开销。
 
>*    Android设备屏幕大小不一，分辨率各不相同。如果在UI中需要加载一副大图片，明智的做法是先获取屏幕的分辨率，然后再决定加载图片的大小。在一款低分辨率的设备上加载高分辨率图片除了增加内存消耗，别无他用。因为设备最大能展示的清晰度为本身的屏幕分辨率。所以，如果一个不需要放大图片的控件需要加载图片，最大只需要加载该控件本身宽高的图片即可。

BitmapFactory提供了一些解码（decode）的方法（decodeByteArray(), decodeFile(), decodeResource()等），用来从不同的资源中创建一个Bitmap。

设置 inJustDecodeBounds 属性为true可以在解码的时候避免内存的分配，它会返回一个null的Bitmap，但是可以获取到 outWidth, outHeight 与 outMimeType。这样可以允许你在构造Bitmap之前优先读图片的尺寸与类型。

>*    BitmapFactory.Options 中设置 inSampleSize 的值。例如, 一个分辨率为2048x1536的图片，如果设置 inSampleSize 为4，那么会产出一个大约512x384大小的Bitmap。大小只是原来的1/16。
```java
// Calculate inSampleSize
options.inSampleSize = calculateInSampleSize(options, reqWidth, reqHeight);
```

在处理图片时，一般防止图将主线程卡住。一般会使用AsyncTask，将处理图片的过程放到单独的线程中处理。使用弱、软引用时，要注意到目前的虚拟机中GC的频率很高，在API LEVEL 11以前bitmap是放到native内存中，这样被回收后的状态是不可预知的，可能会产生crash。所以要慎重使用这两种引用方式。

使用缓存是处理图片时的常用方式，一般分为两种方式，内存缓存可以使用LruCache（本质就是一个LinkedHashMap）。也可以使用DiskLruCache。

## Android后台任务

### IntentService
>* The IntentService class provides a straightforward structure for running an operation on a single background thread.
IntentService类提供了一个简单的后台单线程的实现。

但是IntentService有下面几个局限性：
An IntentService has a few limitations:
>* It can't interact directly with your user interface. To put its results in the UI, you have to send them to an Activity.
不可以直接和UI做交互。为了把他执行的结果体现在UI上，需要把结果返回给Activity。

>* Work requests run sequentially. If an operation is running in an IntentService, and you send it another request, the request waits until the first operation is finished.
工作任务队列是顺序执行的，如果一个任务正在IntentService中执行，此时你再发送一个新的任务请求，这个新的任务会一直等待直到前面一个任务执行完毕才开始执行。

>* An operation running on an IntentService can't be interrupted.
正在执行的任务无法打断。

>* **这么一大堆，其实就是在说：这只是一个方便使用的单线程。**

主线程拉起一个Inservice线程，并和IntentService来来回回没完没了的交互。总（long）共分四步：

* 建一个IntentService子类，实现里面的回调方法onHandleIntent。
* 在主线程创建一个IntentService子类的对象。并拉起来。
* 建一个Receiver，并实现回调。
* 把Receiver注册上。（把冰箱门关上）

#### Let's Go !

1, 创建一个IntentService class
```xml
<application
    android:icon="@drawable/icon"
    android:label="@string/app_name">
    ...
    <!--
        Because android:exported is set to "false",
        the service is only available to this app.
    -->
    <service
        android:name=".RSSPullService"
        android:exported="false"/>
    ...
<application/>
```
java code
```java
public class RSSPullService extends IntentService {
    @Override
    protected void onHandleIntent(Intent workIntent) {
        // Gets data from the incoming Intent
        String dataString = workIntent.getDataString();
        // to do something bala bala...
    }
}
```
2, 创建一个具体的Intent对象，并用来启动IntentService。
```java
/*
 * Creates a new Intent to start the RSSPullService
 * IntentService. Passes a URI in the
 * Intent's "data" field.
 */
mServiceIntent = new Intent(getActivity(), RSSPullService.class);
mServiceIntent.setData(Uri.parse(dataUrl));
// Let's go!
getActivity().startService(mServiceIntent);
```
3, 如果需要发送数据给主线程
```java
Intent localIntent = new Intent(Constants.BROADCAST_ACTION);
// Puts the status into the Intent
localIntent.putExtra(Constants.EXTENDED_DATA_STATUS, status);
oadcastManager.getInstance(this).sendBroadcast(localIntent);
```
定义主线程接收消息的类，需要一个BroadcastReceiver的子类。并复写onReceive。
```java
// Broadcast receiver for receiving status updates from the IntentService
private class ResponseReceiver extends BroadcastReceiver
{
    // Prevents instantiation
    private DownloadStateReceiver() {
    }
    // Called when the BroadcastReceiver gets an Intent it's registered to receive
    @
    public void onReceive(Context context, Intent intent) {
...
        /*
         * Handle Intents here.
         */
...
    }
}
```
4, 即使有了receiver类，如果想要收到子线程发的消息。还要让程序知道，接收什么样的消息，用谁来接收。
```java
// 接收什么样的消息
IntentFilter statusIntentFilter = new IntentFilter(Constants.BROADCAST_ACTION);
statusIntentFilter.addCategory(Intent.CATEGORY_DEFAULT);

// 用谁来接收（注册receiver）
mDownloadStateReceiver = new ResponseReceiver();
LocalBroadcastManager.getInstance(this).registerReceiver(mDownloadStateReceiver, statusIntentFilter);
```

### 点亮屏幕

```java
getWindow().addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);
//清除flag回到正常状态
getWindow().clearFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON)
```
也可以AndroidManifest.xml里定义该属性，由于在AndroidManifest里定义的内容不能动态改变。所以设了就不能反悔。
