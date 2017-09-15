# Activity

> 简单点说：Activity就是布满整个窗口或者悬浮于其他窗口上的交互界面。

## 生命周期

![](/assets/Activity1.png)

三种静态状态，其它状态 \(Created与Started\)都是短暂的瞬态，Activity只能在三种状态之一下存在很长时间：

* **Resumed**：在这种状态下，Activity处于前台，且用户可以与其交互。（有时也称为“运行”状态。）
* **Paused**：在这种状态下，Activity被在前台中处于半透明状态或者未覆盖整个屏幕的另一个Activity—部分阻挡。暂停的Activity不会接收用户输入并且无法执行任何代码。
* **Stopped**：在这种状态下，Activity被完全隐藏并且对用户不可见；它被视为处于后台。停止时，Activity实例及其诸如成员变量等所有状态信息将保留，但它无法执行任何代码。

我们可以在_AndroidManifest.xml_中定义作为主activity的activity：

```java
<activity android:name=".MainActivity" android:label="@string/app_name">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

### 启动与销毁

我们必须实现onCreate\(\)方法来执行程序启动所需要的基本逻辑，一旦onCreate 操作完成，系统会迅速调用onStart\(\) 与onResume\(\)方法。我们的activity不会在Created或者Started状态停留。

技术上来说, activity在onStart\(\)被调用后开始被用户可见，但onResume\(\)会迅速被执行使得activity停留在Resumed状态，直到一些因素发生变化才会改变这个状态。例如接收到一个来电，用户切换到另外一个activity，或者是设备屏幕关闭。

activity的第一个生命周期回调函数是 onCreate\(\),它最后一个回调是onDestroy\(\).当收到需要将该activity彻底移除的信号时，系统会调用这个方法。

> 大多数 app并不需要实现这个方法，因为局部类的references会随着activity的销毁而销毁，并且我们的activity应该在onPause\(\)与onStop\(\)中执行清除activity资源的操作。然而，如果activity含有在onCreate调用时创建的后台线程，或者是其他有可能导致内存泄漏的资源，则应该在OnDestroy\(\)时进行资源清理，杀死后台线程。

### 暂停与恢复

在正常使用app时，前端的activity有时会被其他可见的组件阻塞\(obstructed\)，从而导致当前的activity进入Pause状态。例如，当打开一个半透明的activity时\(例如以对话框的形式\)，之前的activity会被暂停。 只要之前的activity仍然被部分可见，这个activity就会一直处于Paused状态。然而，一旦之前的activity被完全阻塞并不可见时，则其会进入Stop状态。

当我们的activity收到调用onPause\(\)的信号时，那可能意味者activity将被暂停一段时间，并且用户很可能回到我们的activity。然而，那也是用户要离开我们的activtiy的第一个信号。

