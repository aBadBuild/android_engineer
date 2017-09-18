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
* 第二个activity指定的result code。如果操作成功则是RESULT\_OK，如果用户没有操作成功，而是直接点击回退或者其他什么原因，那么则是`RESULT_CANCELED`。
* 包含了所返回result数据的intent。

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    // Check which request we're responding to
    if (requestCode == PICK_CONTACT_REQUEST) {
        // Make sure the request was successful
        if (resultCode == RESULT_OK) {
            
        }
    }
}
```



