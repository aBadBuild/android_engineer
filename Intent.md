# Intent

一个Android app通常都会有多个activities。为了让用户能够从一个activity跳到另一个activity以及数据交互的时候，我们的app必须使用Intent来定义自己的意图。

## Intent的发送

### 建立显性Intent

用于同一个app的两个Acticity之间的切换。

创建一个Intent去启动目标Activity：

```java
Intent intent = new Intent(this, 目标.class);
```



