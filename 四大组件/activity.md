# Activity

> 简单点说：Activity就是布满整个窗口或者悬浮于其他窗口上的交互界面。

## 生命周期

![](/assets/Activity1.png)

三种静态状态，其它状态 \(Created与Started\)都是短暂的瞬态，Activity只能在三种状态之一下存在很长时间：

* **Resumed**：在这种状态下，Activity处于前台，且用户可以与其交互。（有时也称为“运行”状态。）
* **Paused**：在这种状态下，Activity被在前台中处于半透明状态或者未覆盖整个屏幕的另一个Activity—部分阻挡。暂停的Activity不会接收用户输入并且无法执行任何代码。
* **Stopped**：在这种状态下，Activity被完全隐藏并且对用户不可见；它被视为处于后台。停止时，Activity实例及其诸如成员变量等所有状态信息将保留，但它无法执行任何代码。

我们可以在_AndroidManifest.xml_中定义作为主activity的activity：

```xml
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

![](/assets/Activity2.png)

activity的第一个生命周期回调函数是 onCreate\(\),它最后一个回调是onDestroy\(\).当收到需要将该activity彻底移除的信号时，系统会调用这个方法。

> 大多数 app并不需要实现这个方法，因为局部类的references会随着activity的销毁而销毁，并且我们的activity应该在onPause\(\)与onStop\(\)中执行清除activity资源的操作。然而，如果activity含有在onCreate调用时创建的后台线程，或者是其他有可能导致内存泄漏的资源，则应该在OnDestroy\(\)时进行资源清理，杀死后台线程。

### 暂停与恢复

在正常使用app时，前端的activity有时会被其他可见的组件阻塞\(obstructed\)，从而导致当前的activity进入Pause状态。例如，当打开一个半透明的activity时\(例如以对话框的形式\)，之前的activity会被暂停。 只要之前的activity仍然被部分可见，这个activity就会一直处于Paused状态。然而，一旦之前的activity被完全阻塞并不可见时，则其会进入Stop状态。

当我们的activity收到调用onPause\(\)的信号时，那可能意味者activity将被暂停一段时间，并且用户很可能回到我们的activity。然而，那也是用户要离开我们的activtiy的第一个信号。

![](/assets/Activity3.png)

> 当一个半透明的activity阻塞activity时，系统会调用onPause\(\)方法并且这个activity会停留在Paused 状态\(1\). 如果用户在这个activity还是在Paused 状态时回到这个activity，系统则会调用它的onResume\(\) \(2\)。

当系统调用activity中的onPause\(\)，从技术上讲，意味着activity仍然处于部分可见的状态.但更多时候意味着用户正在离开这个activity，并马上会进入Stopped state. 通常应该在onPause\(\)回调方法里面做以下事情：

* 停止动画或者是其他正在运行的操作，那些都会导致CPU的浪费.
* 提交在用户离开时期待保存的内容\(例如邮件草稿\).
* 释放系统资源，例如broadcast receivers, sensors \(比如GPS\), 或者是其他任何会影响到电量的资源。

通常，不应该使用onPause\(\)来保存用户改变的数据 \(例如填入表格中的个人信息\) 到永久存储\(File或者DB\)上。

当用户从Paused状态恢复activity时，系统会调用onResume\(\)方法。系统每次调用这个方法时，activity都处于前台，包括第一次创建的时候。所以，应该实现onResume\(\)来初始化那些在onPause方法里面释放掉的组件，并执行那些activity每次进入Resumed state都需要的初始化动作。

### 停止与重启

在下面一些关键的场景中会涉及到停止与重启：

* 用户打开最近使用app的菜单并从我们的app切换到另外一个app，这个时候我们的app是被停止的。如果用户通过手机主界面的启动程序图标或者最近使用程序的窗口回到我们的app，那么我们的activity会重启。
* 用户在我们的app里面执行启动一个新activity的操作，当前activity会在第二个activity被创建后stop。如果用户点击back按钮，第一个activtiy会被重启。
* 用户在使用我们的app时接收到一个来电通话。

> 因为系统在activity停止时会在内存中保存Activity的实例，所以有时不需要实现onStop\(\),onRestart\(\)甚至是onStart\(\)方法. 因为大多数的activity相对比较简单，activity会自己停止与重启，我们只需要使用onPause\(\)来停止正在运行的动作并断开系统资源链接。

![](/assets/Activity4.png)

> 当activity调用onStop\(\)方法, activity不再可见，并且应该释放那些不再需要的所有资源。一旦activity停止了，系统会在需要内存空间时摧毁它的实例\(和栈结构有关，通常back操作会导致前一个activity被销毁\)。极端情况下，系统会直接杀死我们的app进程，并不执行activity的onDestroy\(\)回调方法, 因此我们需要使用onStop\(\)来释放资源，从而避免内存泄漏。\(这点需要注意\)

即使系统会在activity stop时停止这个activity，它仍然会保存View对象的状态\(比如EditText中的文字\) 到一个Bundle中，并且在用户返回这个activity时恢复它们。

> 我们在onStop里面做了哪些清除的操作，就该在onStart里面重新把那些清除掉的资源重新创建出来。

当系统Destory我们的activity，它会为activity调用onDestroy\(\)方法。因为我们会在onStop方法里面做释放资源的操作，那么onDestory方法则是我们最后去清除那些可能导致内存泄漏的地方。

### 重新创建

然而，如果因为系统资源紧张而导致Activity的Destory， 系统会在用户回到这个Activity时有这个Activity存在过的记录，系统会使用那些保存的记录数据（描述了当Activity被Destory时的状态）来重新创建一个新的Activity实例。那些被系统用来恢复之前状态而保存的数据被叫做 "instance state" ，它是一些存放在Bundle对象中的key-value pairs。\(请注意这里的描述，这对理解onSaveInstanceState执行的时刻很重要\)。

> 你的Activity会在每次旋转屏幕时被destroyed与recreated。当屏幕改变方向时，系统会Destory与Recreate前台的activity，因为屏幕配置被改变，你的Activity可能需要加载另一些替代的资源\(例如layout\)。



