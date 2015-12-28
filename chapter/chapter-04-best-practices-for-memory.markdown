# chapter-04-best-practices-for-memory

Tags:Training-for-Android-developers

---

## 管理内存

Android没有使用Linux中的swap的文件交换区方式。但是还是使用了纯内存paging（以下称内存页）和mmapped（文件内存映射）。为了满足每个app对RAM的需要，Android在进程间共享内存页。

### 共享内存

**通常遵循下面方式实现：**

>* 每一个app的进程都是从一个被叫做Zygote的进程中fork出来的（Android也是基于COW，所以性能还不错）。那么理所当然的继承了父进程内存页。
    
具体过程是：

        开机后系统会通过init.rc配置（例如小米4：service zygote /system/bin/app_process -Xzygote /system/bin --zygote --start-system-server）先拉起Zygote进程，这个进程启动后自动加载android框架代码和资源（例如activity themes）然后再fork出具体的app进程，那这样一来就可以让所有的android app进程共享框架load的内存资源。

>* 大多数静态数据被映射到进程中。这样让的数据在进程间共享。

    静态数据包括：
    1、Dalvik 代码 (放在一个预链接好的.odex 文件中以便直接mapping)
    2、App Resources(它们被组织成资源映射表的结构，然后把APK中的文件做对齐的操作来，以便更容易做内存映射）
    3、native代码如.so文件。

>* 用户显示的分配内存如：ashmem pmem gralloc等方式。
    
如要查看分析共享内存：
http://developer.android.com/tools/debugging/debugging-memory.html

### Android系统对内存的分配和回收
>* Android虚拟机和java的结构非常类似，主要对象的存储也是在堆上。(不过Android是跑在别人的手机里。所以你不能像服务器随便设置虚拟机堆的各项参数。虚拟机又不是咱们家的，当然不可以这么干)。系统会限制单个应用的堆内存大小。

>* heap的逻辑大小和实际使用的物理内存大小是不相同的。假设共享内存大小是10M，一共有20个Process在共享使用，根据权重，可能认为其中有0.3M才是你的进程所使用的大小。这个实际的值叫PSS（Proportional Set Size）。所以我们只需要查看Total Pss的值就可以知道该应用运行时所占的内存的大小。

>* 系统不会做内存碎片整理，android只会清一下堆的未端不使用的内存。虽然不做碎片整理，但是它会在GC之后，挑出整块不使用的内存页干掉。不过，如果是共有的，很小的内存就不行了。

adb shell dumpsys meminfo com.baidu.vsfinance -d

```bash
Applications Memory Usage (kB):
Uptime: 18460072 Realtime: 105595846

** MEMINFO in pid 19441 [com.baidu.vsfinance] **
                   Pss  Private  Private  Swapped     Heap     Heap     Heap
                 Total    Dirty    Clean    Dirty     Size    Alloc     Free
                ------   ------   ------   ------   ------   ------   ------
  Native Heap       24       24        0        0    15920     7033      502
  Dalvik Heap    20988    20604        0        0    42724    40882     1842
 Dalvik Other     3404     3352        0        0                           
        Stack      192      192        0        0                           
    Other dev    12400     9444        4        0                           
     .so mmap     2527     1436      524        0                           
    .jar mmap        4        0        4        0                           
    .apk mmap      344        0      228        0                           
    .ttf mmap      390        0      136        0                           
    .dex mmap     3778       72     3324        0                           
   Other mmap       62       12       36        0                           
      Unknown     3843     3840        0        0                           
        TOTAL    47956    38976     4256        0    58644    47915     2344
 
 Objects
               Views:      263         ViewRootImpl:        6
         AppContexts:        6           Activities:        3
              Assets:        5        AssetManagers:        5
       Local Binders:       19        Proxy Binders:       25
    Death Recipients:        1
     OpenSSL Sockets:        0
 
 SQL
         MEMORY_USED:      291
  PAGECACHE_OVERFLOW:       77          MALLOC_SIZE:       62
 
 DATABASES
      pgsz     dbsz   Lookaside(b)          cache  Dbname
         4       32             45         2/17/3  /data/data/com.baidu.vsfinance/databases/afinal.db
```

### API获取内存相关信息

除了使用dumpsys命令外，也可以使用ActivityManager类的查看各种内存信息，这个方式好在它可以实时查，另外使用getMemoryClass()可以查看当前系统对单个app的最大内存限制。
        
>    在一些特殊的情景下，可以通过在manifest的application标签下添加largeHeap=true的属性来声明一个更大的heap空间。如果这样做你可以通过getLargeMemoryClass())来获取到一个更大的heap size。
不要轻易的因为你需要使用大量的内存而去请求一个大的heap size。只有当你清楚的知道哪里会使用大量的内存并且为什么这些内存必须被保留时才去使用large heap。
另外, large heap并不一定能够获取到更大的heap。在某些有严格限制的机器上，large heap的大小和通常的heap size是一样的。因此即使你申请了large heap，你还是应该通过执行getMemoryClass()来检查实际获取到的heap大小。

ComponentCallbacks2类有一个onTrimMemory(int)回调，接受的int值表示现在的内存状态。比如ComponentCallbacks2.TRIM_MEMORY_RUNNING_MODERATE表示app正在运行并且不会被列为可杀死的。但是设备此时正运行于低内存状态下，系统开始触发杀死LRU Cache中的Process的机制。
具体还有很多值，都在：http://developer.android.com/reference/android/content/ComponentCallbacks2.html#onTrimMemory(int)

因为android的内存不像java一样舒爽，所以有些内存数据也要适当关注一下：

Enums often require more than twice as much memory as static constants. You should strictly avoid using enums on Android.
Every class in Java (including anonymous inner classes) uses about 500 bytes of code.
Every class instance has 12-16 bytes of RAM overhead.
Putting a single entry into a HashMap requires the allocation of an additional entry object that takes 32 bytes.

>* Enums的内存消耗通常是static constants的2倍。你应该尽量避免在Android上使用enums。
>* 在Java中的每一个类(包括匿名内部类)都会使用大概500 bytes。
>* 每一个类的实例花销至少是12-16 bytes。
>* 往HashMap添加一个entry需要额一个额外占用的32 bytes的entry对象。

Often, developers use abstractions simply as a "good programming practice," because abstractions can improve code flexibility and maintenance. However, abstractions come at a significant cost: generally they require a fair amount more code that needs to be executed, requiring more time and more RAM for that code to be mapped into memory. So if your abstractions aren't supplying a significant benefit, you should avoid them.

>* 更不爽的是，在google文档中居然还明确指出，没有特别明显的优化效果，不要使用抽象。这在设计时不免可惜。

>* 为了降低内存占用，将部分内容放在服务端，同时google推荐使用他们的“homebrew(家酿)”产品protobuf的微缩版nano protobuf。

>* 为了节省资源，google也不建议你使用类似spring的诸如Guice、RoboGuice等依赖注入框架（还有天理吗）。

每次限定使用时，google都非常open的告诉你这个东西好。然后跟一个but:它耗资源。然并卵！

>* ProGuard能够通过移除不需要的代码，重命名类，域与方法等方对代码进行压缩，优化与混淆。使用ProGuard可以使得你的代码更加紧凑，这样能够使用更少mapped代码所需要的RAM。http://developer.android.com/tools/help/proguard.html

zipalign这个东西可以帮你的apk做对齐，这个进内存的效率提升和空间节省都会有的。而且有些图片儿啥的不能做内存mapped。必须要对齐一下。而且Google Play不接受没有经过zipalign的APK。
http://developer.android.com/tools/help/zipalign.html