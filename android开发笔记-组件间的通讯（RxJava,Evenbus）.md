# Evenbus 
> EventBus能够简化各组件间的通信，让我们的代码书写变得简单，能有效的分离事件发送方和接收方(也就是解耦的意思)，能避免复杂和容易出错的依赖性和生命周期问题。

### 1、三要素
- Event  事件。它可以是任意类型。
- Subscriber 事件订阅者。在EventBus3.0之前我们必须定义以onEvent开头的那几个方法，分别是onEvent、onEventMainThread、onEventBackgroundThread和onEventAsync，而在3.0之后事件处理的方法名可以随意取，不过需要加上注解@subscribe()，并且指定线程模型，默认是POSTING。
- Publisher 事件的发布者。我们可以在任意线程里发布事件，一般情况下，使用EventBus.getDefault()就可以得到一个EventBus对象，然后再调用post(Object)方法即可。

### 2、 四种线程模型
- POSTING (默认)  表示事件处理函数的线程跟发布事件的线程在同一个线程。
- MAIN 表示事件处理函数的线程在主线程(UI)线程，因此在这里不能进行耗时操作。
- BACKGROUND 表示事件处理函数的线程在后台线程，因此不能进行UI操作。如果发布事件的线程是主线程(UI线程)，那么事件处理函数将会开启一个后台线程，如果果发布事件的线程是在后台线程，那么事件处理函数就使用该线程。
- ASYNC 表示无论事件发布的线程是哪一个，事件处理函数始终会新建一个子线程运行，同样不能进行UI操作。

### 3、基本用法
3.1 自定义一个事件类当做消息的实体类
```java
public class MessageEvent{
    private String message;
    public  MessageEvent(String message){
        this.message=message;
    }
    public String getMessage() {
        return message;
    }
 
    public void setMessage(String message) {
        this.message = message;
    }
}
```
3.2 注册事件
当我们需要在Activity或者Fragment里 *订阅事件* 时，我们需要注册EventBus。我们一般选择在Activity的onCreate()方法里去注册EventBus，在onDestory()方法里，去解除注册。
```java
@Override
protected void onCreate(Bundle savedInstanceState) {           
     super.onCreate(savedInstanceState);
     setContentView(R.layout.activity_main)；
     EventBus.getDefault().register(this)；
} 
```
3.3 解除注册
```java
@Override
protected void onDestroy() {
    super.onDestroy();
    EventBus.getDefault().unregister(this);
}
```
3.4 发送事件
```java
EventBus.getDefault().post(messageEvent);
```
3.5 处理事件
```java
@Subscribe(threadMode = ThreadMode.MAIN)
public void XXX(MessageEvent messageEvent) {
    ...
}
```
# Rxjava
 > "a library for composing asynchronous and event-based programs using observable sequences for the Java VM"（一个在 Java VM 上使用可观测的序列来组成异步的、基于事件的程序的库）
 
完全听不懂啊，

同样是做异步，为什么人们用它，而不用现成的 AsyncTask / Handler / XXX / ... 
因为简洁啊。
Android 创造的 AsyncTask 和Handler ，其实都是为了让异步代码更加简洁。
RxJava 的优势也是简洁，但它的简洁的与众不同之处在于，随着程序逻辑变得越来越复杂，它依然能够保持简洁。
