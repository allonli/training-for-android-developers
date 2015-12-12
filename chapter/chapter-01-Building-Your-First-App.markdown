## chapter-01-Building-Your-First-App

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

### Getting Started
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

    Andoird中有 View 和 ViewGroup 的概念。View 就是可见的控件，按钮啥的。类似一些GUI里的 Component ， ViewGroup类似一些语言 GUI 里的 Panel。
    ViewGroup 是不可见的 View 容器而已。为了不使这些 View 散乱和更易管理。把许多 View 放在 ViewGroup  中集中调整它的布局等属性。
    
wrap_content 根据控件文本等自适应大小。简单来讲：够用就得。
match_parent 充满整个父容器边界。

### 权重

*The weight value is a number that specifies the amount of remaining space each view should consume, relative to the amount consumed by sibling views. This works kind of like the amount of ingredients in a drink recipe: "2 parts soda, 1 part syrup" means two-thirds of the drink is soda. For example, if you give one view a weight of 2 and another one a weight of 1, the sum is 3, so the first view fills 2/3 of the remaining space and the second view fills the rest. If you add a third view and give it a weight of 1, then the first view (with weight of 2) now gets 1/2 the remaining space, while the remaining two each get 1/4.*

权重的值指的是每个部件所占剩余空间的大小，该值与同级部件所占空间大小有关。就类似于饮料的成分配方：“两份伏特加酒，一份咖啡利口酒”，即该酒中伏特加酒占三分之二。例如，我们设置一个View的权重是2，另一个View的权重是1，那么总数就是3，这时第一个View占据2/3的空间，第二个占据1/3的空间。如果你再加入第三个View，权重设为1，那么第一个View(权重为2的)会占据1/2的空间，剩余的另外两个View各占1/4。(请注意，使用权重的前提一般是给View的宽或者高的大小设置为0dp，然后系统根据上面的权重规则来计算View应该占据的空间。但是很多情况下，如果给View设置了match_parent的属性，那么上面计算权重时则不是通常的正比，而是反比，也就是权重值大的反而占据空间小)。
对于所有的View默认的权重是0，如果只设置了一个View的权重大于0，则该View将占据除去别的View本身占据的空间的所有剩余空间。因此这里设置EditText的权重为1，使其能够占据除了按钮之外的所有空间。
    
**总结：权值PK，此消彼长。**

### Activity 间传递及拉起

两个界面 （Activity） 之间如果要传值或者一个拉起另一个的操作，都需要 Intent做为媒介（哪怕不传值，只是拉起下游 Activity 也要使用 Intet 源码中这里没有任何处理，只要你不传它就报 NPE）。Intent 特别类似于网络请求中的 Request 对象，如果服务端没接到 Request 对象，后面的业务也不会被拉起。同样的像 Request 一样，Intent 也可以做为信息传递的载体，putExtra方法和 Request 中的 setAttribute 一样。用 K->V 的方式存储信息。
