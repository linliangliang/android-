# 1、四大组件（生命周期，使用场景，如何启动）？
Activity，Service，Broadcast Receiver，Content Provider，外加一个重要组件 intent。

## 一个Activity的启动顺序：onCreate()——>onStart()——>onResume()

- 1、当另一个Activity启动时:第一个Activity onPause()——>第二个Activity onCreate()——>onStart()——>onResume()——>第一个Activity.onStop()


- 2、当返回到第一个Activity时：第二个Activity onPause() ——> 第一个Activity　onRestart()——>onStart()——>onResume() ——>第二个Activity    2、onStop()——>onDestroy()

一个Activity的销毁顺序:
（情况一）onPause()——><Process Killed> 
（情况二）onPause()——>onStop()——><Process Killed> 
（情况三）onPause()——>onStop()——>onDestroy()

> onPause ：当一个正在前台运行的活动因为其他的活动需要前台运行而转入后台运行的时候，触发该方法。这时候需要将活动的状态持久化，比如正在编辑的数据库记录等。onStop ：当一个活动不再需要展示给用户的时候，触发该方法。如果内存紧张，系统会直接结束这个活动，而不会触发 onStop 方法。 所以保存状态信息是应该在onPause时做，而不是onStop时做。因此在一些情况下，onPause方法或许是活动触发的最后的方法，因此开发者需要在这个时候保存需要保存的信息。

### 在android里，有4种activity的启动模式
分别为：
 - standard: 标准模式，一调用startActivity()方法就会产生一个新的实例。
 - singleTop: 如果已经有一个实例位于Activity栈的顶部时，就不产生新的实例，而只是调用Activity中的newInstance()方法。如果不位于栈顶，会产生一个新的实例。
 - singleTask: 会在一个新的task中产生这个实例，以后每次调用都会使用这个，不会去产生新的实例了。
 - singleInstance: 这个跟singleTask基本上是一样，只有一个区别：“singleInstance”独占一个task，其它activity不能存在那个task里；

[这四种模式的区别](https://blog.csdn.net/sinat_14849739/article/details/78072401)

## Server是在一段不定的时间运行在后台，不和用户交互应用组件。
每个Service必须在manifest中 通过<service>来声明。可以通过contect.startservice和contect.bindserverice来启动。Service和其他的应用组件一样，运行在进程的主线程中。这就是说如果service需要很多耗时或者阻塞的操作，需要在其子线程中实现。

### Service的生命周期：

使用context.startService() 启动Service: contex .startService() ->onCreate()- >onStart()->Service running
　　           context.stopService() | ->onDestroy() ->Service stop
如果Service还没有运行，则android先调用onCreate()然后调用onStart()；如果Service已经运行，则只调用onStart()，所以一个Service的onStart方法可能会重复调用多次。
　　stopService的时候直接onDestroy，如果是调用者自己直接退出而没有调用stopService的话，Service会一直在后台运行。该Service的调用者再启动起来后可以通过stopService关闭Service。所以调用startService的生命周期为：onCreate --> onStart(可多次调用) --> onDestroy

使用使用context.bindService()启动Service：
　　context.bindService()->onCreate()->onBind()->Service running
　　onUnbind() -> onDestroy() ->Service stop

onBind将返回给客户端一个IBind接口实例，IBind允许客户端回调服务的方法，比如得到Service运行的状态或其他操作。这个时候把调用者（Context，例如Activity）会和Service绑定在一起，Context退出了，Srevice就会调用onUnbind->onDestroy相应退出。所以调用bindService的生命周期为：onCreate --> onBind(只一次，不可多次绑定) --> onUnbind --> onDestory。在Service每一次的开启关闭过程中，只有onStart可被多次调用(通过多次startService调用)，其他onCreate，onBind，onUnbind，onDestory在一个生命周期中只能被调用一次。

这两个方法都可以启动Service，但是它们的使用场合有所不同。1. 使用startService()方法启用服务，调用者与服务之间没有关连，即使调用者退出了，服务仍然运行。如果打算采用Context.startService()方法启动服务，在服务未被创建时，系统会先调用服务的onCreate()方法，接着调用onStart()方法。如果调用startService()方法前服务已经被创建，多次调用startService()方法并不会导致多次创建服务，但会导致多次调用onStart()方法。

三、Broadcast Receiver

BroadcastReceiver 用于异步接收广播Intent。

       ·正常广播 Normal broadcasts（用 Context.sendBroadcast()发送）是完全异步的。它们都运行在一个未定义的顺序，通常是在同一时间。这样会更有效，但意味着receiver不能包含所要使用的结果或中止的API。
　　·有序广播 Ordered broadcasts（用 Context.sendOrderedBroadcast()发送）每次被发送到一个receiver。所谓有序，就是每个receiver执行后可以传播到下一个receiver，也可以完全中止传播--不传播给其他receiver。 而receiver运行的顺序可以通过matched intent-filter 里面的android:priority来控制，当priority优先级相同的时候，Receiver以任意的顺序运行。

Broadcast Receiver 并没有提供可视化的界面来显示广播信息。可以使用Notification和Notification Manager来实现可视化的信息的界面，显示广播信息的内容，图标及震动信息。

生命周期：一个BroadcastReceiver 对象只有在被调用onReceive(Context, Intent)的才有效的，当从该函数返回后，该对象就无效的了，结束生命周期。

注册Receiver，注册有两种方式：

1、静态方式，在AndroidManifest.xml的application里面定义receiver并设置要接收的action。

<receiver android:name=".SMSReceiver">
　　<intent-filter>
　　<action android:name="android.provider.Telephony.SMS_RECEIVED" />
　　</intent-filter>
</receiver>
2、动态方式, 在activity里面调用函数来注册

 receiver = new CallReceiver(); 
 registerReceiver(receiver, new IntentFilter("android.intent.action.PHONE_STATE")); 
动态注册，需要特别注意的是，在退出程序前要记得调用Context.unregisterReceiver()方法。一般在activity的onStart()里面进行注册, onStop()里面进行注销。官方提醒，如果在Activity.onResume()里面注册了，就必须在Activity.onPause()注销。

四、Content Provider

一个应用实现ContentProvider来提供内容给别的应用来操作，
一个应用通过ContentResolver来操作别的应用数据，当然在自己的应用中也可以。

2、数据持久化 – SQLite，SharedPreferences，ContentProvider
（1）SharedPreferences 一个轻量级的存储类（是用xml文件存放数据，文件存放在/data/data/<package name>/shared_prefs目录下）

A、存放数据信息
//1、打开Preferences，名称为setting，如果存在则打开它，否则创建新的Preferences

SharedPreferences settings = getSharedPreferences(“setting”, 0);

//2、让setting处于编辑状态

SharedPreferences.Editor editor = settings.edit();

//3、存放数据

editor.putString(“name”,”ATAAW”);

//4、完成提交

editor.commit();

B、读取数据信息

//1、获取Preferences

SharedPreferences settings = getSharedPreferences(“setting”, 0);

//2、取出数据

String name = settings.getString(“name”,”默认值”);

(2)SQLite 是一款轻型的数据库

打开或者创建数据库:

db=SQLiteDatabase.openOrCreateDatabase("/data/data/com.lingdududu.db/databases/stu.db",null);

创建一张表:

private void createTable(SQLiteDatabase db){
    //创建表SQL语句
    String stu_table="create table usertable(_id integer primary key autoincrement,sname                                     text,snumber text)";
    //执行SQL语句
    db.execSQL(stu_table);
}

private void insert(SQLiteDatabase db){
    //实例化常量值
    ContentValues cValue = new ContentValues();
    //添加用户名
    cValue.put("sname","xiaoming");
    //添加密码
    cValue.put("snumber","01005");
    //调用insert()方法插入数据
    db.insert("stu_table",null,cValue);
}
//第二种方法的代码：
private void insert(SQLiteDatabase db){
    //插入数据SQL语句
    String stu_sql="insert into stu_table(sname,snumber) values('xiaoming','01005')";
    //执行SQL语句
    db.execSQL(sql);
}

//第一种方法的代码：
private void delete(SQLiteDatabase db) {
    //删除条件
    String whereClause = "id=?";
    //删除条件参数
    String[] whereArgs = {String.valueOf(2)};
    //执行删除
    db.delete("stu_table",whereClause,whereArgs);
}

//第二种方法的代码：
private void delete(SQLiteDatabase db) {  
    //删除SQL语句  
    String sql = "delete from stu_table where _id = 6";  
    //执行SQL语句  
    db.execSQL(sql);  
}  

android 6.0/7.0/8.0特性：

https://blog.csdn.net/u012185875/article/details/78875629

性能优化 – 布局优化，内存优化，电量优化
布局优化：总结起来就是：减少嵌套，避免过度加载。

                 1、如果能使用linearlayout的，尽量不用RelativeLayout

                 2、使用标签，include标签可以使一个我们已经写好的布局加载到当前的布局中，通过include标签，可以使我们的代码变得简洁，不用再多次重写相同的页面。merge标签和include一般一起使用，用来减少布局的嵌套。一般来说，如果外面的布局是一个linearlayout,而被包含的布局也是一个linearlayou,那么我们就可以使用merge标签，去减少多余的那一层LinearLayout嵌套。ViewStub标签使用 

                 3、性能优化工具Hierarchy Viewer查看层级

内存优化：（常见内存泄露及优化方案）如果一个无用对象（不需要再使用的对象）仍然被其他对象持有引用，造成该对象无法被系统回收，以致该对象在堆中所占用的内存单元无法被释放而造成内存空间浪费，这中情况就是内存泄露。

                1、单例导致内存泄露：因为单例的静态特性使得它的生命周期同应用的生命周期一样长，如果一个对象已经没有用处了，但是单例还持有它的引用，那么在整个应用程序的生命周期它都不能正常被回收，从而导致内存泄露。例如：

public class AppSettings {

    private static AppSettings sInstance;
    private Context mContext;

    private AppSettings(Context context) {
        this.mContext = context;
    }

    public static AppSettings getInstance(Context context) {
        if (sInstance == null) {
            sInstance = new AppSettings(context);
        }
        return sInstance;
    }
}
如果我们在调用getInstance(Context context)方法的时候传入的context参数是Activity、Service等上下文，就会导致内存泄露。

以Activity为例，当我们启动一个Activity，并调用getInstance(Context context)方法去获取AppSettings的单例，传入Activity.this作为context，这样AppSettings类的单例sInstance就持有了Activity的引用，当我们退出Activity时，该Activity就没有用了，但是因为sIntance作为静态单例（在应用程序的整个生命周期中存在）会继续持有这个Activity的引用，导致这个Activity对象无法被回收释放，这就造成了内存泄露。

为了避免这样单例导致内存泄露，我们可以将context参数改为全局的上下文：全局的上下文Application Context就是应用程序的上下文，和单例的生命周期一样长，这样就避免了内存泄漏。

private AppSettings(Context context) {
    this.mContext = context.getApplicationContext();
}
                2、静态变量导致内存泄露：

               3、非静态内部类导致内存泄露，非静态内部类（包括匿名内部类）默认就会持有外部类的引用，当非静态内部类对象的生命周期比外部类对象的生命周期长时，就会导致内存泄露。非静态内部类导致的内存泄露在Android开发中有一种典型的场景就是使用Handler。熟悉Handler消息机制的都知道，mHandler会作为成员变量保存在发送的消息msg中，即msg持有mHandler的引用，而mHandler是Activity的非静态内部类实例，即mHandler持有Activity的引用，那么我们就可以理解为msg间接持有Activity的引用。msg被发送后先放到消息队列MessageQueue中，然后等待Looper的轮询处理（MessageQueue和Looper都是与线程相关联的，MessageQueue是Looper引用的成员变量，而Looper是保存在ThreadLocal中的）。那么当Activity退出后，msg可能仍然存在于消息对列MessageQueue中未处理或者正在处理，那么这样就会导致Activity无法被回收，以致发生Activity的内存泄露。

               4、非静态内部类造成内存泄露还有一种情况就是使用Thread或者AsyncTask。比如在Activity中直接new一个子线程Thread，或者直接新建AsyncTask异步任务：

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        new Thread(new Runnable() {
            @Override
            public void run() {
                // 模拟相应耗时逻辑
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
}
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        new AsyncTask<Void, Void, Void>() {
            @Override
            protected Void doInBackground(Void... params) {
                // 模拟相应耗时逻辑
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                return null;
            }
        }.execute();
    }
}
很多初学者都会像上面这样新建线程和异步任务，殊不知这样的写法非常地不友好，这种方式新建的子线程Thread和AsyncTask都是匿名内部类对象，默认就隐式的持有外部Activity的引用，导致Activity内存泄露。要避免内存泄露的话还是需要像上面Handler一样使用静态内部类+弱应用的方式（代码就不列了，参考上面Hanlder的正确写法）。

               5、未取消注册或回调导致内存泄露，比如我们在Activity中注册广播，如果在Activity销毁后不取消注册，那么这个刚播会一直存在系统中，同上面所说的非静态内部类一样持有Activity引用，导致内存泄露。因此注册广播后在Activity销毁后一定要取消注册。在注册观察则模式的时候，如果不及时取消也会造成内存泄露。比如使用Retrofit+RxJava注册网络请求的观察者回调，同样作为匿名内部类持有外部引用，所以需要记得在不用或者销毁的时候取消注册。

               6、Timer和TimerTask导致内存泄露，Timer和TimerTask在Android中通常会被用来做一些计时或循环任务，比如实现无限轮播的ViewPager：

public class MainActivity extends AppCompatActivity {

    private ViewPager mViewPager;
    private PagerAdapter mAdapter;
    private Timer mTimer;
    private TimerTask mTimerTask;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        init();
        mTimer.schedule(mTimerTask, 3000, 3000);
    }

    private void init() {
        mViewPager = (ViewPager) findViewById(R.id.view_pager);
        mAdapter = new ViewPagerAdapter();
        mViewPager.setAdapter(mAdapter);

        mTimer = new Timer();
        mTimerTask = new TimerTask() {
            @Override
            public void run() {
                MainActivity.this.runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        loopViewpager();
                    }
                });
            }
        };
    }

    private void loopViewpager() {
        if (mAdapter.getCount() > 0) {
            int curPos = mViewPager.getCurrentItem();
            curPos = (++curPos) % mAdapter.getCount();
            mViewPager.setCurrentItem(curPos);
        }
    }

    private void stopLoopViewPager() {
        if (mTimer != null) {
            mTimer.cancel();
            mTimer.purge();
            mTimer = null;
        }
        if (mTimerTask != null) {
            mTimerTask.cancel();
            mTimerTask = null;
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        stopLoopViewPager();
    }
}
当我们Activity销毁的时，有可能Timer还在继续等待执行TimerTask，它持有Activity的引用不能被回收，因此当我们Activity销毁的时候要立即cancel掉Timer和TimerTask，以避免发生内存泄漏。

              7、集合中的对象未清理造成内存泄露，这个比较好理解，如果一个对象放入到ArrayList、HashMap等集合中，这个集合就会持有该对象的引用。当我们不再需要这个对象时，也并没有将它从集合中移除，这样只要集合还在使用（而此对象已经无用了），这个对象就造成了内存泄露。并且如果集合被静态引用的话，集合里面那些没有用的对象更会造成内存泄露了。所以在使用集合时要及时将不用的对象从集合remove，或者clear集合，以避免内存泄漏。

              8、资源未关闭或释放导致内存泄露，在使用IO、File流或者Sqlite、Cursor等资源时要及时关闭。这些资源在进行读写操作时通常都使用了缓冲，如果及时不关闭，这些缓冲对象就会一直被占用而得不到释放，以致发生内存泄露。因此我们在不需要使用它们的时候就及时关闭，以便缓冲能及时得到释放，从而避免内存泄露。

             9、属性动画造成内存泄露，

动画同样是一个耗时任务，比如在Activity中启动了属性动画（ObjectAnimator），但是在销毁的时候，没有调用cancle方法，虽然我们看不到动画了，但是这个动画依然会不断地播放下去，动画引用所在的控件，所在的控件引用Activity，这就造成Activity无法正常释放。因此同样要在Activity销毁的时候cancel掉属性动画，避免发生内存泄漏。



             10、WebView造成内存泄露，关于WebView的内存泄露，因为WebView在加载网页后会长期占用内存而不能被释放，因此我们在Activity销毁后要调用它的destory()方法来销毁它以释放内存。

另外在查阅WebView内存泄露相关资料时看到这种情况：

Webview下面的Callback持有Activity引用，造成Webview内存无法释放，即使是调用了Webview.destory()等方法都无法解决问题（Android5.1之后）。

最终的解决方案是：在销毁WebView之前需要先将WebView从父容器中移除，然后在销毁WebView。详细分析过程请参考这篇文章：WebView内存泄漏解决方法。https://blog.csdn.net/xygy8860/article/details/53334476

@Override
protected void onDestroy() {
    super.onDestroy();
    // 先从父控件中移除WebView
    mWebViewContainer.removeView(mWebView);
    mWebView.stopLoading();
    mWebView.getSettings().setJavaScriptEnabled(false);
    mWebView.clearHistory();
    mWebView.removeAllViews();
    mWebView.destroy();
}
安全 – 数据加密，代码混淆，WebView/Js调用，https
数据加密 推荐用AES+RSA进行加密（或者AES+HTTPS）
https://www.jianshu.com/p/4f4a927339a9

按可逆性：加密可分为可逆算法和不可逆算法

按对称性：加密可分为对称算法和非对称算法

0、MD5加密，

1、AES加密

2、RSA非对称加密

是先由服务器创建RSA密钥对，RSA公钥保存在安卓的so文件里面，服务器保存RSA私钥。而安卓创建AES密钥（这个密钥也是在so文件里面），并用该AES密钥加密待传送的明文数据，同时用接受的RSA公钥加密AES密钥，最后把用RSA公钥加密后的AES密钥同密文一起通过Internet传输发送到服务器。当服务器收到这个被加密的AES密钥和密文后，首先调用服务器保存的RSA私钥，并用该私钥解密加密的AES密钥，得到AES密钥。最后用该AES密钥解密密文得到明文。

so文件：ndk开发的so，可以存放一些重要的数据，如：密钥、私钥、API_KEy等，不过这里我建议大家使用分段存放，C层（so文件）+String文件（string.xml）+gradle文件，用的时候再拼接合并，还有如上图所示，AES的加密算法是放在C层进行实现的，这样也是最大程度保护我们数据的安全

代码混淆
http://www.jianshu.com/p/f3455ecaa56e

HTTPS：
现在很多APP都用HTTPS作为网络传输的保证，防止中间人攻击，提高数据传输的安全性（用Retrofit的网络请求框架的，要加上HTTPS也不是什么难事，推荐 http://www.jianshu.com/p/16994e49e2f6 ，这里说的HTTPS是指自签的）

WebView/Js调用
https://blog.csdn.net/carson_ho/article/details/64904691/

高效 保活长连接：手把手教你实现 自适应的心跳保活机制
https://blog.csdn.net/carson_ho/article/details/79522975


其他 – JNI（java原色接口，可以与c语言交互），AIDL（提供一些机制在不同进程之间进行数据通信：https://www.cnblogs.com/BeyondAnyTime/p/3204119.html），Handler，Intent等


开源框架
Gilde：https://blog.csdn.net/qq_31679853/article/details/78616289
Glide.with(context)
    .load("http://inthecheesefactory.com/uploads/source/glidepicasso/cover.jpg")
    .into(ivImg);
下面我们就来详细解析一下这行代码。

首先，调用Glide.with()方法用于创建一个加载图片的实例。with()方法可以接收Context、Activity或者Fragment类型的参数。也就是说我们选择的范围非常广，不管是在Activity还是Fragment中调用with()方法，都可以直接传this。那如果调用的地方既不在Activity中也不在Fragment中呢？也没关系，我们可以获取当前应用程序的ApplicationContext，传入到with()方法当中。注意with()方法中传入的实例会决定Glide加载图片的生命周期，如果传入的是Activity或者Fragment的实例，那么当这个Activity或Fragment被销毁的时候，图片加载也会停止。如果传入的是ApplicationContext，那么只有当应用程序被杀掉的时候，图片加载才会停止。Glide支持加载各种各样的图片资源，包括网络图片、本地图片、应用资源、二进制流、Uri对象等等。

观察刚才加载网络图片的效果，你会发现，点击了Load Image按钮之后，要稍微等一会图片才会显示出来。这其实很容易理解，因为从网络上下载图片本来就是需要时间的。那么我们有没有办法再优化一下用户体验呢？当然可以，Glide提供了各种各样非常丰富的API支持，其中就包括了占位图功能。顾名思义，占位图就是指在图片的加载过程中，我们先显示一张临时的图片，等图片加载出来了再替换成要加载的图片。

Glide.with(this)
     .load(url)
     .placeholder(R.drawable.loading)
     .into(imageView)

不过如果你现在重新运行一下代码并点击Load Image，很可能是根本看不到占位图效果的。因为Glide有非常强大的缓存机制，我们刚才加载那张必应美图的时候Glide自动就已经将它缓存下来了，下次加载的时候将会直接从缓存中读取，不会再去网络下载了，因而加载的速度非常快，所以占位图可能根本来不及显示。可以禁用掉Glide的缓存功能。

当然，这只是占位图的一种，除了这种加载占位图之外，还有一种异常占位图。异常占位图就是指，如果因为某些异常情况导致图片加载失败，比如说手机网络信号不好，这个时候就显示这张异常占位图。

Glide.with(this)
     .load(url)
     .placeholder(R.drawable.loading)
     .error(R.drawable.error)
     .diskCacheStrategy(DiskCacheStrategy.NONE)
     .into(imageView);


Picasso:
Picasso.with(context)
    .load("http://inthecheesefactory.com/uploads/source/glidepicasso/cover.jpg")
    .into(ivImg);
Gilde和Picasso的比较 http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0327/2650.html




通信 – 网络连接（HttpClient，HttpUrlConnetion），Socket
(https://blog.csdn.net/itachi85/article/details/50982995)
Android与服务器的通信方式主要有两种，一是Http通信，一是Socket通信。两者的最大差异在于，http连接使用的是“请求—响应方式”，即在请求时建立连接通道，当客户端向服务器发送请求后，服务器端才能向客户端返回数据。而Socket通信则是在双方建立起连接后就可以直接进行数据的传输，在连接时可实现信息的主动推送，而不需要每次由客户端想服务器发送请求。 那么，什么是socket？Socket又称套接字，在程序内部提供了与外界通信的端口，即端口通信。通过建立socket连接，可为通信双方的数据传输传提供通道。socket的主要特点有数据丢失率低，使用简单且易于移植。



Volley： https://blog.csdn.net/u012602304/article/details/79170137

Volley是2013年谷歌官方发布的一款Android平台上的网络通信库。Volley非常适合一些数据量不大，但需要频繁通信的网络操作。使用Volley进行网络开发可以使我们的开发效率得到很大的提升，而且性能的稳定性也比较高。但是Volley不适用于文件的上传下载操作。

Volley的使用：使用Volley需要建立一个全局的请求队列，这样我们就可以将一个请求加入到这个全局队列中，并可以管理整个APP的所有请求，包括取消一个或所有的请求。RequestQueue内部的设计就是非常合适高并发的，因此我们不必为每一次HTTP请求都创建一个RequestQueue对象，这是非常浪费资源的。

Volley的Get和Post请求方式的使用：

Volley自带了三种返回类型：

StringRequest：主要使用在对请求数据的返回类型不确定的情况下，StringRequest涵盖了JsonObjectRequest和JsonArrayRequest。

JsonObjectRequest：当确定请求数据的返回类型为JsonObject时使用。

JsonArrayRequest：当确定请求数据的返回类型为JsonArray时使用。

下面通过代码进行实例的演示。

使用Volley前需要往项目中导入Volley的jar包。

首先我们需要自定义一个Application用于创建一个全局的请求队列。MyApplication.java

public class MyApplication extends Application{
    private static RequestQueue queues ;
    @Override
    public void onCreate() {
        super.onCreate();
        queues = Volley.newRequestQueue(getApplicationContext());
    }

    public static RequestQueue getHttpQueues() {
        return queues;
    }
}
manifest文件添加 android:name=".MyApplication"
1、Get方式请求数据返回StringRequest对象

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        volleyGet();
    }

    /**
     *  new StringRequest(int method,String url,Listener listener,ErrorListener errorListener)
     *  method：请求方式，Get请求为Method.GET，Post请求为Method.POST
     *  url：请求地址
     *  listener：请求成功后的回调
     *  errorListener：请求失败的回调
     */
    private void volleyGet() {
        String url = "https://tcc.taobao.com/cc/json/mobile_tel_segment.htm?tel=15850781443";
        StringRequest request = new StringRequest(Method.GET, url,
                new Listener<String>() {
                    @Override
                    public void onResponse(String s) {//s为请求返回的字符串数据
                        Toast.makeText(MainActivity.this,s,Toast.LENGTH_LONG).show();
                    }
                },
                new ErrorListener() {
                    @Override
                    public void onErrorResponse(VolleyError volleyError) {
                        Toast.makeText(MainActivity.this,volleyError.toString(),Toast.LENGTH_LONG).show();
                    }
                });
        //设置请求的Tag标签，可以在全局请求队列中通过Tag标签进行请求的查找
        request.setTag("testGet");
        //将请求加入全局队列中
        MyApplication.getHttpQueues().add(request);
    }

}
2、Get方式请求数据返回JsonObjectRequest对象

/**
 *  new JsonObjectRequest(int method,String url,JsonObject jsonObject,Listener listener,ErrorListener errorListener)
 *  method：请求方式，Get请求为Method.GET，Post请求为Method.POST
 *  url：请求地址
 *  JsonObject：Json格式的请求参数。如果使用的是Get请求方式，请求参数已经包含在url中，所以可以将此参数置为null
 *  listener：请求成功后的回调
 *  errorListener：请求失败的回调
 */
private void volleyGet() {
    String url = "http://int.dpool.sina.com.cn/iplookup/iplookup.php?format=json&ip=218.4.255.255";
    JsonObjectRequest request = new JsonObjectRequest(Method.GET, url, null,
            new Listener<JSONObject>() {
                @Override
                public void onResponse(JSONObject jsonObject) {//jsonObject为请求返回的Json格式数据
                    Toast.makeText(MainActivity.this,jsonObject.toString(),Toast.LENGTH_LONG).show();
                }
            },
            new ErrorListener() {
                @Override
                public void onErrorResponse(VolleyError volleyError) {
                    Toast.makeText(MainActivity.this,volleyError.toString(),Toast.LENGTH_LONG).show();
                }
            });

    //设置请求的Tag标签，可以在全局请求队列中通过Tag标签进行请求的查找
    request.setTag("testGet");
    //将请求加入全局队列中
    MyApplication.getHttpQueues().add(request);
}
3、Post方式请求数据返回StringRequest对象

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        volleyPost();
    }

    /**
     * 使用Post方式返回String类型的请求结果数据
     * 
     *  new StringRequest(int method,String url,Listener listener,ErrorListener errorListener)
     *  method：请求方式，Get请求为Method.GET，Post请求为Method.POST
     *  url：请求地址
     *  listener：请求成功后的回调
     *  errorListener：请求失败的回调
     */
    private void volleyPost() {
        String url = "https://tcc.taobao.com/cc/json/mobile_tel_segment.htm";
        StringRequest request = new StringRequest(Method.POST, url,
                new Listener<String>() {
                    @Override
                    public void onResponse(String s) {//s为请求返回的字符串数据
                        Toast.makeText(MainActivity.this,s,Toast.LENGTH_LONG).show();
                    }
                },
                new ErrorListener() {
                    @Override
                    public void onErrorResponse(VolleyError volleyError) {
                        Toast.makeText(MainActivity.this,volleyError.toString(),Toast.LENGTH_LONG).show();
                    }
                }){
                    @Override
                    protected Map<String, String> getParams() throws AuthFailureError {
                        Map<String,String> map = new HashMap<>();
                        //将请求参数名与参数值放入map中
                        map.put("tel","15850781443");
                        return map;
                    }
                }
                ;
        //设置请求的Tag标签，可以在全局请求队列中通过Tag标签进行请求的查找
        request.setTag("testPost");
        //将请求加入全局队列中
        MyApplication.getHttpQueues().add(request);
    }
}
4、使用Post方式请求数据返回JsonObject对象

/**
 *  使用Post方式返回JsonObject类型的请求结果数据
 *
 *  new JsonObjectRequest(int method,String url,JsonObject jsonObject,Listener listener,ErrorListener errorListener)
 *  method：请求方式，Get请求为Method.GET，Post请求为Method.POST
 *  url：请求地址
 *  JsonObject：Json格式的请求参数。如果使用的是Get请求方式，请求参数已经包含在url中，所以可以将此参数置为null
 *  listener：请求成功后的回调
 *  errorListener：请求失败的回调
 */
private void volleyPost() {
    String url = "http://www.kuaidi100.com/query";
    Map<String,String> map = new HashMap<>();
    map.put("type","yuantong");
    map.put("postid","229728279823");
    //将map转化为JSONObject对象
    JSONObject jsonObject = new JSONObject(map);

    JsonObjectRequest request = new JsonObjectRequest(Method.POST, url, jsonObject,
            new Listener<JSONObject>() {
                @Override
                public void onResponse(JSONObject jsonObject) {//jsonObject为请求返回的Json格式数据
                    Toast.makeText(MainActivity.this,jsonObject.toString(),Toast.LENGTH_LONG).show();
                }
            },
            new ErrorListener() {
                @Override
                public void onErrorResponse(VolleyError volleyError) {
                    Toast.makeText(MainActivity.this,volleyError.toString(),Toast.LENGTH_LONG).show();
                }
            });
    //设置请求的Tag标签，可以在全局请求队列中通过Tag标签进行请求的查找
    request.setTag("testPost");
    //将请求加入全局队列中
    MyApplication.getHttpQueues().add(request);
}
5、与Activity生命周期联动

可以在Activity关闭时取消请求队列中的请求。

@Override
protected void onStop() {
    super.onStop();
    //通过Tag标签取消请求队列中对应的全部请求
    MyApplication.getHttpQueues().cancelAll("abcGet");
    MyApplication.getHttpQueues().cancelAll("loadImage");
}
6、使用ImageRequest加载网络图片

public class MainActivity extends AppCompatActivity {
    private ImageView image;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        image = (ImageView) findViewById(R.id.image);
        loadImageByVolley();
    }

    /**
     *  通过Volley加载网络图片
     *
     *  new ImageRequest(String url,Listener listener,int maxWidth,int maxHeight,Config decodeConfig,ErrorListener errorListener)
     *  url：请求地址
     *  listener：请求成功后的回调
     *  maxWidth、maxHeight：设置图片的最大宽高，如果均设为0则表示按原尺寸显示
     *  decodeConfig：图片像素的储存方式。Config.RGB_565表示每个像素占2个字节，Config.ARGB_8888表示每个像素占4个字节等。
     *  errorListener：请求失败的回调
     */
    private void loadImageByVolley() {
        String url = "http://pic20.nipic.com/20120409/9188247_091601398179_2.jpg";
        ImageRequest request = new ImageRequest(
                            url,
                            new Listener<Bitmap>() {
                                @Override
                                public void onResponse(Bitmap bitmap) {
                                    image.setImageBitmap(bitmap);
                                }
                            },
                            0, 0, Config.RGB_565,
                            new ErrorListener() {
                                @Override
                                public void onErrorResponse(VolleyError volleyError) {
                                    image.setImageResource(R.mipmap.ic_launcher);
                                }
                            });

        //设置请求的Tag标签，可以在全局请求队列中通过Tag标签进行请求的查找
        request.setTag("loadImage");
        //通过Tag标签取消请求队列中对应的全部请求
        MyApplication.getHttpQueues().add(request);
    }

}
7、使用ImageLoader加载及缓存网络图片

public class MainActivity extends AppCompatActivity {
    private ImageView image;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        image = (ImageView) findViewById(R.id.image);
        loadImageWithCache();
    }
/**
     *  通过ImageLoader加载及缓存网络图片
　　　*
     *  new ImageLoader(RequestQueue queue,ImageCache imageCache)
     *  queue：请求队列
     *  imageCache：一个用于图片缓存的接口，一般需要传入它的实现类
     *
     *  getImageListener(ImageView view, int defaultImageResId, int errorImageResId)
     *  view：ImageView对象
     *  defaultImageResId：默认的图片的资源Id
     *  errorImageResId：网络图片加载失败时显示的图片的资源Id
     */
    private void loadImageWithCache() {
        String url = "http://pic20.nipic.com/20120409/9188247_091601398179_2.jpg";
        ImageLoader loader = new ImageLoader(MyApplication.getHttpQueues(), new BitmapCache());
        ImageListener listener = loader.getImageListener(image,R.mipmap.ic_launcher,R.mipmap.ic_launcher);
        //加载及缓存网络图片
        loader.get(url,listener);
    }
}

public class BitmapCache implements ImageLoader.ImageCache{
    //LruCache是基于内存的缓存类
    private LruCache<String,Bitmap> lruCache;
    //LruCache的最大缓存大小
    private int max = 10 * 1024 * 1024;

    public BitmapCache() {
        lruCache = new LruCache<String, Bitmap>(max){
            @Override
            //缓存图片的大小
            protected int sizeOf(String key, Bitmap value) {
                return value.getRowBytes() * value.getHeight();
            }
        };
    }

    @Override
    public Bitmap getBitmap(String s) {
        return lruCache.get(s);
    }

    @Override
    public void putBitmap(String s, Bitmap bitmap) {
        lruCache.put(s,bitmap);
    }
}


1.HttpClient
Android SDK中包含了HttpClient，在Android6.0版本直接删除了HttpClient类库，如果仍想使用则解决方法是：

如果使用的是android studio则 在相应的module下的build.gradle中加入：

android {
    useLibrary 'org.apache.http.legacy'
}
2.OKHttp(okhttp请求网络切换回来是在线程里面的，不是在主线程，不能直接刷新UI，需要我们手动处理。)
1、get请求的使用方法：一种是同步请求，一种是异步请求。下面分情况进行介绍。

1.1 get的同步请求:(同步请求在请求时需要开启子线程，请求成功后需要跳转到UI线程修改UI。)

public void getDatasync(){
    new Thread(new Runnable() {
        @Override
        public void run() {
            try {
                OkHttpClient client = new OkHttpClient();//创建OkHttpClient对象
                Request request = new Request.Builder()
                        .url("http://www.baidu.com")//请求接口。如果需要传参拼接到接口后面。
                        .build();//创建Request 对象
                Response response = null;
                response = client.newCall(request).execute();//得到Response 对象
                if (response.isSuccessful()) {
                Log.d("kwwl","response.code()=="+response.code());
                Log.d("kwwl","response.message()=="+response.message());
                Log.d("kwwl","res=="+response.body().string());
                //此时的代码执行在子线程，修改UI的操作请使用handler跳转到UI线程。
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }).start();
}
1.2get的异步请求 (这种方式不用再次开启子线程，但回调方法是执行在子线程中，所以在更新UI时还要跳转到UI线程中。)

private void getDataAsync() {
    OkHttpClient client = new OkHttpClient();
    Request request = new Request.Builder()
            .url("http://www.baidu.com")
            .build();
    client.newCall(request).enqueue(new Callback() {
        @Override
        public void onFailure(Call call, IOException e) {
        }
        @Override
        public void onResponse(Call call, Response response) throws IOException {
            if(response.isSuccessful()){//回调的方法执行在子线程。
                Log.d("kwwl","获取数据成功了");
                Log.d("kwwl","response.code()=="+response.code());
                Log.d("kwwl","response.body().string()=="+response.body().string());
            }
        }
    });
}
注意事项：
回调接口的onFailure方法和onResponse执行在子线程。
response.body().string()方法也必须放在子线程中。当执行这行代码得到结果后，再跳转到UI线程修改UI。

2、post请求的使用方法

Post请求也分同步和异步两种方式，同步与异步的区别和get方法类似，所以此时只讲解post异步请求的使用方法。
使用示例如下：

private void postDataWithParame() {
    OkHttpClient client = new OkHttpClient();//创建OkHttpClient对象。
    FormBody.Builder formBody = new FormBody.Builder();//创建表单请求体
    formBody.add("username","zhangsan");//传递键值对参数
    Request request = new Request.Builder()//创建Request 对象。
            .url("http://www.baidu.com")
            .post(formBody.build())//传递请求体
            .build();
    client.newCall(request).enqueue(new Callback() {。。。});//回调方法的使用与get异步请求相同，此时略。
}
2.1 使用FormBody传递键值对参数

上传String类型的键值对

private void postDataWithParame() {
    OkHttpClient client = new OkHttpClient();//创建OkHttpClient对象。
    FormBody.Builder formBody = new FormBody.Builder();//创建表单请求体
    formBody.add("username","zhangsan");//传递键值对参数
    Request request = new Request.Builder()//创建Request 对象。
            .url("http://www.baidu.com")
            .post(formBody.build())//传递请求体
            .build();
    client.newCall(request).enqueue(new Callback() {。。。});//此处省略回调方法。
}
上传json对象

OkHttpClient client = new OkHttpClient();//创建OkHttpClient对象。
MediaType JSON = MediaType.parse("application/json; charset=utf-8");//数据类型为json格式，
String jsonStr = "{\"username\":\"lisi\",\"nickname\":\"李四\"}";//json数据.
RequestBody body = RequestBody.create(JSON, josnStr);
Request request = new Request.Builder()
        .url("http://www.baidu.com")
        .post(body)
        .build();
client.newCall(request).enqueue(new Callback() {。。。});//此处省略回调方法。
上传File对象

OkHttpClient client = new OkHttpClient();//创建OkHttpClient对象。
MediaType fileType = MediaType.parse("File/*");//数据类型为json格式，
File file = new File("path");//file对象.
RequestBody body = RequestBody.create(fileType , file );
Request request = new Request.Builder()
        .url("http://www.baidu.com")
        .post(body)
        .build();
client.newCall(request).enqueue(new Callback() {。。。});//此处省略回调方法。
使用MultipartBody同时传递键值对参数和File对象

我们知道FromBody传递的是字符串型的键值对，RequestBody传递的是多媒体，那么如果我们想二者都传递怎么办？此时就需要使用MultipartBody类。

OkHttpClient client = new OkHttpClient();
MultipartBody multipartBody =new MultipartBody.Builder()
        .setType(MultipartBody.FORM)
        .addFormDataPart("groupId",""+groupId)//添加键值对参数
        .addFormDataPart("title","title")
        .addFormDataPart("file",file.getName(),RequestBody.create(MediaType.parse("file/*"), file))//添加文件
        .build();
final Request request = new Request.Builder()
        .url(URLContant.CHAT_ROOM_SUBJECT_IMAGE)
        .post(multipartBody)
        .build();
client.newCall(request).enqueue(new Callback() {。。。});
使用RequestBody上传文件时，并没有实现断点续传的功能。我可以使用这种方法结合RandomAccessFile类实现断点续传的功能。

下载文件
https://blog.csdn.net/easkshark/article/details/79180344

对于OKHttp的使用封装






Socket
1 Java中的Socket编程接口介绍
1.1 Socket类

Socket类实现了一个客户端socket，作为两台机器通信的终端，默认采用的传输层协议为TCP，是一个可靠传输的协议。Socket类除了构造函数返回一个socket外，还提供了connect, getOutputStream, getInputStream和close方法。connect方法用于请求一个socket连接，getOutputStream用于获得写socket的输出流，getInputStream用于获得读socket的输入流，close方法用于关闭一个流。

1.2 DatagramSocket类

DatagramSocket类实现了一个发送和接收数据报的socket，传输层协议使用UDP，不能保证数据报的可靠传输。DataGramSocket主要有send, receive和close三个方法。send用于发送一个数据报，Java提供了DatagramPacket对象用来表达一个数据报。receive用于接收一个数据报，调用该方法后，一直阻塞接收到直到数据报或者超时。close是关闭一个socket。

1.3 ServerSocket类

ServerSocket类实现了一个服务器socket，一个服务器socket等待客户端网络请求，然后基于这些请求执行操作，并返回给请求者一个结果。ServerSocket提供了bind、accept和close三个方法。bind方法为ServerSocket绑定一个IP地址和端口，并开始监听该端口。accept方法为ServerSocket接受请求并返回一个Socket对象，accept方法调用后，将一直阻塞直到有请求到达。close方法关闭一个ServerSocket对象。

1.4 SocketAddress

SocketAddress提供了一个socket地址，不关心传输层协议。这是一个虚类，由子类来具体实现功能、绑定传输协议。它提供了一个不可变的对象，被socket用来绑定、连接或者返回数值。

1.5 InetSocketAddress

InetSocketAddress实现了IP地址的SocketAddress，也就是有IP地址和端口号表达Socket地址。如果不制定具体的IP地址和端口号，那么IP地址默认为本机地址，端口号随机选择一个。

1.6. DatagramPacket

DatagramSocket是面向数据报socket通信的一个可选通道。数据报通道不是对网络数据报socket通信的完全抽象。socket通信的控制由DatagramSocket对象实现。DatagramPacket需要与DatagramSocket配合使用才能完成基于数据报的socket通信。



Socket的实现是多样化的。本指南中只介绍TCP/IP协议族的内容，在这个协议族当中主要的Socket类型为流套接字（streamsocket）和数据报套接字(datagramsocket)。流套接字将TCP作为其端对端协议，提供了一个可信赖的字节流服务。数据报套接字使用UDP协议，提供数据打包发送服务。

1、Socket基本实现原理（https://www.cnblogs.com/zhujiabin/p/5675716.html）(https://www.jianshu.com/p/fb4dfab4eec1)

1.1基于TCP协议的Socket 
服务器端首先声明一个ServerSocket对象并且指定端口号，然后调用Serversocket的accept（）方法接收客户端的数据。accept（）方法在没有数据进行接收的处于堵塞状态。（Socketsocket=serversocket.accept()）,一旦接收到数据，通过inputstream读取接收的数据。
  客户端创建一个Socket对象，指定服务器端的ip地址和端口号（Socketsocket=newSocket("172.168.10.108",8080);）,通过inputstream读取数据，获取服务器发出的数据（OutputStreamoutputstream=socket.getOutputStream()），最后将要发送的数据写入到outputstream即可进行TCP协议的socket数据传输。

1.2基于UDP协议的数据传输 
服务器端首先创建一个DatagramSocket对象，并且指点监听的端口。接下来创建一个空的DatagramSocket对象用于接收数据（bytedata[]=newbyte[1024;]DatagramSocketpacket=newDatagramSocket（data，data.length））,使用DatagramSocket的receive方法接收客户端发送的数据，receive（）与serversocket的accepet（）类似，在没有数据进行接收的处于堵塞状态。
客户端也创建个DatagramSocket对象，并且指点监听的端口。接下来创建一个InetAddress对象，这个对象类似与一个网络的发送地址（InetAddressserveraddress=InetAddress.getByName（"172.168.1.120"））.定义要发送的一个字符串，创建一个DatagramPacket对象，并制定要讲这个数据报包发送到网络的那个地址以及端口号，最后使用DatagramSocket的对象的send（）发送数据。*（Stringstr="hello";bytedata[]=str.getByte();DatagramPacketpacket=new DatagramPacket(data,data.length,serveraddress,4567);socket.send(packet);）

例子：

1、添加权限

<!--允许应用程序改变网络状态-->    
<uses-permission android:name="android.permission.CHANGE_NETWORK_STATE"/>    
    
<!--允许应用程序改变WIFI连接状态-->    
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE"/>    
    
<!--允许应用程序访问有关的网络信息-->    
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>    
    
<!--允许应用程序访问WIFI网卡的网络信息-->    
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>    
    
<!--允许应用程序完全使用网络-->    
<uses-permission android:name="android.permission.INTERNET"/>
2、客户端创建socket

protected void connectServerWithTCPSocket() {  
  
        Socket socket;  
        try {// 创建一个Socket对象，并指定服务端的IP及端口号  
            socket = new Socket("192.168.1.32", 1989);  
            // 创建一个InputStream用户读取要发送的文件。  
            InputStream inputStream = new FileInputStream("e://a.txt");  
            // 获取Socket的OutputStream对象用于发送数据。  
            OutputStream outputStream = socket.getOutputStream();  
            // 创建一个byte类型的buffer字节数组，用于存放读取的本地文件  
            byte buffer[] = new byte[4 * 1024];  
            int temp = 0;  
            // 循环读取文件  
            while ((temp = inputStream.read(buffer)) != -1) {  
                // 把数据写入到OuputStream对象中  
                outputStream.write(buffer, 0, temp);  
            }  
            // 发送读取的数据到服务端  
            outputStream.flush();  
  
            /** 或创建一个报文，使用BufferedWriter写入,看你的需求 **/  
//          String socketData = "[2143213;21343fjks;213]";  
//          BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(  
//                  socket.getOutputStream()));  
//          writer.write(socketData.replace("\n", " ") + "\n");  
//          writer.flush();  
            /************************************************/  
        } catch (UnknownHostException e) {  
            e.printStackTrace();  
        } catch (IOException e) {  
            e.printStackTrace();  
        }  
  
    }
3、服务器端简单实现：

public void ServerReceviedByTcp() {  
    // 声明一个ServerSocket对象  
    ServerSocket serverSocket = null;  
    try {  
        // 创建一个ServerSocket对象，并让这个Socket在1989端口监听  
        serverSocket = new ServerSocket(1989);  
        // 调用ServerSocket的accept()方法，接受客户端所发送的请求，  
        // 如果客户端没有发送数据，那么该线程就停滞不继续  
        Socket socket = serverSocket.accept();  
        // 从Socket当中得到InputStream对象  
        InputStream inputStream = socket.getInputStream();  
        byte buffer[] = new byte[1024 * 4];  
        int temp = 0;  
        // 从InputStream当中读取客户端所发送的数据  
        while ((temp = inputStream.read(buffer)) != -1) {  
            System.out.println(new String(buffer, 0, temp));  
        }  
        serverSocket.close();  
    } catch (IOException e) {  
        e.printStackTrace();  
    }  
}
1、socket发送数据报，客户端：

protected void connectServerWithUDPSocket() {  
    DatagramSocket socket;  
    try {  
        //创建DatagramSocket对象并指定一个端口号，注意，如果客户端需要接收服务器的返回数据,  
        //还需要使用这个端口号来receive，所以一定要记住  
        socket = new DatagramSocket(1985);  
        //使用InetAddress(Inet4Address).getByName把IP地址转换为网络地址    
        InetAddress serverAddress = InetAddress.getByName("192.168.1.32");  
        //Inet4Address serverAddress = (Inet4Address) Inet4Address.getByName("192.168.1.32");    
        String str = "[2143213;21343fjks;213]";//设置要发送的报文    
        byte data[] = str.getBytes();//把字符串str字符串转换为字节数组    
        //创建一个DatagramPacket对象，用于发送数据。    
        //参数一：要发送的数据  参数二：数据的长度  参数三：服务端的网络地址  参数四：服务器端端口号   
        DatagramPacket packet = new DatagramPacket(data, data.length ,serverAddress ,10025);    
        socket.send(packet);//把数据发送到服务端。    
    } catch (SocketException e) {  
        e.printStackTrace();  
    } catch (UnknownHostException e) {  
        e.printStackTrace();  
    } catch (IOException e) {  
        e.printStackTrace();  
    }    
}
2、客户端接收服务器返回的数据：

public void ReceiveServerSocketData() {  
    DatagramSocket socket;  
    try {  
        //实例化的端口号要和发送时的socket一致，否则收不到data  
        socket = new DatagramSocket(1985);  
        byte data[] = new byte[4 * 1024];  
        //参数一:要接受的data 参数二：data的长度  
        DatagramPacket packet = new DatagramPacket(data, data.length);  
        socket.receive(packet);  
        //把接收到的data转换为String字符串  
        String result = new String(packet.getData(), packet.getOffset(),  
                packet.getLength());  
        socket.close();//不使用了记得要关闭  
        System.out.println("the number of reveived Socket is  :" + flag  
                + "udpData:" + result);  
    } catch (SocketException e) {  
        e.printStackTrace();  
    } catch (IOException e) {  
        e.printStackTrace();  
    }  
}




















RxJava等（简历上写你会的，用过的）
拓展–，kotlin语言，I/O大会





1.Activity的启动过程（不要回答生命周期） 
http://blog.csdn.net/luoshengyang/article/details/6689748

2.Activity的启动模式以及使用场景 
（1）manifest设置，（2）startActivity flag 
http://blog.csdn.net/CodeEmperor/article/details/50481726 
此处延伸：栈(First In Last Out)与队列(First In First Out)的区别

3.Service的两种启动方式 
（1）startService()，（2）bindService() 
http://www.jianshu.com/p/2fb6eb14fdec

4.Broadcast注册方式与区别 
（1）静态注册(minifest)，（2）动态注册 
http://www.jianshu.com/p/ea5e233d9f43 
此处延伸：什么情况下用动态注册

5.HttpClient与HttpUrlConnection的区别 
http://blog.csdn.net/guolin_blog/article/details/12452307 
此处延伸：Volley里用的哪种请求方式（2.3前HttpClient，2.3后HttpUrlConnection）

6.http与https的区别 
http://blog.csdn.net/whatday/article/details/38147103 
此处延伸：https的实现原理

7.手写算法（选择冒泡必须要会） 
http://www.jianshu.com/p/ae97c3ceea8d

8.进程保活（不死进程） 
http://www.jianshu.com/p/63aafe3c12af 
此处延伸：进程的优先级是什么（下面这篇文章，都有说） 
https://segmentfault.com/a/1190000006251859

9.进程间通信的方式 
（1）AIDL，（2）广播，（3）Messenger 
AIDL : https://www.jianshu.com/p/a8e43ad5d7d2 
https://www.jianshu.com/p/0cca211df63c 
Messenger : http://blog.csdn.net/lmj623565791/article/details/47017485 
此处延伸：简述Binder ， http://blog.csdn.net/luoshengyang/article/details/6618363/

10.加载大图 
PS：有家小公司（规模写假的，给骗过去了），直接把项目给我看，让我说实现原理。。 
最让我无语的一次面试，就一个点问的我底裤都快穿了，就差帮他们写代码了。。 
http://blog.csdn.net/lmj623565791/article/details/49300989

11.三级缓存（各大图片框架都可以扯到这上面来） 
（1）内存缓存，（2）本地缓存，（3）网络 
内存：http://blog.csdn.net/guolin_blog/article/details/9526203 
本地：http://blog.csdn.net/guolin_blog/article/details/28863651

12.MVP框架（必问） 
http://blog.csdn.net/lmj623565791/article/details/46596109 
此处延伸：手写mvp例子，与mvc之间的区别，mvp的优势

13.讲解一下Context 
http://blog.csdn.net/lmj623565791/article/details/40481055

14.JNI 
http://www.jianshu.com/p/aba734d5b5cd 
此处延伸：项目中使用JNI的地方，如：核心逻辑，密钥，加密逻辑

15.java虚拟机和Dalvik虚拟机的区别 
http://www.jianshu.com/p/923aebd31b65

16.线程sleep和wait有什么区别 
http://blog.csdn.net/liuzhenwen/article/details/4202967

17.View，ViewGroup事件分发 
http://blog.csdn.net/guolin_blog/article/details/9097463 
http://blog.csdn.net/guolin_blog/article/details/9153747

18.保存Activity状态 
onSaveInstanceState() 
http://blog.csdn.net/yuzhiboyi/article/details/7677026

19.WebView与js交互（调用哪些API） 
http://blog.csdn.net/cappuccinolau/article/details/8262821/

20.内存泄露检测，内存性能优化 
http://blog.csdn.net/guolin_blog/article/details/42238627 
这篇文章有四篇，很详细。 
此处延伸： 
（1）内存溢出（OOM）和内存泄露（对象无法被回收）的区别。 
（2）引起内存泄露的原因

21.布局优化 
http://blog.csdn.net/guolin_blog/article/details/43376527

22.自定义view和动画 
以下两个讲解都讲得很透彻，这部分面试官多数不会问很深，要么就给你一个效果让你讲原理。 
（1）http://www.gcssloop.com/customview/CustomViewIndex 
（2）http://blog.csdn.net/yanbober/article/details/50577855

23.设计模式（单例，工厂，观察者。作用，使用场景） 
一般说自己会的就ok，不要只记得名字就一轮嘴说出来，不然有你好受。 
http://blog.csdn.net/jason0539/article/details/23297037/ 
此处延伸：Double Check的写法被要求写出来。

24.String，Stringbuffer，Stringbuilder 区别 
http://blog.csdn.net/kingzone_2008/article/details/9220691

25.开源框架，为什么使用，与别的有什么区别 
这个问题基本必问。在自己简历上写什么框架，他就会问什么。 
如：Volley，面试官会问我Volley的实现原理，与okhttp和retrofit的区别。 
开源框架很多，我就选几个多数公司都会用的出来（框架都是针对业务和性能，所以不一定出名的框架就有人用） 
网络请求：Volley，okhttp，retrofit 
异步：RxJava，AsyncTask 
图片处理：Picasso，Glide 
消息传递：EventBus 
以上框架请自行查找，太多了就不贴出来了。

26.RecyclerView 
这个挺搞笑的。有另外一个同事也在找工作，面试官嫌他没用过RecyclerView直接pass掉。 
http://blog.csdn.net/lmj623565791/article/details/45059587



Android-BAT-Java 面试题分类： 
- 1.Java特性 
- 2.字符串String、数组、数据类型转换 
- 3.Java各方面基础 
- 4.抽象类与接口 
- 5.JVM、垃圾回收（GC） 
- 6.Java数据结构和算法 
- 7.设计模式 
- 8.泛型与枚举 
- 9.常用网络协议 
- 10.Java IO 与 NIO 
- 11.多线程，并发及线程基础



Android-BAT-Android 面试题分类： 
- 1.四大组件 
- 2.Fragment 
- 3.自定义组件、动画 
- 4.存储 
- 5.网络 
- 6.图片 
- 7.布局 
- 8.性能优化 
- 9.JNI 
- 10.进程间通信（简称：IPC） 
- 11.WebView 
- 12.进程保活 
- 13.杂7杂8

1.四大组件 
- Activity 
https://www.jianshu.com/p/7c193337702d 
- Service 
https://www.jianshu.com/p/d963c55c3ab9 
https://www.jianshu.com/p/e04c4239b07e 
- Content Provider 
https://www.jianshu.com/p/ea8bc4aaf057 
- Broadcast Receiver 
https://www.jianshu.com/p/ca3d87a4cdf3 
请阅读上面的基础知识

（1.1）四大组件是什么 
看开头 
（1.2）四大组件的生命周期 
看开头文章 
（1.3）Activity之间的通信方式 
https://www.jianshu.com/p/f836432396f0 
（1.4）横竖屏切换的时候，Activity 各种情况下的生命周期 
https://blog.csdn.net/hzw19920329/article/details/51345971 
（1.5）Activity与Fragment之间生命周期比较 
https://www.jianshu.com/p/b1ff03a7bb1f 
（1.6）Activity上有Dialog的时候按Home键时的生命周期 
https://blog.csdn.net/hanhan1016/article/details/47977489 
（1.7）两个Activity 之间跳转时必然会执行的是哪几个方法？ 
https://blog.csdn.net/m_xiaoer/article/details/72881082 
（1.8）Activity的四种启动模式对比以及使用场景 
https://blog.csdn.net/CodeEmperor/article/details/50481726 
（1.9）Activity状态保存与恢复 
https://blog.csdn.net/sinat_33921105/article/details/78631823 
（1.10）Activity 怎么和Service 绑定 
https://www.jianshu.com/p/5d73389f3ab2 
（1.11）Service和Activity怎么进行数据交互？ 
https://www.jianshu.com/p/cd69f208f395 
（1.12）Service的开启方式 
https://www.jianshu.com/p/2fb6eb14fdec 
（1.13）请描述一下Service 的生命周期 
https://www.jianshu.com/p/8d0cde35eb10 
（1.14）谈谈你对ContentProvider的理解 
https://www.jianshu.com/p/f5ec75a9cfea 
（1.15）ContentProvider、ContentResolver、ContentObserver 之间的关系 
https://blog.csdn.net/heqiangflytosky/article/details/31777363 
（1.16）请描述一下广播BroadcastReceiver的理解（实现原理） 
https://www.jianshu.com/p/ca3d87a4cdf3 
（1.17）广播的分类 
https://www.jianshu.com/p/ca3d87a4cdf3 
（1.18）广播使用的方式和场景 
https://www.jianshu.com/p/ca3d87a4cdf3 
（1.19）本地广播和全局广播有什么差别？ 
https://www.jianshu.com/p/bfbb6ebc1c04 
（1.20）Application 和 Activity 的 Context 对象的区别 
https://www.cnblogs.com/liyiran/p/5283551.html

2.Fragment 
有个大神自己封装了Fragment，基本覆盖了大部分你能遇到的坑，看看他的文章： 
https://www.jianshu.com/p/d9143a92ad94 
（2.1）什么是Fragment 
https://www.jianshu.com/p/2bf21cefb763 
（2.2）为什么要用Fragment 
https://www.cnblogs.com/shaweng/p/3918985.html 
（2.3）Fragment与Activity的通信方式 
https://www.jianshu.com/p/825eb1f98c19 
（2.4）Fragment各种情况下的生命周期 
https://www.jianshu.com/p/b1ff03a7bb1f 
（2.5）Fragment之间传递数据的方式？ 
https://www.jianshu.com/p/f87baad32662 
（2.6）Fragment的add与replace的区别 
https://blog.csdn.net/gsw333/article/details/51858524 
（2.7）用Fragment有遇过什么坑吗，怎么解决 
https://www.jianshu.com/p/d9143a92ad94 
（2.8）getFragmentManager，getSupportFragmentManager ，getChildFragmentManager三者之间的区别 
https://blog.csdn.net/allan_bst/article/details/64920076 
（2.9）FragmentPagerAdapter与FragmentStatePagerAdapter的区别与使用场景 
https://blog.csdn.net/lamp_zy/article/details/52446842

3.自定义组件、动画 
了解自定义view，这里： 
https://www.jianshu.com/p/c84693096e41 
（3.1）描述一下View的绘制流程 
https://www.jianshu.com/p/060b5f68da79 
（3.2）说说自定义view的几个构造函数 
http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2016/0806/4575.html 
（3.3）View 里面的 onSavedInstanceState和onRestoreInstanceState的作用 
https://blog.csdn.net/shouniezhe/article/details/47705001 
（3.4）onLayout() 和Layout()的区别 
https://blog.csdn.net/h183288132/article/details/50184423 
（3.5）描述一下getX、getRawX、getTranslationX 
https://blog.csdn.net/dmk877/article/details/51550031 
（3.6）Android中的动画有哪几类，它们的特点和区别是什么 
https://www.jianshu.com/p/420629118c10 
（3.7）Interpolator和TypeEvaluator的作用 
https://www.cnblogs.com/wondertwo/p/5327586.html 
（3.8）请描述一下View事件传递分发机制 
https://www.jianshu.com/p/e99b5e8bd67b 
（3.9）事件分发中的onTouch 和onTouchEvent 有什么区别，又该如何使用？ 
https://blog.csdn.net/huiguixian/article/details/22193977 
（3.10）View和ViewGroup分别有哪些事件分发相关的回调方法 
https://www.jianshu.com/p/e99b5e8bd67b 
（3.11）View刷新机制 
https://blog.csdn.net/chenzhiqin20/article/details/8628952

4.存储 
（4.1）描述一下你知道的数据存储方式 
https://www.jianshu.com/p/540e44f00d3e 
（4.2）SharedPreferences的应用场景，核心原理是什么 
https://www.jianshu.com/p/ae2c7004179d 
https://blog.csdn.net/dbs1215/article/details/78531258 
（4.3）SharedPreferences是线程安全的吗？ 
去源码看看有没有同步锁就知道了，答案是线程安全的。 
（4.4）描述一下图片存储在本地的方式 
https://www.jianshu.com/p/8cede074ba5b 
https://blog.csdn.net/ccpat/article/details/45314175 
（4.5）sqlite升级，增加字段的语句 
https://blog.csdn.net/xu_song/article/details/49658195 
（4.6）数据库框架对比和源码分析 
https://blog.csdn.net/u012702547/article/details/52226163 
https://www.jianshu.com/p/c4e9288d2ce6 
（4.7）数据库的优化 
https://www.jianshu.com/p/3b4452fc1bbd 
（4.8）数据库数据迁移问题 
https://www.cnblogs.com/awkflf11/p/6033074.html

5.网络 
（5.1）描述一次网络请求的流程 
https://blog.csdn.net/seu_calvin/article/details/53304406 
（5.2）HTTP报文结构 
https://www.jianshu.com/p/e544b7a76dac 
（5.3）HttpClient和HttpURLConnection的区别 
https://www.cnblogs.com/zhousysu/p/5483896.html 
（5.4）Volley，okhttp，retrofit之间的区别和核心原理和使用场景 
https://www.jianshu.com/p/050c6db5af5a 
（5.5）描述一下https 
https://showme.codes/2017-02-20/understand-https/ 
（5.6）https中哪里用了对称加密，哪里用了非对称加密，对加密算法（如RSA）等是否有了解? 
https://showme.codes/2017-02-20/understand-https/ 
（5.7）说一下三次握手，四次挥手的具体细节 
我经常用面试问别人这道题，哈哈，为什么不能两次握手呢？要三次？ 
https://www.cnblogs.com/Andya/p/7272462.html 
（5.8）描述一下socket是什么东西 
https://www.jianshu.com/p/089fb79e308b 
（5.9）从网络加载一个10M的图片，说下注意事项 
https://www.jianshu.com/p/7c81d3742c38 
https://www.jianshu.com/p/f850a23ab99c 
（5.10）TCP与UDP的区别 
https://blog.csdn.net/li_ning_/article/details/52117463 
（5.11）client如何确定自己发送的消息被server收到? 
HTTP协议里，有请求就有响应，根据响应的状态吗就能知道拉。 
（5.12）WebSocket与socket的区别 
https://blog.csdn.net/wwd0501/article/details/54582912 
（5.13）网络请求缓存处理，okhttp如何处理网络缓存的 
看源码，看缓存策略 
http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0326/2643.html 
（5.14）自己去设计网络请求框架，怎么做？（随便套个开源框架的原理） 
就套okhttp的，被google承认并使用的框架，准没错。 
http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0326/2643.html

6.图片 
（6.1）说一下OOM的原因，如何避免 
https://blog.csdn.net/boyupeng/article/details/47726765 
（6.2）说一下三级缓存的原理 
https://www.jianshu.com/p/97455f080065 
（6.3）描述一下内存缓存的容器 
LruCache其实是一个Hash表，内部使用的是LinkedHashMap存储数据 
https://blog.csdn.net/justloveyou_/article/details/71713781 
（6.4）图片库对比 
https://www.jianshu.com/p/fc72001dc18d 
（6.5）图片库的源码分析 
https://blog.csdn.net/guolin_blog/article/details/53759439 
（6.6）图片框架缓存实现 
郭霖大神写了几篇文章介绍Glide，都有详细介绍 
https://blog.csdn.net/guolin_blog/article/details/53759439 
（6.7）LRUCache原理 
https://www.cnblogs.com/tianzhijiexian/p/4248677.html 
（6.9）自己去实现图片库，怎么做？（随便套个开源框架的原理） 
套Glide的就OK拉，从设计思想，然后到实现方式 
（6.12）说说Glide内存缓存的具体实现？ 
https://blog.csdn.net/guolin_blog/article/details/54895665

7.布局 
（7.1）说一下布局性能的排序，谁的效率最高 
https://blog.csdn.net/seu_calvin/article/details/53047682 
（7.2）描述一下约束布局 
https://blog.csdn.net/zhaoyanjun6/article/details/62896784 
（7.3）关于布局优化的方案 
学会用约束布局，基本优化很多了，但是老方法还是要会，面试官多数比较守旧。因为资深，年纪也可能稍微大一点，哈哈。 
https://www.cnblogs.com/hoolay/p/6248514.html 
（7.4）怎么检测布局深度 
https://blog.csdn.net/hp910315/article/details/52684039 
（7.5）LinearLayout、RelativeLayout、FrameLayout的特性及对比，并介绍使用场景。 
https://blog.csdn.net/seu_calvin/article/details/53047682

8.性能优化 
PS：性能优化包括内存，处理效率，视觉流畅度，CPU，电量，流量等方面，针对手机的性能去做相应的方案。个人认为更应该把握好内存优化、处理效率（代码质量）、视觉流畅度（布局优化）。 
（8.1）ANR产生的原因是什么？ 
https://www.jianshu.com/p/7fd95bc2a55c 
（8.3）oom是什么？ 
https://blog.csdn.net/hudfang/article/details/51781997 
（8.4）什么情况导致oom？ 
https://blog.csdn.net/hudfang/article/details/51781997 
（8.5）有什么解决方法可以避免OOM？ 
https://blog.csdn.net/hudfang/article/details/51781997 
（8.6）Oom 是否可以try catch？为什么？ 
有一种情况可以：在try语句中声明了很大的对象，导致OOM，并且可以确认OOM是由try语句中的对象声明导致的，但是这通常不是合适的做法。 
（8.7）内存泄漏是什么？ 
https://www.jianshu.com/p/bf159a9c391a 
（8.8）什么情况导致内存泄漏？ 
https://www.jianshu.com/p/90caf813682d 
（8.9）如何防止线程的内存泄漏？ 
https://www.jianshu.com/p/90caf813682d 
https://www.cnblogs.com/ywq-come/p/5926422.html 
（8.10）内存泄露的解决方法 
https://blog.csdn.net/carson_ho/article/details/79407707 
（8.11）内存泄漏和内存溢出区别？ 
https://blog.csdn.net/sinat_29255093/article/details/52556760 
（8.12）如何对Android 应用进行性能分析以及优化? 
这个作者做了很多片性能优化的文章，建议看完 
https://www.jianshu.com/p/da2a4bfcba68 
（8.13）怎么去除无用代码？ 
https://blog.csdn.net/it_flower/article/details/52305558 
（8.14）性能优化如何分析systrace？ 
https://www.jianshu.com/p/6f528e862d31 
（8.15）用IDE如何分析内存泄漏？ 
跑一段你觉得有问题的代码段，gc，再跑，再gc，看看内存会不会一直上升 
https://blog.csdn.net/u010944680/article/details/51721532 
（8.16）Java多线程引发的性能问题，怎么解决？ 
https://zhuanlan.zhihu.com/p/23389156 
（8.17）启动页白屏及黑屏解决？ 
https://blog.csdn.net/zivensonice/article/details/51691136 
（8.18）启动太慢怎么解决？ 
应用启动速度，取决于你在application里面时候做了什么事情，比如你集成了很多sdk，并且sdk的init操作都需要在主线程里实现，那自然就慢咯。在非必要的情况下可以把加载延后。或者丢子线程里。 
https://www.jianshu.com/p/4f10c9a10ac9 
（8.19）怎么保证应用启动不卡顿？ 
同上面一个道理，也可以做个闪屏页当缓冲时间。 
https://www.jianshu.com/p/4f10c9a10ac9 
（8.20）App启动崩溃异常捕捉 
https://www.jianshu.com/p/fb28a5322d8a 
（8.21）自定义View注意事项 
减少不必要的调用invalidate()方法 
https://blog.csdn.net/whb20081815/article/details/74474736 
（8.22）现在下载速度很慢,试从网络协议的角度分析原因,并优化(提示：网络的5层都可以涉及)。 
这个问题让我去请教一下再来回答 
（8.23）Https请求慢的解决办法（提示：DNS，携带数据，直接访问IP） 
https://www.cnblogs.com/mylanguage/p/5635524.html 
（8.24）如何保持应用的稳定性 
内存，布局优化，代码质量，数据结构效率，针对业务合理的设计框架 
（8.25）RecyclerView和ListView的性能对比 
https://blog.csdn.net/fanenqian/article/details/61191532 
（8.26）ListView的优化 
可以说上分页加载哦 
https://www.cnblogs.com/yuhanghzsd/p/5595532.html 
（8.27）RecycleView优化 
https://blog.csdn.net/axi295309066/article/details/52741810 
https://www.jianshu.com/p/411ab861034f 
（8.28）View渲染 
https://www.cnblogs.com/ldq2016/p/6668148.html 
（8.29）Bitmap如何处理大图，如一张30M的大图，如何预防OOM 
重点是在对于对内存的了解以及内存使用率的掌握 
https://blog.csdn.net/guolin_blog/article/details/9316683 
（8.30）java中的四种引用的区别以及使用场景 
https://blog.csdn.net/qq_23547831/article/details/46505287 
（8.31）强引用置为null，会不会被回收？ 
会，GC执行时，就被回收掉，前提是没有被引用的对象 
一定要了解垃圾回收原理 
https://blog.csdn.net/qq_33048603/article/details/52727991

9.JNI 
（9.1）请介绍一下NDK 
https://www.jianshu.com/p/fa613762f516 
https://blog.csdn.net/carson_ho/article/details/73250163 
（9.2）什么是NDK库? 
https://blog.csdn.net/carson_ho/article/details/73250163 
（9.3）如何在JNI中注册native函数，有几种注册方式? 
https://blog.csdn.net/wwj_748/article/details/52347341 
（9.4）Java如何调用c、c++语言？ 
https://www.jianshu.com/p/27404a899d88 
（9.5）JNI如何调用java层代码？ 
https://www.jianshu.com/p/4893848a3249 
（9.6）你用JNI来实现过什么功能吗？怎么实现的？ 
加密处理、影音方面、图形图像处理 
https://blog.csdn.net/summer0527/article/details/1827186

10.进程间通信（简称：IPC） 
（10.1）进程间通信的方式？ 
https://www.jianshu.com/p/ce1e35c84134 
（10.2）Binder机制的作用和原理 
http://www.cnblogs.com/innost/archive/2011/01/09/1931456.html 
https://blog.csdn.net/luoshengyang/article/details/6618363/ 
（10.3）简述IPC？ 
https://blog.csdn.net/luoshengyang/article/details/6618363/ 
（10.4）什么是AIDL？ 
https://www.jianshu.com/p/d1fac6ccee98 
https://www.jianshu.com/p/a5c73da2e9be 
（10.5）AIDL解决了什么问题？ 
官方文档： 
Note: Using AIDL is necessary only if you allow clients from different applications to access your service for IPC and want to handle multithreading in your service. If you do not need to perform concurrent IPC across different applications, you should create your interface by implementing a Binder or, if you want to perform IPC, but do not need to handle multithreading, implement your interface using a Messenger. Regardless, be sure that you understand Bound Services before implementing an AIDL. 
“只有当你允许来自不同的客户端访问你的服务并且需要处理多线程问题时你才必须使用AIDL” 
（10.6）AIDL如何使用？ 
https://www.jianshu.com/p/d1fac6ccee98 
https://www.jianshu.com/p/a5c73da2e9be 
（10.8）Android进程分类？ 
https://blog.csdn.net/zhongshujunqia/article/details/72458271 
（10.9）进程和 Application 的生命周期？ 
（10.10）进程调度 
https://blog.csdn.net/innost/article/details/6940136 
（10.11）谈谈对进程共享和线程安全的认识 
https://blog.csdn.net/coding_glacier/article/details/8230159 
https://blog.csdn.net/oweixiao123/article/details/9057445 
问你线程安全的时候，不止要回答主线程跟子线程之间的切换，还有数据结构处理的线程安全问题，多线程操作同一个数据的一致性问题，等等。

11.WebView 
https://www.jianshu.com/p/3c94ae673e2a 
https://www.jianshu.com/p/52ec85259ccc 
过一遍这个 
（11.1）描述一下Webview的作用 
WebView控件功能强大，除了具有一般View的属性和设置外，还可以对url请求、页面加载、渲染、页面交互进行强大的处理。 
（11.2）WebView的内核是什么 
Android的Webview在低版本和高版本采用了不同的webkit版本内核，4.4后直接使用了Chrome。 
（11.3）描述一下WebView与js的交互方式 
https://blog.csdn.net/carson_ho/article/details/64904691 
（11.4）描述一下WebView的缓存机制 
https://www.jianshu.com/p/5e7075f4875f 
（11.5）关于WebView的优化你知道哪些 
https://www.jianshu.com/p/95d4d73be3d1 
（11.6）有没有用过第三方WebView组件？讲一讲优势 
https://www.jianshu.com/p/d3ef9c62b6c8

12.进程保活 
关于守护进程、不死进程、进程保活这些话题，有几句话想说一下： 
这个近期是面的越来越少。在google的控制下，高版本基本上是扼杀了这种无赖行为，市面上现在做进程保活基本都是走厂商白名单和系统签名进程等方式，又或者应用之间互相拉起，各大应用相互合作。 
但并不是说不能做，只能用各种方式混搭，去提高保活的成功率。

熟悉进程保活的干货： 
https://www.jianshu.com/p/63aafe3c12af 
https://blog.csdn.net/tencent_bugly/article/details/52192423 
https://www.cnblogs.com/Doing-what-I-love/p/5530291.html

还有一个大神深入研究这个话题，并开源出自己的代码，在github上挺受欢迎的，大家可以去拿来试一下： 
blog：https://blog.csdn.net/marswin89/article/details/48015453 
github：https://github.com/Marswin/MarsDaemon

看完以上文章，so以下的问题大家心里都有数了吧？ 
（12.1）做过进程保活吗？ 
（12.2）5.0下和5.0上的保活方式了解吗？ 
（12.3）描述一下进程回收的过程 
（12.4）如何降低进程的oom_adj

13.杂7杂8 
（13.1）Handler机制和底层实现 
https://www.cnblogs.com/ryanleee/p/8204450.html 
https://www.jianshu.com/p/b03d46809c4d 
（13.2）Handler、Thread和HandlerThread的差别 
https://blog.csdn.net/lmj623565791/article/details/47079737/ 
（13.3）handler发消息给子线程，looper怎么启动？ 
什么问题呢。。发消息就是把消息塞进去消息队列，looper在应用起来的时候已经就启动了，一直在轮询取消息队列的消息。 
（13.4）关于Handler，在任何地方new Handler 都是什么线程下? 
我自己看不太懂这个问题 
（13.5）ThreadLocal原理，实现及如何保证Local属性？ 
https://blog.csdn.net/singwhatiwanna/article/details/48350919 
（13.6）请解释下在单线程模型中Message、Handler、Message Queue、Looper之间的关系 
https://blog.csdn.net/lovedren/article/details/51701477 
（13.7）AsyncTask机制 
https://blog.csdn.net/yanbober/article/details/46117397 
（13.8）AsyncTask原理及不足 
https://www.cnblogs.com/absfree/p/5357678.html 
（13.9）如何取消AsyncTask？ 
调用cancel() 
但是： 
https://www.jianshu.com/p/0c6f4b6ed558 
（13.10）为什么不能在子线程更新UI？ 
https://blog.csdn.net/cewei711/article/details/53028358?locationNum=2&fps=1 
（13.11）LruCache默认内存缓存大小 
基本上设置为手机内存的1/8 
（13.12）ContentProvider的权限管理(解答：读写分离，权限控制-精确到表级，URL控制) 
（13.13）如何通过广播拦截和abort一条短信？ 
https://blog.csdn.net/ljw124213/article/details/50492449 
（13.14）广播是否可以请求网络？ 
子线程可以，主线程超过10s引起anr 
（13.15）广播引起anr的时间限制是多少？ 
onReceive的生命周期为10秒 
https://blog.csdn.net/me_dong/article/details/53582115 
（13.16）描述一下Activity栈 
https://www.jianshu.com/p/c1386015856a 
（13.17）Android线程有没有上限？ 
跟内存挂钩，我也不太清楚，自己查哈 
（13.18）线程池有没有上限？ 
跟内存挂钩，我也不太清楚，自己查哈 
http://www.trinea.cn/android/java-android-thread-pool/ 
https://blog.csdn.net/cfy137000/article/details/51422316 
（13.19）ListView重用的是什么？ 
https://blog.csdn.net/u011692041/article/details/53099584 
（13.20）Android为什么引入Parcelable？ 
https://blog.csdn.net/jaycee110905/article/details/21517853 
（13.21）有没有尝试简化Parcelable的使用？ 
as的插件 
（13.22）ListView 中图片错位的问题是如何产生的? 
https://blog.csdn.net/a394268045/article/details/50726560 
（13.23）混合开发有了解吗？ 
https://blog.csdn.net/sinat_33195772/article/details/72961814 
（13.24）知道哪些混合开发的方式？说出它们的优缺点和各自使用场景？（解答：比如:RN，weex，H5，小程序，WPA等) 
https://www.jianshu.com/p/22aa14664cf9 
https://www.jianshu.com/p/52071a3d07b4 
（13.25）屏幕适配的处理技巧都有哪些? 
https://blog.csdn.net/wangwangli6/article/details/63258270 
（13.26）服务器只提供数据接收接口，在多线程或多进程条件下，如何保证数据的有序到达？ 
（13.27）动态布局的理解 
https://blog.csdn.net/ustory/article/details/42424313 
（13.28）画出 Android 的大体架构图 
https://blog.csdn.net/wang5318330/article/details/51917092 
（13.29）Recycleview和ListView的区别 
https://blog.csdn.net/sanjay_f/article/details/48830311 
（13.30）ListView图片加载错乱的原理和解决方案 
https://blog.csdn.net/a394268045/article/details/50726560 
（13.31）动态权限适配方案，权限组的概念 
https://blog.csdn.net/xc765926174/article/details/49103483 
（13.32）Android系统为什么会设计ContentProvider？ 
重点，跨进程 
https://www.cnblogs.com/zhainanJohnny/articles/3275908.html 
（13.33）下拉状态栏是不是影响activity的生命周期 
不会 
（13.36）Bitmap 使用时候注意什么？ 
https://blog.csdn.net/newbie_coder/article/details/9842995 
（13.37）Bitmap的recycler() 
https://blog.csdn.net/lonelyroamer/article/details/7569248 
（13.38）Android中开启摄像头的主要步骤 
https://www.yiibai.com/android/android_camera.html 
（13.39）ViewPager使用细节，如何设置成每次只初始化当前的 
懒加载 
https://www.jianshu.com/p/e324e8378948 
https://blog.csdn.net/linglongxin24/article/details/53205878 
（13.41）点击事件被拦截，但是想传到下面的View，如何操作？ 
问题就是viewgroup被拦截，要传到子view那里，好好看这篇分发机制的文章，你就知道了 
https://www.jianshu.com/p/e99b5e8bd67b 
（13.42）描述一下微信主页面的实现方式 
自己去研究下吧这个，无非fragment嵌套 
（13.43）invalidate和postInvalidate的区别及使用 
https://blog.csdn.net/mars2639/article/details/6650876 
（13.44）Activity-Window-View三者的差别 
https://www.jianshu.com/p/a533467f5af5 
（13.45）谈谈对Volley的理解 
https://blog.csdn.net/ysh06201418/article/details/46443235 
（13.46）ActivityThread，AMS，WMS的工作原理 
https://www.jianshu.com/p/47eca41428d6 
（13.47）LaunchMode应用场景 
https://blog.csdn.net/CodeEmperor/article/details/50481726 
（13.48）SpareArray原理 
https://blog.csdn.net/easyer2012/article/details/37871031 
（13.49）请介绍下ContentProvider 是如何实现数据共享的？ 
https://blog.csdn.net/yhaolpz/article/details/51304345 
（13.50）IntentService原理及作用是什么？ 
https://www.jianshu.com/p/4dd46616564d 
（13.51）ApplicationContext和ActivityContext的区别 
https://www.jianshu.com/p/94e0f9ab3f1d 
（13.53）封装View的时候怎么知道view的大小 
https://blog.csdn.net/fwt336/article/details/52979876 
（13.55）AndroidManifest的作用与理解 
https://www.jianshu.com/p/6ed30112d4a4

可以问的问题：

1、我想知道公司是否定期有开技术会议，老员工是否会分享自己的一些经验等这些问题。

2、公司所做的产品主要是哪方面的

可以准备的问题：
1.你工作中最牛逼or最成功or最有贡献的一件事是什么？ 
2.项目中的亮点是哪些？怎么实现的？（实在没有的自己去找，只能用别人的案例了） 
3.做项目的过程中有没有遇到过困难？怎么克服的？

android测试
如果要选取工具，最好能贴近使用场景，挑一个能满足切身需求的，真的能帮节省工作量，提高工作效率。下面是一些常用工具。
monkey
monkeyrunner
monkeytalk
Instrumentation
UIAutomator
Espresso
Calabash
Selendroid
Robotium
Appium
SeeTest
SilkMobile
Ranorex

大概有如下几个工具：
android针对上面这些会影响到应用性能的情况提供了一些列的工具：
1 布局复杂度：
hierarchyviewer：检测布局复杂度，各视图的布局耗时情况：

Android开发者模式—GPU过渡绘制：

2 耗电量：Android开发者模式中的电量统计；
3 内存：
应用运行时内存使用情况查看：Android Studio—Memory/CPU/GPU；

内存泄露检测工具：DDMS—MAT；
4 网络：Android Studio—NetWork；
5 程序执行效率：
静态代码检查工具：Android studio—Analyze—Inspect Code.../Code cleanup... ，用于检测代码中潜在的问题、存在效率问题的代码段并提供改善方案；
DDMS—TraceView，用于查找程序运行时具体耗时在哪；
StrictMode：用于查找程序运行时具体耗时在哪，需要集成到代码中；
Andorid开发者模式—GPU呈现模式分析。
6 程序稳定性：monkey，通过monkey对程序在提交测试前做自测，可以检测出明显的导致程序不稳定的问题，执行monkey只需要一行命令，提交测试前跑一次可以避免应用刚提交就被打回的问题。
说明：
上面提到的这些工具可以进Android开发者官网性能工具介绍查看每个工具的介绍和使用说明；

Android开发者选项中有很多测试应用性能的工具，对应用性能的检测非常有帮助，具体可以查看：All about your phone's developer options和15个必知的Android开发者选项对Android开发者选项中每一项的介绍；

针对Android应用性能的优化，Google官方提供了一系列的性能优化视频教程，对应用性能优化具有非常好的指导作用，具体可以查看：优酷Google Developers或者Android Performance Patterns。

二 第三方性能优化工具介绍
除了android官方提供的一系列性能检测工具，还有很多优秀的第三方性能检测工具使用起来更方便，比如对内存泄露的检测，使用leakcanry比MAT更人性化，能够快速查到具体是哪存在内存泄露。
leakcanary：square/leakcanary · GitHub，通过集成到程序中的方式，在程序运行时检测应用中存在的内存泄露，并在页面中显示，在应用中集成leancanry后，程序运行时会存在卡顿的情况，这个是正常的，因为leancanry就是通过gc操作来检测内存泄露的，gc会知道应用卡顿，说明文档：LeakCanary 中文使用说明、LeakCanary: 让内存泄露无所遁形。
GT：GT Home，GT是腾讯开发的一款APP的随身调测平台，利用GT，可以对CPU、内存、流量、点亮、帧率/流畅度进行测试，还可以查看开发日志、crash日志、抓取网络数据包、APP内部参数调试、真机代码耗时统计等等，需要说明的是，应用需要集成GT的sdk后，GT这个apk才能在应用运行时对各个性能进行检测。
