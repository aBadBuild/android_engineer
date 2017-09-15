# Activity

> 简单点说：Activity就是布满整个窗口或者悬浮于其他窗口上的交互界面。

## 生命周期

![](/assets/Activity1.png)

三种静态状态，其它状态 \(Created与Started\)都是短暂的瞬态，Activity只能在三种状态之一下存在很长时间：

* **Resumed**：在这种状态下，Activity处于前台，且用户可以与其交互。（有时也称为“运行”状态。）
* **Paused**：在这种状态下，Activity被在前台中处于半透明状态或者未覆盖整个屏幕的另一个Activity—部分阻挡。暂停的Activity不会接收用户输入并且无法执行任何代码。
* **Stopped**：在这种状态下，Activity被完全隐藏并且对用户不可见；它被视为处于后台。停止时，Activity实例及其诸如成员变量等所有状态信息将保留，但它无法执行任何代码。



