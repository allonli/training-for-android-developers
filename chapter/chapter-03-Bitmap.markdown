# chapter-03-Bitmap

Tags:Training-for-Android-developers

---

## 位图（Bitmap）
和Drawable的区别：一个Bitmap对象是一张bitmap格式图片的表示(类似于Java.awt.image）。一个Drawable对象是“那些能够在其上面画的任意对象”，它也许是一个bitmap对象，也可能是一个solid color、一个其他Drawable对象的集合。 大多数Android UI框架喜欢用Drawable对象，而不是Bitmap对象。一个View可以接受任何Drawable对象作为background。

    Mobile devices typically have constrained system resources. Android devices can have as little as 16MB of memory available to a single application. The Android Compatibility Definition Document (CDD), Section 3.7. Virtual Machine Compatibility gives the required minimum application memory for various screen sizes and densities. Applications should be optimized to perform under this minimum memory limit. However, keep in mind many devices are configured with higher limits.

Android设备为每个app分配16MB的内存空间，虽然厂商会更改rom，定制更高的单个App内存空间，但是开发者应该依据最小的内存限制即16MB控制内存开销。
 
    Android设备屏幕大小不一，分辨率各不相同。如果在UI中需要加载一副大图片，明智的做法是先获取屏幕的分辨率，然后再决定加载图片的大小。在一款低分辨率的设备上加载高分辨率图片除了增加内存消耗，别无他用。因为设备最大能展示的清晰度为本身的屏幕分辨率。所以，如果一个不需要放大图片的控件需要加载图片，最大只需要加载该控件本身宽高的图片即可。

BitmapFactory提供了一些解码（decode）的方法（decodeByteArray(), decodeFile(), decodeResource()等），用来从不同的资源中创建一个Bitmap。

设置 inJustDecodeBounds 属性为true可以在解码的时候避免内存的分配，它会返回一个null的Bitmap，但是可以获取到 outWidth, outHeight 与 outMimeType。这样可以允许你在构造Bitmap之前优先读图片的尺寸与类型。

>*    BitmapFactory.Options 中设置 inSampleSize 的值。例如, 一个分辨率为2048x1536的图片，如果设置 inSampleSize 为4，那么会产出一个大约512x384大小的Bitmap。大小只是原来的1/16。
```java
// Calculate inSampleSize
options.inSampleSize = calculateInSampleSize(options, reqWidth, reqHeight);
```

在处理图片时，一般防止图将主线程卡住。一般会使用AsyncTask，将处理图片的过程放到单独的线程中处理。使用弱、软引用时，要注意到目前的虚拟机中GC的频率很高，在API LEVEL 11以前bitmap是放到native内存中，这样被回收后的状态是不可预知的，可能会产生crash。所以要慎重使用这两种引用方式。

使用缓存是处理图片时的常用方式，一般分为两种方式，内存缓存可以使用LruCache（本质就是一个LinkedHashMap）。也可以使用DiskLruCache。

