# Android应用崩溃后异常捕获并重启并写入日志
现在安装Android系统的手机版本和设备千差万别，在模拟器上运行良好的程序安装到某款手机上说不定就出现崩溃的现象，开发者个人不可能购买所有设备逐个调试，所以在程序发布出去之后，如果出现了崩溃现象，开发者应该及时获取在该设备上导出崩溃的信息
  
Android中提供了一个全局异常的捕获，方式如下： 
1.定义一个类实现UncaughtExceptionHandler 
2.在Application中初始化，Thread.setDefaultUncaughtExceptionHandler(handler); 
3.将抓取到的错误保存到本地txt文件 
4.上传到服务器 

### （1） 定义一个CrashHandler 
```java
public class CrashHandler implements UncaughtExceptionHandler { 
    private static CrashHandler myCrashHandler; 
    private CrashHandler() {}; 
    private Context context; 
    public final static String LOGPATH =Environment.getExternalStorageDirectory() + “/usershopping/crash.txt” ; 
    private UncaughtExceptionHandler defaultExceptionHandler; 
    public synchronized static CrashHandler getInstance() { 
      if (myCrashHandler == null) { 
        myCrashHandler = new CrashHandler(); 
      } 
    return myCrashHandler; 
    } 
  public void init(Context context) { 
      this.context = context; 
      defaultExceptionHandler = Thread.getDefaultUncaughtExceptionHandler(); 
      Thread.setDefaultUncaughtExceptionHandler(this); 
  } 
  public void uncaughtException(Thread thread, Throwable ex) { 
  if(!sdCardIsAvailable()){ 
      ex.printStackTrace(); 
      new Thread() { 
      @Override 
      public void run() { 
        Looper.prepare(); 
        Toast.makeText(context, “异常退出”, Toast.LENGTH_LONG).show(); 
        Looper.loop(); 
      } 
      }.start(); 
      new Thread() { 
          @Override 
          public void run() { 
            try { 
              Thread.sleep(4000); 
            } catch (InterruptedException e) { 
              // TODO Auto-generated catch block 
              e.printStackTrace(); 
          } 
              android.os.Process.killProcess(android.os.Process.myPid()); 
          } 
      }.start(); 
    return; 
    }
        //(3)将错误日志保存到本地txt文件 
        try { 
            // 在throwable的参数里面保存的有程序的异常信息 
            StringBuffer sb = new StringBuffer(); 
            // 1.得到手机的版本信息 硬件信息 
            Field[] fields = Build.class.getDeclaredFields(); 
            for (Field filed : fields) { 
            filed.setAccessible(true); // 暴力反射 
            String name = filed.getName(); 
            String value = filed.get(null).toString(); 
            sb.append(name); 
            sb.append(“=”); 
            sb.append(value); 
            sb.append(“\n”); 
        }
        // 2.得到当前程序的版本号
        PackageInfo info = context.getPackageManager().getPackageInfo(
                context.getPackageName(), 0);
        sb.append(info.versionName);
        sb.append("\n");
        // 3.得到当前程序的异常信息
        Writer writer = new StringWriter();
        PrintWriter printwriter = new PrintWriter(writer);

        ex.printStackTrace(printwriter);
        printwriter.flush();
        printwriter.close();

        sb.append(writer.toString());

        File ff = new File(LOGPATH);
        ff.createNewFile(); 
        FileWriter fw = new FileWriter(new File(LOGPATH));
        fw.write(sb.toString());
        // System.out.println(sb); 
        fw.close();
            } catch (Exception e1) {
        e1.printStackTrace();
    }

    new Thread() {

        @Override
        public void run() {
            Looper.prepare();

            Toast.makeText(context, "异常退出", Toast.LENGTH_LONG).show();
            Looper.loop();

        }

    }.start();
    //出现异常后，退出app
    new Thread() {

        @Override
        public void run() {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
            android.os.Process.killProcess(android.os.Process.myPid());
        }

    }.start();

}

/**
 * 检测sdcard是否可用
 *
 * @return true为可用，否则为不可用
 */
public static boolean sdCardIsAvailable() {
    String status = Environment.getExternalStorageState();
    if (!status.equals(Environment.MEDIA_MOUNTED))
        return false;
    return true;
}


### （2）在Application中开一个子线程，初始化 
//抓错误日志 
new Thread(){ 
@Override 
public void run() { 
//把异常处理的handler设置到主线程里面 
CrashHandler ch = CrashHandler.getInstance(); 
ch.init(getApplicationContext()); 
} 
}.start(); 
### （4）上传到服务器 
前几步，是只要发生崩溃，我们就会将崩溃日志保存到本地，然后强制退出app，接下来我们就可以在启动app的首个activity中，即闪屏页里，判断本地是否有错误日志txt，有就开启服务上传，上传成功后就要删除。这样做可以避免多个错误日志覆盖的问题。
```java 
//检查cash.txt是否存在 存在就开启服务  上传到服务器
    String path= Environment.getExternalStorageDirectory() + "/usershopping/crash.txt";
    final File cashFile=new File(path);
    if (cashFile.exists()){
        startService(new Intent(context, UploadCashService.class));
    }
    //这个服务是intentService  自动开启异步，执行结束后自动销毁

    public class UploadCashService extends IntentService {

        public UploadCashService(String name) {
            super(name);
        }
        public UploadCashService() {
            super("UploadCashService");
        }

        @Override
        public IBinder onBind(Intent intent) {
            return null;
        }

        @Override
        protected void onHandleIntent(@Nullable Intent intent) {

        }

        @SuppressWarnings("deprecation")
        @Override
        public int onStartCommand(Intent intent, int flags, int startId) {
            //开启上传
            upLoadCash(getApplicationContext());
            return super.onStartCommand(intent, flags, startId);

            }
        private void upLoadCash(final Context context) {
            String path= Environment.getExternalStorageDirectory() + "/usershopping/crash.txt";
            final File file=new File(path);
            final Map<String, String> headers = new HashMap<String, String>();
            final Map<String, String> params = new HashMap<String, String>();
            params.put("v1", "true");
            headers.put(App.TOKEN_NAME, SharedPreferencesUtils.getString(context, "TOKEN"));
            headers.put("ContentType", "multipart/form-data");
            headers.put("dcode",CommonUtil.getDeviceInfo(context));//手机唯一标识
            headers.put("brand", android.os.Build.BRAND);   //手机品牌
            headers.put("model", Build.MODEL);//手机型号
            headers.put("ver",android.os.Build.VERSION.RELEASE);//系统版本号
                        headers.put("appver", String.valueOf(CommonUtil.getVersion(context)));//app版本号
            headers.put("verName", String.valueOf(CommonUtil.getVersionName(context)));//app版本名
            headers.put("appname","CLIENT");
            PostFormBuilder post = OkHttpUtils.post()
                .url(new Constant(context).CASHUPLOAD)
                .params(params)
                .headers(headers);

            post.addFile("mFile" , System.currentTimeMillis() + Math.random() * 10 + ".txt",
                file);
            post.build().execute(new Callback<Object>() {

                @Override
                public Object parseNetworkResponse(
                        Response response)
                        throws Exception {
                    // TODO Auto-generated method stub
                    if (file.exists()){
                        file.delete();
                    }

                    return null;
                }

                @Override
                public void onError(Call call,
                                    Exception e) {
                    // TODO Auto-generated method stub
                    MyToast.showToast(context, "错误日志上传失败");
                }

                @Override
                public void inProgress(float progress) {
                    super.inProgress(progress);

                }

                @Override
                public void onResponse(Object response) {
                    // TODO Auto-generated method stub

                }
            });
}

```
