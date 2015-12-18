# chapter-02-sharing-and-multimedia

Tags:Training-for-Android-developers

---

## 分享
### 分享简单的数据
在构建一个intent时，必须指定这个intent需要触发的actions。Android定义了一些actions，比如ACTION_SEND，该action表明该intent用于从一个activity发送数据到另外一个activity的，甚至可以是跨进程之间的数据发送。

    在不同的程序之间使用intent收发数据是在社交分享内容时最常用的方法。Intent使用户能够通过最常用的程序进行快速简单的分享信息。

```java
Intent sendIntent = new Intent();
sendIntent.setAction(Intent.ACTION_SEND);
sendIntent.putExtra(Intent.EXTRA_TEXT, "This is my text to send.");
sendIntent.setType("text/plain"); // 再度印证了这玩意像request
startActivity(sendIntent);
```
分享多张图片
```java
ArrayList<Uri> imageUris = new ArrayList<Uri>();
imageUris.add(imageUri1); // Add your image URIs here
imageUris.add(imageUri2);

Intent shareIntent = new Intent();
shareIntent.setAction(Intent.ACTION_SEND_MULTIPLE);
shareIntent.putParcelableArrayListExtra(Intent.EXTRA_STREAM, imageUris);
shareIntent.setType("image/*");
startActivity(Intent.createChooser(shareIntent, "Share images to.."));
```
分享多种不同类型的内容，需要使用ACTION_SEND_MULTIPLE与指定到那些数据的URIs列表。分享3张JPEG的图片，那么MIME类型仍然是image/jpeg。如果是不同图片格式的话，应该是用image/*来匹配那些可以接收任何图片类型的activity。如果完全不限格式就是 \*/\*。

### 接收数据

```xml
<activity android:name=".ui.MyActivity" >
    <intent-filter>
        <action android:name="android.intent.action.SEND" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="image/*" />
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.SEND" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="text/plain" />
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.SEND_MULTIPLE" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="image/*" />
    </intent-filter>
</activity>
```
代码处理
```java
void onCreate (Bundle savedInstanceState) {
    ...
    // Get intent, action and MIME type
    Intent intent = getIntent();
    String action = intent.getAction();
    String type = intent.getType();

    if (Intent.ACTION_SEND.equals(action) && type != null) {
        if ("text/plain".equals(type)) {
            handleSendText(intent); // Handle text being sent
        } else if (type.startsWith("image/")) {
            handleSendImage(intent); // Handle single image being sent
        }
    } else if (Intent.ACTION_SEND_MULTIPLE.equals(action) && type != null) {
        if (type.startsWith("image/")) {
            handleSendMultipleImages(intent); // Handle multiple images being sent
        }
    } else {
        // Handle other intents, such as being started from the home screen
    }
    ...
}

void handleSendText(Intent intent) {
    String sharedText = intent.getStringExtra(Intent.EXTRA_TEXT);
    if (sharedText != null) {
        // Update UI to reflect text being shared
    }
}

void handleSendImage(Intent intent) {
    Uri imageUri = (Uri) intent.getParcelableExtra(Intent.EXTRA_STREAM);
    if (imageUri != null) {
        // Update UI to reflect image being shared
    }
}

void handleSendMultipleImages(Intent intent) {
    ArrayList<Uri> imageUris = intent.getParcelableArrayListExtra(Intent.EXTRA_STREAM);
    if (imageUris != null) {
        // Update UI to reflect multiple images being shared
    }
}
```
### 大文件分享

分享大文件方式和分享文本和流相似，只不过它不发数据过去，传个引用过去。发的其实不是文件，是文件的URI。“服务端”app把数据分享给其他“客户端”app总共分三步：
一、生成文件的content URI。
二、授予URI的临时访问权限。
三、将URI发送给接收文件的应用程序。
“客户端”app要拿到这个文件，总共有两步：
一、发起请求。拿到URI。
二、通过URI拿到FileDescriptor，开始处理文件。
    
>  这一过程中没有文件的安全问题，因为客户端应用程序所收到的所有数据只有文件的Content URI而已。由于URI不包含目录路径信息，客户端应用程序无法查询或打开任何服务端应用程序的其他文件。客户端应用程序仅仅获取了这个文件的访问渠道以及由服务端应用程序授予的访问权限。同时访问权限是临时的，一旦这个客户端应用的任务栈结束了，这个文件将无法再被除服务端应用程序之外的其他应用程序访问。

### NFC传文件

>* Android 4.1（API Level 16）及以上版本的Android系统中使用。
>* 文件必须放置于外部存储。
>* 文件必须是全局可读的。我们可以通过File.setReadable(true,false)来为文件设置相应的读权限。

## Android多媒体

### 音频播放
Android为播放音乐，闹铃，通知铃，来电声音，系统声音，打电话声音与拨号声音分别维护了一个独立的音频流。音频操作在AudioManger中进行。
```java
AudioManager am = (AudioManager)activity.getSystemService(Context.AUDIO_SERVICE);
setVolumeControlStream(AudioManager.STREAM_MUSIC);
```
为了防止多个音乐播放应用同时播放音频，Android使用音频焦点（Audio Focus）来控制音频的播放——即只有获取到音频焦点的应用才能够播放音频。requestAudioFocus()方法可以获取我们希望得到的音频流焦点。

短暂的焦点锁定：当计划播放一个短暂的音频时使用（比如播放导航指示）。
永久的焦点锁定：当计划播放一个较长但时长可预期的音频时使用（比如播放音乐）。