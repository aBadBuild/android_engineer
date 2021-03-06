# Intent

一个Android app通常都会有多个activities。为了让用户能够从一个activity跳到另一个activity以及数据交互的时候，我们的app必须使用Intent来定义自己的意图。

## Intent的发送

### 建立显性（explicit）Intent

> 用于同一个app的两个Acticity之间的切换。

创建一个Intent去启动目标Activity：

```java
Intent intent = new Intent(this, 目标.class);
```

在这个Intent构造函数中有两个参数：

* 第一个参数是Context\(之所以用this是因为当前Activity是Context的子类\)
* 接受系统发送Intent的应用组件的Class（在这个案例中，指将要被启动的activity）。

新Activity的启动：

```java
startActivity(intent);
```

Intent可以携带称作_extras_的键-值对数据类型。putExtra\(\)方法把键名作为第一个参数，把值作为第二个参数。

```java
intent.putExtra(EXTRA_MESSAGE, message);
```

当然，为了应用程序与其他应用程序进行交互时仍可以确保键唯一，可以在原Activity中定义健常量：

```java
public class MyActivity extends ActionBarActivity {
    public final static String EXTRA_MESSAGE = "com.mycompany.myfirstapp.MESSAGE";
    // TODO
}
```

而在新的Activity接受Intent的时候，可以如下进行获取数据：

```java
Intent intent = getIntent(); //得到intent并赋值给本地变量
String message = intent.getStringExtra(MyActivity.EXTRA_MESSAGE); //可以通过原Activity里的常量来获取键值
map=(HashMap)intent.getSerializableExtra(MyActivity.EXTRA_MESSAGE); //获取键值对
```

### 建立隐性（implicit）Intent

> 用于唤醒不同app来执行某个动作。

intents并不声明要启动组件的具体类名，而是声明一个需要执行的action。

指定电话号码的intent：

```java
Uri number = Uri.parse("tel:123456");
Intent callIntent = new Intent(Intent.ACTION_DIAL, number);
```

通过执行`startActivity()`启动这个intent时，Phone app会使用之前的电话号码来拨出这个电话。

查看地图：

```java
Uri location = Uri.parse("geo:0,0?q=1600+Amphitheatre+Parkway,+Mountain+View,+California");
Intent mapIntent = new Intent(Intent.ACTION_VIEW, location);
```

查看网页：

```java
Uri webpage = Uri.parse("http://www.android.com");
Intent webIntent = new Intent(Intent.ACTION_VIEW, webpage);
```

发送带附件的email：

```java
Intent emailIntent = new Intent(Intent.ACTION_SEND);
emailIntent.setType(HTTP.PLAIN_TEXT_TYPE);
emailIntent.putExtra(Intent.EXTRA_EMAIL, new String[] {"jon@example.com"}); // recipients
emailIntent.putExtra(Intent.EXTRA_SUBJECT, "Email subject");
emailIntent.putExtra(Intent.EXTRA_TEXT, "Email message text");
emailIntent.putExtra(Intent.EXTRA_STREAM, Uri.parse("content://path/to/email/attachment"));
```

创建一个日历事件：

```java
Intent calendarIntent = new Intent(Intent.ACTION_INSERT, Events.CONTENT_URI);
Calendar beginTime = Calendar.getInstance().set(2012, 0, 19, 7, 30);
Calendar endTime = Calendar.getInstance().set(2012, 0, 19, 10, 30);
calendarIntent.putExtra(CalendarContract.EXTRA_EVENT_BEGIN_TIME, beginTime.getTimeInMillis());
calendarIntent.putExtra(CalendarContract.EXTRA_EVENT_END_TIME, endTime.getTimeInMillis());
calendarIntent.putExtra(Events.TITLE, "Ninja class");
calendarIntent.putExtra(Events.EVENT_LOCATION, "Secret dojo");
```

### 验证是否有App去接收这个Intent {#appintent}

> 尽管Android系统会确保每一个确定的intent会被系统内置的app\(such as the Phone, Email, or Calendar app\)之一接收，但是我们还是应该在触发一个intent之前做验证是否有App接受这个intent的步骤。

为了验证是否有合适的activity会响应这个intent，需要执行`queryIntentActivities()`来获取到能够接收这个intent的所有activity的list。若返回的List非空，那么我们才可以安全的使用这个intent。例如：

```java
PackageManager packageManager = getPackageManager();
List<ResolveInfo> activities = packageManager.queryIntentActivities(intent, 0);
boolean isIntentSafe = activities.size() > 0;
```

如果`isIntentSafe`为`true`, 那么至少有一个app可以响应这个intent。`false`则说明没有app可以handle这个intent。

正确的启动方式：

```java
// 新建intent
Uri location = Uri.parse("geo:0,0?q=1600+Amphitheatre+Parkway,+Mountain+View,+California");
Intent mapIntent = new Intent(Intent.ACTION_VIEW, location);

// 验证是否有app接收
PackageManager packageManager = getPackageManager();
List<ResolveInfo> activities = packageManager.queryIntentActivities(mapIntent, 0);
boolean isIntentSafe = activities.size() > 0;

// 启动
if (isIntentSafe) {
    startActivity(mapIntent);
}
```

### 显示app的选择界面

> 请注意，当以`startActivity()`的形式传递一个intent，并且有多个app可以handle时，用户可以在弹出dialog的时候选择默认启动的app。该功能对于用户有特殊偏好的时候非常有用（例如用户总是喜欢启动某个app来查看网页，总是喜欢启动某个camera来拍照）。

为了显示chooser, 需要使用`createChooser()`来创建Intent：

```java
Intent intent = new Intent(Intent.ACTION_SEND);
// TODO
String title = getResources().getText(R.string.chooser_title); // 指定标题
// 添加以及启动chooser
Intent chooser = Intent.createChooser(intent, title);
startActivity(chooser);
```

## 接受Activity返回的结果

启动另外一个activity并不一定是单向的。我们也可以启动另外一个activity然后接受一个返回的`result`。为接受`result`，我们需要使用`startActivityForResult()` ，而不是`startActivity()`。当然，被启动的activity需要指定返回的`result`。它需要把这个`result`作为另外一个`intent`对象返回，我们的activity需要在`onActivityResult()`的回调方法里面去接收`result`。

### 启动Activity

对于`startActivityForResult()`方法中的intent与之前介绍的并无太大差异，不过是需要在这个方法里面多添加一个int类型的参数。该integer参数称为"**request code**"，用于**标识请求**。当我们接收到result Intent时，可从回调方法里面的参数去判断这个result是否是我们想要的。

启动Activity来选择联系人：

```java
static final int PICK_CONTACT_REQUEST = 1;  // The request code
private void pickContact() {
    Intent pickContactIntent = new Intent(Intent.ACTION_PICK, Uri.parse("content://contacts"));
    pickContactIntent.setType(Phone.CONTENT_TYPE); // Show user only contacts w/ phone numbers
    startActivityForResult(pickContactIntent, PICK_CONTACT_REQUEST);
}
```

### 接收result

当用户完成了启动之后activity操作之后，系统会调用我们activity中的`onActivityResult()` 回调方法。该方法有三个参数：

* 通过`startActivityForResult()`传递的request code。
* 第二个activity指定的result code。如果操作成功则是`RESULT_OK`，如果用户没有操作成功，而是直接点击回退或者其他什么原因，那么则是`RESULT_CANCELED`。
* 包含了所返回result数据的intent。

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode == PICK_CONTACT_REQUEST) {// 判断回调的标识
        if (resultCode == RESULT_OK) {// 判断是否成功
            // TODO
        }
    }
}
```

## Intent过滤

> 但如果我们的app的功能对别的app也有用，那么其应该做好响应的准备。例如，如果创建了一个social app，它可以分享messages 或者 photos 给好友，那么最好我们的app能够接收`ACTION_SEND` 的intent,这样当用户在其他app触发分享功能的时候，我们的app能够出现在待选对话框。
>
> 通过在manifest文件中的`<activity>`标签下添加`<intent-filter>`的属性，使其他的app能够启动我们的activity。当app被安装到设备上时，系统可以识别intent filter并把这些信息记录下来。当其他app使用implicit intent执行`startActivity()`或者 `startActivityForResult()`时，系统会自动查找出那些可以响应该intent的activity。

### 添加Intent Filter

为了尽可能确切的定义activity能够handle的intent，每一个intent filter都应该尽可能详尽的定义好action与data。

若activity中的intent filter满足以下intent对象的标准，系统就能够把特定的intent发送给activity：

* **Action:**一个想要执行的动作的名称。通常是系统已经定义好的值，如`ACTION_SEND`或`ACTION_VIEW`。 在intent filter中通过`<action>`指定它的值，值的类型必须为字符串，而不是API中的常量\(看下面的例子\)
* **Data:**Intent附带数据的描述。在intent filter中通过`<data>`指定它的值，可以使用一个或者多个属性，我们可以只定义`MIME type`或者是只指定`URI prefix`，也可以只定义一个`URI scheme`，或者是他们综合使用。
* **Category:**提供一个附加的方法来标识这个activity能够handle的intent。通常与用户的手势或者是启动位置有关。系统有支持几种不同的categories,但是大多数都很少用到。而且，所有的implicit intents都默认是`CATEGORY_DEFAULT`类型的。在intent filter中用`<category>`指定它的值。

例如，这个有intent filter的activity，当数据类型为文本或图像时会处理ACTION\_SEND的intent：

```xml
<activity android:name="ShareActivity">
    <intent-filter>
        <action android:name="android.intent.action.SEND"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:mimeType="text/plain"/>
        <data android:mimeType="image/*"/>
    </intent-filter>
</activity>
```

每一个发送出来的intent只会包含一个action与data类型，但handle这个intent的activity的 `<intent-filter>`可以声明多个`<action>`,，`<category>`与`<data>`。

如果任何的两对action与data是互相矛盾的，就应该创建不同的intent filter来指定特定的action与type。

### 在Activity中Handle发送过来的Intent {#activityhandleintent}

可以执行`getIntent()`来获取启动我们activity的那个intent。我们可以在activity生命周期的任何时候去执行这个方法，**但最好是在**`onCreate()`**或者**`onStart()`**里面去执行**：

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.main);

    Intent intent = getIntent();
    Uri data = intent.getData();

    if (intent.getType().indexOf("image/") != -1) {
        // 图片数据的处理
    } else if (intent.getType().equals("text/plain")) {
        // 文本数的处理
    }
}
```

### 返回result

参数的作用可以结合接收result的`onActivityResult()`函数去看。

如果想返回一个result给启动的那个activity，仅仅需要执行`setResult()`，通过指定一个resultCode与result intent。操作完成之后，用户需要返回到原来的activity，通过执行`finish()` 关闭被唤起的activity：

```java
Intent result = new Intent("com.example.RESULT_ACTION", Uri.parse("content://result_uri");
setResult(Activity.RESULT_OK, result);
finish();
```

我们必须总是指定一个resultCode。通常不是`RESULT_OK`就是`RESULT_CANCELED`。我们可以通过Intent 来添加需要返回的数据。

> 默认的result code是`RESULT_CANCELED`。因此，如果用户在没有完成操作之前点击了back key，那么之前的activity接受到的result code就是"**canceled**"。

如果只是纯粹想要返回一个int来表示某些返回的result数据之一，则可以设置result code为任何大于0的数值。如果我们返回的result只是一个int，那么连intent都可以不需要返回了，可以调用setResult\(\)然后只传递result code如下：

```java
setResult(RESULT_COLOR_RED);
finish();
```

> 我们没有必要在意自己的activity是被用`startActivity()`还是`startActivityForResult()`方法所叫起的。系统会自动去判断该如何传递result。在不需要的result的case下，result会被自动忽略。



