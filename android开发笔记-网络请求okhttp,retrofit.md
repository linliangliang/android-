关于网络请求，在“android面试笔记”中有记录，这里单独拿出来
# OKHTTP
OKHTTP用于替代HttpUrlConnection和Apache HttpClient(android API23 里已移除HttpClient)。

okhttp有自己的官网 ：OKHttp官网[http://square.github.io/okhttp/]

- 一般的get请求
- 一般的post请求
- 基于Http的文件上传
- 文件下载
- 加载图片
- 支持请求回调，直接返回对象、对象集合
- 支持session的保持

### 一、get的同步请求
对于同步请求在请求时需要开启子线程，请求成功后需要跳转到UI线程修改UI。
```java
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
```
### get的异步请求
```java
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
```
- 以上就是发送一个get请求的步骤，首先构造一个Request对象，参数最起码有个url，当然你可以通过Request.Builder设置更多的参数比如：header、method等。

- 然后通过request的对象去构造得到一个Call对象，类似于将你的请求封装成了任务，既然是任务，就会有execute()和cancel()等方法。

- 最后，我们希望以异步的方式去执行请求，所以我们调用的是call.enqueue，将call加入调度队列，然后等待任务执行完成，我们在Callback中即可得到结果。
注意事项：
1，回调接口的onFailure方法和onResponse执行在子线程。
2，response.body().string()方法也必须放在子线程中。当执行这行代码得到结果后，再跳转到UI线程修改UI。

### post异步请求
```java
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
```
##### POST请求传递参数的方法总结
1，使用FormBody传递键值对参数

这种方式用来上传String类型的键值对
使用示例如下：
```java
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
```
2、使用RequestBody传递Json或File对象
RequestBody是抽象类，故不能直接使用，但是他有静态方法create，使用这个方法可以得到RequestBody对象。
上传json对象使用示例如下：
```java
OkHttpClient client = new OkHttpClient();//创建OkHttpClient对象。
MediaType JSON = MediaType.parse("application/json; charset=utf-8");//数据类型为json格式，
String jsonStr = "{\"username\":\"lisi\",\"nickname\":\"李四\"}";//json数据.
RequestBody body = RequestBody.create(JSON, josnStr);
Request request = new Request.Builder()
    .url("http://www.baidu.com")
    .post(body)
    .build();
client.newCall(request).enqueue(new Callback() {。。。});//此处省略回调方法。
```
上传File对象使用示例如下：
```java
OkHttpClient client = new OkHttpClient();//创建OkHttpClient对象。
MediaType fileType = MediaType.parse("File/*");//，
File file = new File("path");//file对象.
RequestBody body = RequestBody.create(fileType , file );
Request request = new Request.Builder()
    .url("http://www.baidu.com")
    .post(body)
    .build();
client.newCall(request).enqueue(new Callback() {。。。});//此处省略回调方法。
```
3、使用MultipartBody同时传递键值对参数和File对象
这个字面意思是多重的body。我们知道FromBody传递的是字符串型的键值对，RequestBody传递的是多媒体，那么如果我们想二者都传递怎么办？此时就需要使用MultipartBody类。
使用示例如下：
```java
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
```
***4、OkHttp文件上传（1）：实现文件上传进度监听***
**关键点是自定义 支持进度反馈的RequestBody：**
重写write方法按照自定义的SEGMENT_SIZE 来写文件，从而监听进度
**参考[http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0904/3416.html]**

##### 文件下载
在OKHttp中并没有提供下载文件的功能，但是在Response中可以获取流对象，有了流对象我们就可以自己实现文件的下载。代码如下：
这段代码写在回调接口CallBack的onResponse方法中：
```java
try{
InputStream  is = response.body().byteStream();//从服务器得到输入流对象
long sum = 0;
File dir = new File(mDestFileDir); 
if (!dir.exists()){
    dir.mkdirs();
}
File file = new File(dir, mdestFileName);//根据目录和文件名得到file对象
FileOutputStream  fos = new FileOutputStream(file);
byte[] buf = new byte[1024*8];
int len = 0;
while ((len = is.read(buf)) != -1){
    fos.write(buf, 0, len);
}
fos.flush();
return file;

}
```
```java
/**
     * 下载文件
     * @param fileUrl 文件url
     * @param destFileDir 存储目标目录
     */
    public <T> void downLoadFile(String fileUrl, final String destFileDir, final ReqCallBack<T> callBack) {
        final String fileName = MD5.encode(fileUrl);
        final File file = new File(destFileDir, fileName);
        if (file.exists()) {
            successCallBack((T) file, callBack);
            return;
        }
        final Request request = new Request.Builder().url(fileUrl).build();
        final Call call = mOkHttpClient.newCall(request);
        call.enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                Log.e(TAG, e.toString());
                failedCallBack("下载失败", callBack);
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                InputStream is = null;
                byte[] buf = new byte[2048];
                int len = 0;
                FileOutputStream fos = null;
                try {
                    long total = response.body().contentLength();
                    Log.e(TAG, "total------>" + total);
                    long current = 0;
                    is = response.body().byteStream();
                    fos = new FileOutputStream(file);
                    while ((len = is.read(buf)) != -1) {
                        current += len;
                        fos.write(buf, 0, len);
                        Log.e(TAG, "current------>" + current);
                    }
                    fos.flush();
                    successCallBack((T) file, callBack);
                } catch (IOException e) {
                    Log.e(TAG, e.toString());
                    failedCallBack("下载失败", callBack);
                } finally {
                    try {
                        if (is != null) {
                            is.close();
                        }
                        if (fos != null) {
                            fos.close();
                        }
                    } catch (IOException e) {
                        Log.e(TAG, e.toString());
                    }
                }
            }
        });
    }
```

带进度文件下载
```java
/**
     * 下载文件
     * @param fileUrl 文件url
     * @param destFileDir 存储目标目录
     */
    public <T> void downLoadFile(String fileUrl, final String destFileDir, final ReqProgressCallBack<T> callBack) {
        final String fileName = MD5.encode(fileUrl);
        final File file = new File(destFileDir, fileName);
        if (file.exists()) {
            successCallBack((T) file, callBack);
            return;
        }
        final Request request = new Request.Builder().url(fileUrl).build();
        final Call call = mOkHttpClient.newCall(request);
        call.enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                Log.e(TAG, e.toString());
                failedCallBack("下载失败", callBack);
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                InputStream is = null;
                byte[] buf = new byte[2048];
                int len = 0;
                FileOutputStream fos = null;
                try {
                    long total = response.body().contentLength();
                    Log.e(TAG, "total------>" + total);
                    long current = 0;
                    is = response.body().byteStream();
                    fos = new FileOutputStream(file);
                    while ((len = is.read(buf)) != -1) {
                        current += len;
                        fos.write(buf, 0, len);
                        Log.e(TAG, "current------>" + current);
                        progressCallBack(total, current, callBack);
                    }
                    fos.flush();
                    successCallBack((T) file, callBack);
                } catch (IOException e) {
                    Log.e(TAG, e.toString());
                    failedCallBack("下载失败", callBack);
                } finally {
                    try {
                        if (is != null) {
                            is.close();
                        }
                        if (fos != null) {
                            fos.close();
                        }
                    } catch (IOException e) {
                        Log.e(TAG, e.toString());
                    }
                }
            }
        });
    }
```
 接口ReqProgressCallBack.java实现
 ```java
 public interface ReqProgressCallBack<T>  extends ReqCallBack<T>{
    /**
     * 响应进度更新
     */
    void onProgress(long total, long current);
}
 ```
 进度回调实现
 ```java
 /**
     * 统一处理进度信息
     * @param total    总计大小
     * @param current  当前进度
     * @param callBack
     * @param <T>
     */
    private <T> void progressCallBack(final long total, final long current, final ReqProgressCallBack<T> callBack) {
        okHttpHandler.post(new Runnable() {
            @Override
            public void run() {
                if (callBack != null) {
                    callBack.onProgress(total, current);
                }
            }
        });
    }
 ```
