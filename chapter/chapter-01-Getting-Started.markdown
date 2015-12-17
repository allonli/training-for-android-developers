# chapter-01-Getting-Started

Tags: Training-for-Android-developers

---

这是一个读 google的 android 教程的整理。仅以记录，不求误人。卽此而已。

### 关于 IDE
欲善其事必利其器，在工具的对比和选择时间耗费整整用了一天。不过仍有磨刀不费砍柴功的感觉。直接上结论。

>*  eclipse pass ,其因不再赘述。
>*  IDEA + android support插件，和 android studio 是相同的代码 compile 出来的，只不过移植要慢一些。由于之前长期使用 idea，在上面有很多种类已经在用的代码插件，目前以来用它通吃了 python\scala\java\php。如果转到其他 IDE 势必会出现要使用两个 IDEA 的情况，不过在使用了一段时间 idea 打 android 插件的方式以后，发现对 android 开发的支持不太完备，用起来有些勉强。十分不情愿的放弃了它。 想说爱你不容易。
>*  Android Studio 实际上是 IDEA的太监版，试着装了一些诸如 scala 语言的插件，几乎是不可用的。但是！我的目的现在是找一个好用的 IDE for Android。试了一下它的所有android 相关功能非常不错。更重要的是 google官方教程也使用的这款 IDE。
>*    注（官网答复）： Android Studio and the Android plugin for IntelliJ IDEA are built from the same code, and all of the changes in Android Studio are, and will continue to be, available in IntelliJ IDEA releases.


### gradle
android 程序和 java 程序一样也需要编译好后打包发布。在 java   中早期使用的 ant 比较多。后期逐步由 maven 做了大部分工作。和 ant、maven 一样。gradle 是一种相对较新的打包发布工具。目前在 android 中应用较多。主要是 android studio 使它。其实就是 ant 的升级版，完全兼容 ant 同时可以使用 groovy 和 scala 一些 jvm 平台上的其他语言做为配置语言。比使用 XML 灵活一些（然并卵！谁没事天天配这东西）。类似于 scala 里的 sbt ， nodejs 里的 npm 。
有人写了一本书还不错：
https://dongchuan.gitbooks.io/gradle-user-guide-/content/index.html

最后，感觉对于 android 新人，完全没必要在这个上面浪费太多时间。现学现卖足矣。

---

## Getting Started
新建一个 android 工程。

activity ，一个 android 界面就是一个activity。它需要一个布局配置。如下：


    app/src/main/res/layout/activity_my.xml

全局描述文件：

    app/src/main/AndroidManifest.xml
    
gradle 配置文件：

    app/build.gradle

values、layout、menu 目录：

    values 目录常放一些数字和字符串的值，就是一些存值的配置。类似于 properties 文件。另外两个见名知义。
    
### 运行和Activity消息传递

* Andoird中有 View 和 ViewGroup 的概念。View 就是可见的控件，按钮啥的。类似一些GUI里的 Component ， ViewGroup类似一些语言 GUI 里的 Panel。
* ViewGroup 是不可见的 View 容器而已。为了不使这些 View 散乱和更易管理。把许多 View 放在 ViewGroup  中集中调整它的布局等属性。
    
wrap_content 根据控件文本等自适应大小。简单来讲：够用就得。
match_parent 充满整个父容器边界。

### 权重

*The weight value is a number that specifies the amount of remaining space each view should consume, relative to the amount consumed by sibling views. This works kind of like the amount of ingredients in a drink recipe: "2 parts soda, 1 part syrup" means two-thirds of the drink is soda. For example, if you give one view a weight of 2 and another one a weight of 1, the sum is 3, so the first view fills 2/3 of the remaining space and the second view fills the rest. If you add a third view and give it a weight of 1, then the first view (with weight of 2) now gets 1/2 the remaining space, while the remaining two each get 1/4.*

```
    权重的值指的是每个部件所占剩余空间的大小，该值与同级部件所占空间大小有关。
    就类似于饮料的成分配方：“两份伏特加酒，一份咖啡利口酒”，即该酒中伏特加酒占三分之二
    。例如，我们设置一个View的权重是2，另一个View的权重是1，那么总数就是3，这时第一个View占据2/3的空间，第二个占据1/3的空间。如果你再加入第三个View，权重设为1，那么第一个View(权重为2的)会占据1/2的空间，剩余的另外两个View各占1/4。(请注意，使用权重的前提一般是给View的宽或者高的大小设置为0dp，然后系统根据上面的权重规则来计算View应该占据的空间。
    但是很多情况下，如果给View设置了match_parent的属性，那么上面计算权重时则不是通常的正比，而是反比，也就是权重值大的反而占据空间小)。
    对于所有的View默认的权重是0，如果只设置了一个View的权重大于0，则该View将占据除去别的View本身占据的空间的所有剩余空间。
    因此这里设置EditText的权重为1，使其能够占据除了按钮之外的所有空间。
```
    
**总结：权值PK，此消彼长。**

### Activity 间传递及拉起

两个界面 （Activity） 之间如果要传值或者一个拉起另一个的操作，都需要 Intent做为媒介（哪怕不传值，只是拉起下游 Activity 也要使用 Intet 源码中这里没有任何处理，只要你不传它就报 NPE）。Intent 特别类似于网络请求中的 Request 对象，如果服务端没接到 Request 对象，后面的业务也不会被拉起。同样的像 Request 一样，Intent 也可以做为信息传递的载体，putExtra方法和 Request 中的 setAttribute 一样。用 K->V 的方式存储信息。

---

## 适配
```java
//判断指定版本下才可以运行，柔性可用
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
    ActionBar actionBar = getActionBar();
    actionBar.setDisplayHomeAsUpEnabled(true);
}
```

* 有4种普遍尺寸：小(small)，普通(normal)，大(large)，超大(xlarge)
* 4种普遍分辨率：低精度(ldpi), 中精度(mdpi), 高精度(hdpi), 超高精度(xhdpi)

**layout\国际化(语言)\图片 这些差别都是用目录来区分的。系统会根据设置或者默认拉起对应的配置。**


## Activity 的生命周期

 
![图1][1]

上图显示：当用户离开我们的activity时，系统会调用onStop()来停止activity (1). 这个时候如果用户返回，系统会调用onRestart()(2), 之后会迅速调用onStart()(3)与onResume()(4). 请注意：无论什么原因导致activity停止，系统总是会在onStop()之前调用onPause()方法。

* onResume: 初始化操作一般在这里做，onCreate 不要初始化太多东西，不然打开应用会很久看不到界面。
* onStop: 当 activity 隐藏以后要执行的heavy-load操作一般会在 onStop 中。不要把heavy-load操作放到 onPasue 这样会影响界面切换速度。写 DB 建议在此方法中实现。

几种会调用onDestroy的情况:

>*  用户点击 back 按钮
>*  程序里调用了 finish() 方法
>*  资源紧张时（后台长期 stop 状态或者前台需要更多内存），系统可能会干掉它。系统在 destroy 时把现场存到一个 Bundle 对象中的键值对中(instance state)，当用户要回到这个 Activity 时，系统会重建一个 Activity 实例（此 Activity 已是物是人非，非彼 Activity!）
>*  Activity 会在每次旋转屏幕时被 destroyed 与 recreated 。当屏幕改变方向时，系统会 Destory 与 Recreate 前台的 activity ，因为屏幕配置被改变，Activity 可能需要加载另一些替代的资源(例如layout)。为了让所有信息顺利


![图2][2]

**调用 onSaveInstanceState 的时机：**
onSaveInstanceState()的调用遵循一个重要原则，即当系统存在“未经你许可”时销毁了我们的activity的**可能**时，则onSaveInstanceState()**可能**会被系统调用，这是系统的责任，因为它必须要提供一个机会让你保存你的数据（当然你不保存那就随便你了）。如果调用，调用将发生在onPause()或onStop()方法之前。
　1. 用户按下HOME键时。
　2. 长按HOME键，选择运行其他的程序时。
　3. 按下电源按键（关闭屏幕显示）时。
　4. 从activity A中启动一个新的activity时。
　5. 屏幕方向切换时，例如从竖屏切换到横屏时。

**调用 onRestoreInstanceState 的时机：**
只有在Activity真的被系统非正常干掉过，恢复显示Activity的时候，才会调用onRestoreInstanceState。

## Fragments

![图3][3]

Fragment是为了activity的模块化而出现的概念，它拥有自己的生命周期，接收自己的输入事件，可以在acvitity运行过程中添加或者移除（有点像"子activity"，可以在不同的activity里面重复使用，当通过XML布局文件的方式将Fragment添加进activity时，Fragment是不能被动态移除的。如果想要在用户交互的时候把fragment切入与切出，必须在activity启动后，再将fragment添加进activity。）。

![图4][4]

* 为了执行fragment的增加或者移除操作，必须通过 FragmentManager 创建一个FragmentTransaction对象,FragmentTransaction提供了用来增加、移除、替换以及其它一些操作的APIs。

* 如果我们的activity允许fragment移除或者替换，我们应该在activity的onCreate()方法中添加初始化fragment(s).

* fragment其实只是一个虚拟的框，它里面不装东西它就什么也不是。所以必须有一个View把它撑起来（inflate）。
```java
inflater.inflate(R.layout.article_view, container, false);
```

移除或者替换Fragment，如果要适当地让用户可以向后导航与"撤销"这次改变。为了让用户向后导航fragment事务，我们必须在FragmentTransaction提交前调用addToBackStack()方法。（如果不加到栈顶，点返回键栈里没东西时，会直接返回到桌面）
```java
transaction.addToBackStack(null);
```
  
## 数据持久化
为了能方便这里的实验，手机root以后。可以用以下两种方式查看系统文件：

>* 手机连usb后将android sdk的platform加到PATH中，执行adb shell，su - root。就可以切到root下，为所欲为（君子有所为有所不为）。
>* 也可以在手机端安装一个ssh server软件（如：SSHDriodPro）,启动服务，在确认软件拿到root权限同时pc和手机在相同网段后，pc上使用ssh root@[ip] -p [port]连上，再su - root。效果同上。

### 保存到Preference
preference有些类似cookie。每个Prefernece文件就是存一些k->v。

* getSharedPreferences() 可以取多个带名文件，当文件不存在的时间，get时会自动创建。
* getPreference() 不需要名，只取一个文件。 
* 当写文件时，Preference的Editor对象要commit一下才会生效。

> 当我们用root登录到手机上，cd /data/data/com.example.android.fragments/ (包名根据具体app配置决定)，之后我们能看到一个叫shared_prefs文件夹。
进入这个文件夹后看到：MainActivity.xml和preference_file_key.xml。


单个文件，无需指定文件名方式
```xml
<!--MainActivity.xml-->
<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
<map>
    <string name="preference">preference1</string>
</map>
```
自己指定文件名来存储
```xml
<!--preference_file_key.xml-->
<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
<map>
    <string name="sharedPref">SharedPreferences1</string>
</map>
```
再对比一下代码，司马昭之心路人皆知：***所谓的Preference就是一个xml文件而已***
```java
public void saveData(){
    SharedPreferences sharedPref = getSharedPreferences();
    SharedPreferences preference = getPreference();
    
    if (sharedPref.getString("sharedPref", null) == null) {
        SharedPreferences.Editor editor = sharedPref.edit();
        editor.putString("sharedPref", "SharedPreferences1");
        editor.commit();
    }
    
    if (preference.getString("preference", null) == null) {
        SharedPreferences.Editor editor = preference.edit();
        editor.putString("preference", "preference1");
        editor.commit();
    }
}

/**
 * 获取preferences
 *
 * @return
 */
private SharedPreferences getSharedPreferences() {
    return getActivity().getSharedPreferences(getString(R.string.preference_file_key), Context.MODE_PRIVATE);
}

/**
 * 获取单个文件
 *
 * @return
 */
private SharedPreferences getPreference() {
    return getActivity().getPreferences(Context.MODE_PRIVATE);
}
```

### 存文件

也可以直接操作文件来读写，分为External和Internal两种文件操作。使用的API都是java操作文件IO的标准API。当写完文件以后，可以用adb在上面提到的shared_prefs同级目录中查看到一个files和caches目录分别存储用来存普通文件和临时文件。

> 在写入文件时，目录是可以指定的。但是一般情况下的权限只能指定自己app所对应的目录。其他程序目录如果强制被chmod授权，当前程序也可以写到那个目录，或者当前程序拿到root权限也可随便写。

### 写DB

Android的自带DB规范是sqlite。操作和java中的一些操作DB方式基本相同。DB文件存在/data/data/[应用]/databases下。可以使用sqlite3命令查看。如果手机上没有安装，可以从模拟器中pull出来，再push到手机上。前提是手机得root。

搞定以后直接sqlite3 xxx.db。然后sql语句就ok.

> 在调试过程中，经常会有查询DB中的数据需要。除了命令行，也可以使用一个sqlite editor pro的软件。它可以帮你搜索整个手机中的DB数据，直观查看。

查询
```java
//要查哪些列
String[] projection = {
        FeedReaderContract.FeedEntry._ID,
        FeedReaderContract.FeedEntry.COLUMN_NAME_TITLE
};
//排序参数，也可以加各种where
String sortOrder = FeedReaderContract.FeedEntry._ID + " DESC";
//拿到游标
Cursor cursor = db.query(
        FeedReaderContract.FeedEntry.TABLE_NAME,  // The table to query
        projection,                               // The columns to return
        null,                                // The columns for the WHERE clause
        null,                            // The values for the WHERE clause
        null,                                     // don't group the rows
        null,                                     // don't filter by row groups
        sortOrder                                 // The sort order
);
//移到头
cursor.moveToFirst();
//一般于用do while，判断isLast，然后moveToNext。相当于jdbc中的rs.next()。
cursor.moveToNext();
```
写
```java
ContentValues values = new ContentValues();
values.put(FeedReaderContract.FeedEntry.COLUMN_NAME_ENTRY_ID, i);
values.put(FeedReaderContract.FeedEntry.COLUMN_NAME_TITLE, "title" + i);

long newRowId = db.insert(FeedReaderContract.FeedEntry.TABLE_NAME, null, values);

list.add(newRowId);
```
删除
```java
db.delete
```
更新
```java
db.update
```




  [1]: http://developer.android.com/images/training/basics/basic-lifecycle-paused.png
  [2]: http://developer.android.com/images/training/basics/basic-lifecycle-savestate.png
  [3]: http://developer.android.com/images/fundamentals/fragments.png
  [4]: http://developer.android.com/images/fragment_lifecycle.png
