# Intent

一个Android app通常都会有多个activities。为了让用户能够从一个activity跳到另一个activity以及数据交互的时候，我们的app必须使用Intent来定义自己的意图。

## Intent的发送

### 建立显性Intent

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

### 建立隐性Intent

> 用于唤醒不同app来执行某个动作。

intents并不声明要启动组件的具体类名，而是声明一个需要执行的action。

指定电话号码的intent：

```java
Uri number = Uri.parse("tel:123456");
Intent callIntent = new Intent(Intent.ACTION_DIAL, number);
```



