> Retrofit 是一个 RESTful 的 HTTP 网络请求框架的封装。
> 网络请求的工作本质上是 OkHttp 完成，而 Retrofit 仅负责 网络请求接口的封装

Retrofit 在使用时其实就充当了一个适配器（Adapter）的角色，主要是将一个 Java 接口翻译成一个 HTTP 请求对象，然后用 OkHttp 去发送这个请求
**核心思想**：动态代理—通俗来讲，就是你要执行某个操作的前后需要增加一些操作
Retrofit 主要定义了 4 个接口：

> Callback<T>：请求数据的返回；
> Converter<F, T>：对返回数据进行解析，一般用 GSON ；
> Call<T>：发送请求，Retrofit 默认的实现是 OkHttpCall<T>，也可以依需自定义 Call<T>；
> CallAdapter<T>：将 Call 对象转换成其他对象，如转换成支持 RxJava 的 Observable对象

所谓同步方式，是指我们发出网络请求之后当前线程被阻塞，直到请求的结果（成功或者失败）到来，才继续向下执行。所谓异步，是指我们的网络请求发出之后，不必等待请求结果的到来，就可以去做其他的事情，当请求结果到来时，我们在做处理结果的动作

## retrofit的简单使用（异步）
### 1、创建描述网络请求的接口(get请求)
```java
//采用 注解 描述 网络请求参数
public interface GetRequestInterface {

    // 注解里传入 网络请求 的部分URL地址
    // Retrofit把网络请求的URL分成了两部分：一部分放在Retrofit对象里，另一部分放在网络请求接口里
    // 如果接口里的url是一个完整的网址，那么放在Retrofit对象里的URL可以忽略
    // getCall()是接受网络请求数据的方法

    @GET("ajax.php?a=fy&f=auto&t=auto&w=你好")
    Call<Translation> getCall();
}
```

### 2、创建Retrofit对象——创建 网络请求接口 的实例——发送网络请求（异步）
```java
public class MainActivity extends AppCompatActivity {

    private static final String TAG = "MainActivity";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        findViewById(R.id.trans).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                request();
            }
        });

    }

    // 使用Retrofit封装的方法
    private void request() {

        //步骤4:创建Retrofit对象
        Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("http://fy.iciba.com/") 
                .addConverterFactory(GsonConverterFactory.create())
                .build();

        // 步骤5:创建 网络请求接口 的实例
        GetRequestInterface request = retrofit.create(GetRequestInterface.class);
        //获取call对象用来发送请求
        Call<Translation> call = request.getCall();

        //步骤6:发送网络请求(异步)
        call.enqueue(new Callback<Translation>() {
            @Override
            public void onResponse(Call<Translation> call, Response<Translation> response) {
                Log.e(TAG, "onResponse: "+response.body() );

                Translation translation = response.body();
                translation.show();
            }

            @Override
            public void onFailure(Call<Translation> call, Throwable t) {

            }
        });
    }
}
```

其中Translation是一个自定义的用来接收服务器数据的类
```java
public class Translation {

    private String status;
    private content content;
    private static class content{
        private String from;
        private String to;
        private String vendor;
        private String out;
        private int errNo;
    }

    public void show() {
        System.out.println(status);
        System.out.println(content.from);
        System.out.println(content.to);
        System.out.println(content.vendor);
        System.out.println(content.out);
        System.out.println(content.errNo);
    }

}
```
#### 再列举其他的情况：参数不定
```java
http://api.stay4it.com/News?newsId=1&type=类型1…
http://api.stay4it.com/News?newsId={资讯id}&type={类型1}&type={类型2}…
```
接口定义
```java
@GET("News")
Call<NewsBean> getItem(@QueryMap Map<String, String> map);

或者

@GET("News")
Call<NewsBean> getItem(
             @Query("newsId") String newsId，
             @QueryMap Map<String, String> map);

```

## post请求
> POST 一个字段
```java
@POST("mobile/register")
Call<ResponseBody> registerDevice(@Field("id") String registerid);
```
@Field声明字段的key

> POST 两个及两个以上的字段
```java
http://xxx/api/Comments

@FormUrlEncoded
@POST("Comments/{newsId}")
Call<Comment> reportComment(
        @Path("newsId") String commentId,
        @Field("reason") String reason)
```
@FormUrlEncoded表示请求发送编码表单数据，每个键值对需要使用@Field注解

> POST URL不完整，需要补全，使用@Path
```java
http://xxx/api/Comments/1?access_token=1234123
http://xxx/api/Comments/{newsId}?access_token={access_token}

@FormUrlEncoded
@POST("Comments/{newsId}")
Call<Comment> reportComment(
        @Path("newsId") String commentId,
        @Query("access_token") String access_token,
        @Field("reason") String reason);
```


> POST请求发送 对象（JSON）
```java
@POST("mobile/register")
    Call register1(@Body RegisterPost post);
```java
@Body声明对象，Retrofit会自动序列化成JSON，序列化使用的库，通过：
.addConverterFactory(GsonConverterFactory.create())声明。

> POST 文件（图片，MP3，等等）
```java
public interface FileUploadService {
 @Multipart
 @POST("upload")
 Call<ResponseBody> upload(@Part("description") RequestBody description,
                          @Part MultipartBody.Part file);
}
```
而他的请求写法也有点不一样：
```java
// 创建 RequestBody，用于封装构建RequestBody
RequestBody requestFile = RequestBody.create(MediaType.parse("multipart/form-data"), file);
// MultipartBody.Part  和后端约定好Key，这里的partName是用image
MultipartBody.Part body = MultipartBody.Part.createFormData("image", file.getName(), requestFile);
// 添加描述
String descriptionString = "hello, 这是文件描述";
RequestBody description = RequestBody.create(MediaType.parse("multipart/form-data"), descriptionString);
// 执行请求
Call<ResponseBody> call = service.upload(description, body);
call.enqueue(new Callback<ResponseBody>() {
    @Override
    public void onResponse(Call<ResponseBody> call,
                           Response<ResponseBody> response) {
        Log.v("Upload", "success");
    }

    @Override
    public void onFailure(Call<ResponseBody> call, Throwable t) {
        Log.e("Upload error:", t.getMessage());
      }
    });
}
```

文件带进度的文件上传
https://blog.csdn.net/k_bb_666/article/details/79612555
文件下载
https://blog.csdn.net/shusheng0007/article/details/82428733
