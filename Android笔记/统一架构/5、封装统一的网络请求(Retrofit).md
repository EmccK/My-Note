# 封装统一的网络请求(Retrofit)

## 1. 简介

|     介绍     |         一个Restful的HTTP网络请求框架（基于Okhttp）          |
| :----------: | :----------------------------------------------------------: |
|   **作者**   |                          **Square**                          |
|   **功能**   | **基于Okhttp & 遵循Restful API设计风格<br />通过注解配置网络请求参数<br />支持同步 & 异步网络请求<br />支持多种数据的解析 & 序列化格式(Gson、Json、XML、Protobuf)<br />提供对RxJava支持** |
|   **优点**   | **功能强大：支持同步 & 异步、支持多种数据的解析 & 序列化格式、支持RxJava<br />简介易用：通过注解配置网络请求参数、采用大量设计模式简化使用<br />可扩展性好：功能模块高度封装、解耦彻底，如自定义Converters等等** |
| **应用场景** |            **任何网络请求的需求场景都应优先选择**            |

[github地址](https://square.github.io/retrofit/)

## 2. 导入依赖

```java
implementation 'com.squareup.okhttp3:okhttp:4.0.1'
implementation 'com.squareup.retrofit2:retrofit:2.6.0'
implementation 'com.squareup.retrofit2:converter-scalars:2.6.0'
    
//可以选择以下Converters    
Gson: com.squareup.retrofit2:converter-gson
Jackson: com.squareup.retrofit2:converter-jackson
Moshi: com.squareup.retrofit2:converter-moshi
Protobuf: com.squareup.retrofit2:converter-protobuf
Wire: com.squareup.retrofit2:converter-wire
Simple XML: com.squareup.retrofit2:converter-simplexml
Scalars: com.squareup.retrofit2:converter-scalars
```

## 3. 添加网络权限

```java
<uses-permission android:name="android.permission.INTERNET"/>
```

## 4. 对Retrofit进行封装

### 1. 首先创建请求方法`RestService`接口

其中有`GET、POST、POST_RAW、PUT、PUT_RAW、DELETE、UPLOAD、DOWNLOAD`请求

```java
public interface RestService {

    @GET
    Call<String> get(@Url String url, @QueryMap Map<String, Object> params);

    @FormUrlEncoded
    @POST
    Call<String> post(@Url String url, @FieldMap Map<String, Object> params);

    @POST
    Call<String> postRaw(@Url String url, @Body RequestBody body);

    @FormUrlEncoded
    @PUT
    Call<String> put(@Url String url, @FieldMap Map<String, Object> params);

    @PUT
    Call<String> putRaw(@Url String url, @Body RequestBody body);

    @DELETE
    Call<String> delete(@Url String url, @QueryMap Map<String, Object> params);

    @Streaming
    @GET
    Call<ResponseBody> download(@Url String url, @QueryMap Map<String, Object> params);

    @Multipart
    @POST
    Call<String> upload(@Url String url, @Part MultipartBody.Part file);
}
```

### 2. 创建`HttpMethod`枚举类

创建`HttpMethod`枚举类，定义网络请求的方式，方便以后判断

```java
public enum HttpMethod {
    GET,
    POST,
    POST_RAW,
    PUT,
    PUT_RAW,
    DELETE,
    UPLOAD
}
```

### 3. Retrofit和Service创建类`RestCreator`

```java
public class RestCreator {

    /**
     * 使用单例模式获取请求的参数
     * 参数容器
     */
    private static final class ParamsHolder {
        private static final HashMap<String, Object> PARAMS = new HashMap<>();
    }

    public static HashMap<String, Object> getParams() {
        return ParamsHolder.PARAMS;
    }

    /**
     * 构建OkHttp
     */
    private static final class OkHttpHolder {
        private static final int TIME_OUT = 60;
        private static final OkHttpClient.Builder BUILDER = new OkHttpClient.Builder();
        //拦截器
		//private static final ArrayList<Interceptor> INTERCEPTORS = Lottery.getConfiguration(ConfigKeys.INTERCEPTOR);

        private static OkHttpClient.Builder addIntercepor() {
            /*if (INTERCEPTORS != null && !INTERCEPTORS.isEmpty()) {
                for (Interceptor interceptor : INTERCEPTORS) {
                    BUILDER.addInterceptor(interceptor);
                }
            }*/
            return BUILDER;
        }

        private static final OkHttpClient OK_HTTP_CLIENT = addIntercepor()
                .connectTimeout(TIME_OUT, TimeUnit.SECONDS)
                .build();
    }

    /**
     * 使用单例模式创建Retrofit
     * RetrofitHolder
     */
    private static final class RetrofitHolder {
        private static final String BASE_URL = Lottery.getConfiguration(ConfigKeys.API_HOST);
        private static final Retrofit RETROFIT_CLIENT = new Retrofit.Builder()
                .baseUrl(BASE_URL)
                .client(OkHttpHolder.OK_HTTP_CLIENT)
                .addConverterFactory(ScalarsConverterFactory.create())
                .build();
    }

    /**
     * RestService接口
     */
    private static final class RestServiceHolder {
        private static final RestService REST_SERVICE =
                RetrofitHolder.RETROFIT_CLIENT.create(RestService.class);
    }

    /**
     * 使用单例模式创建Retrofit
     * 获取RestService接口
     */
    public static RestService getRestService() {
        return RestServiceHolder.REST_SERVICE;
    }
}
```

### 4. 定义网络请求的回调`RestCallback`

```java
public class RestCallback {

    /**
     * 请求成功
     */
    public interface ISuccess {
        void onSuccess(String response);
    }

    /**
     * 请求错误
     */
    public interface IError {
        void onError(int code, String msg);
    }

    /**
     * 请求失败
     */
    public interface IFailure {
        void onFailure();
    }

    /**
     * 请求方法，执行请求开始之前和请求完成以后的操作
     */
    public interface IRequest {
        void onRequestStart();

        void onRequestEnd();
    }
}
```

### 5. RestClient的建造者`RestClientBuilder`

```java
public class RestClientBuilder {

    private String mUrl = null;
    private static final Map<String, Object> PARAMS = RestCreator.getParams();
    private RestCallback.IRequest mIRequest = null;
    private RestCallback.ISuccess mISuccess = null;
    private RestCallback.IFailure mIFailure = null;
    private RestCallback.IError mIError = null;
    private RequestBody mBody = null;
    //文件上传的参数
    private File mFile = null;
    //文件下载的参数
    private String mDownloadDir = null;
    private String mExtension = null;
    private String mName = null;
    //Loader参数
    private Context mContext = null;
    private LoaderStyle mLoaderStyle = null;
    @ColorInt
    private int mColor;

    //不允许外部的类去直接New这个类，只允许RestClient去new它
    RestClientBuilder() {
    }

    //设置Url
    public final RestClientBuilder url(String url) {
        this.mUrl = url;
        return this;
    }

    //添加请求参数，传入map
    public final RestClientBuilder params(HashMap<String, Object> params) {
        PARAMS.putAll(params);
        return this;
    }

    //重载，传入键值对
    public final RestClientBuilder params(String key, Object value) {
        PARAMS.put(key, value);
        return this;
    }

    //上传的文件
    public final RestClientBuilder file(File file) {
        this.mFile = file;
        return this;
    }

    //要上传文件的路径
    public final RestClientBuilder file(String filePath) {
        this.mFile = new File(filePath);
        return this;
    }

    //所下载文件存放的目录
    public final RestClientBuilder dir(String downloadDir) {
        this.mDownloadDir = downloadDir;
        return this;
    }

    //所下载文件的后缀名
    public final RestClientBuilder extension(String extension) {
        this.mExtension = extension;
        return this;
    }

    //所下载文件的文件名
    public final RestClientBuilder name(String name) {
        this.mName = name;
        return this;
    }

    //如果传入的是原始数据(json数据)
    public final RestClientBuilder jsonRaw(String raw) {
        this.mBody = RequestBody.create(MediaType.parse("application/json;charset=UTF-8"), raw);
        return this;
    }

    //网络请求成功的回调
    public final RestClientBuilder onRequest(RestCallback.IRequest iRequest) {
        this.mIRequest = iRequest;
        return this;
    }

    //网络请求成功的回调
    public final RestClientBuilder success(RestCallback.ISuccess iSuccess) {
        this.mISuccess = iSuccess;
        return this;
    }

    //网络请求失败的回调
    public final RestClientBuilder failure(RestCallback.IFailure iFailure) {
        this.mIFailure = iFailure;
        return this;
    }

    //网络请求错误的回调
    public final RestClientBuilder error(RestCallback.IError iError) {
        this.mIError = iError;
        return this;
    }

    public final RestClientBuilder loader(Context context, LoaderStyle style, @ColorInt int color) {
        this.mContext = context;
        this.mLoaderStyle = style;
        this.mColor = color;
        return this;
    }

    public final RestClientBuilder loader(Context context, LoaderStyle style) {
        this.mContext = context;
        this.mLoaderStyle = style;
        this.mColor = Color.WHITE;
        return this;
    }

    public final RestClientBuilder loader(Context context, @ColorInt int color) {
        this.mContext = context;
        this.mLoaderStyle = LoaderStyle.BallClipRotatePulseIndicator;
        this.mColor = color;
        return this;
    }

    public final RestClientBuilder loader(Context context) {
        this.mContext = context;
        this.mLoaderStyle = LoaderStyle.BallClipRotatePulseIndicator;
        this.mColor = Color.WHITE;
        return this;
    }

    //创建出RestClient
    public final RestClient build() {
        return new RestClient(
                mUrl, PARAMS, mIRequest, mISuccess,
                mIFailure, mIError, mBody, mFile,
                mDownloadDir, mExtension, mName,
                mContext, mLoaderStyle, mColor
        );
    }
}
```

### 6. 创建Retrofit需要的Callback，将自定义的callback添加进去

```java
public class RequestCallbacks implements Callback<String> {

    private final RestCallback.IRequest REQUEST;
    private final RestCallback.ISuccess SUCCESS;
    private final RestCallback.IFailure FAILURE;
    private final RestCallback.IError ERROR;
    private final LoaderStyle LOADER_STYLE;
    private static final Handler HANDLER = Lottery.getHandler();

    public RequestCallbacks(
            RestCallback.IRequest request,
            RestCallback.ISuccess success,
            RestCallback.IFailure failure,
            RestCallback.IError error,
            LoaderStyle loaderStyle
    ) {
        this.REQUEST = request;
        this.SUCCESS = success;
        this.FAILURE = failure;
        this.ERROR = error;
        this.LOADER_STYLE = loaderStyle;
    }

    @Override
    public void onResponse(Call<String> call, Response<String> response) {
        //请求成功
        if (response.isSuccessful()) {
            //call已经执行了
            if (call.isExecuted()) {
                if (SUCCESS != null) {
                    SUCCESS.onSuccess(response.body());
                }
            }
        } else {
            if (ERROR != null) {
                ERROR.onError(response.code(), response.message());
            }
        }

        onRequestFinish();
    }

    @Override
    public void onFailure(Call<String> call, Throwable t) {
        if (FAILURE != null) {
            FAILURE.onFailure();
        }
        if (REQUEST != null) {
            REQUEST.onRequestEnd();
        }
        onRequestFinish();
    }

    private void onRequestFinish() {
        final long delayed = (long) Lottery.getConfigurations().get(ConfigKeys.LOADER_DELAYED);
        if (LOADER_STYLE != null) {
            HANDLER.postDelayed(new Runnable() {
                @Override
                public void run() {
                    LotteryLoader.stopLoading();
                }
            }, delayed);
        }
    }
}
```

### 7. 下载请求中用到的一些文件操作方法

- 文件操作的一些静态方法

  `FileUtil`类

  ```java
  public final class FileUtil {
  
      //格式化的模板
      private static final String TIME_FORMAT = "_yyyyMMdd_HHmmss";
  
      private static final String SDCARD_DIR =
              Environment.getExternalStorageDirectory().getPath();
  
      //默认本地上传图片目录
      public static final String UPLOAD_PHOTO_DIR =
              Environment.getExternalStorageDirectory().getPath() + "/a_upload_photos/";
  
      //网页缓存地址
      public static final String WEB_CACHE_DIR =
              Environment.getExternalStorageDirectory().getPath() + "/app_web_cache/";
  
      //系统相机目录
      public static final String CAMERA_PHOTO_DIR =
              Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DCIM).getPath() + "/Camera/";
  
      private static String getTimeFormatName(String timeFormatHeader) {
          final Date date = new Date(System.currentTimeMillis());
          //必须要加上单引号
          final SimpleDateFormat dateFormat = new SimpleDateFormat("'" + timeFormatHeader + "'" + TIME_FORMAT, Locale.getDefault());
          return dateFormat.format(date);
      }
  
      /**
       * @param timeFormatHeader 格式化的头(除去时间部分)
       * @param extension        后缀名
       * @return 返回时间格式化后的文件名
       */
      public static String getFileNameByTime(String timeFormatHeader, String extension) {
          return getTimeFormatName(timeFormatHeader) + "." + extension;
      }
  
      @SuppressWarnings("ResultOfMethodCallIgnored")
      private static File createDir(String sdcardDirName) {
          //拼接成SD卡中完整的dir
          final String dir = SDCARD_DIR + "/" + sdcardDirName + "/";
          final File fileDir = new File(dir);
          if (!fileDir.exists()) {
              fileDir.mkdirs();
          }
          return fileDir;
      }
  
      @SuppressWarnings("ResultOfMethodCallIgnored")
      public static File createFile(String sdcardDirName, String fileName) {
          return new File(createDir(sdcardDirName), fileName);
      }
  
      private static File createFileByTime(String sdcardDirName, String timeFormatHeader, String extension) {
          final String fileName = getFileNameByTime(timeFormatHeader, extension);
          return createFile(sdcardDirName, fileName);
      }
  
      //获取文件的MIME
      public static String getMimeType(String filePath) {
          final String extension = getExtension(filePath);
          return MimeTypeMap.getSingleton().getMimeTypeFromExtension(extension);
      }
  
      //获取文件的后缀名
      public static String getExtension(String filePath) {
          String suffix = "";
          final File file = new File(filePath);
          final String name = file.getName();
          final int idx = name.lastIndexOf('.');
          if (idx > 0) {
              suffix = name.substring(idx + 1);
          }
          return suffix;
      }
  
      /**
       * 保存Bitmap到SD卡中
       *
       * @param dir      目录名,只需要写自己的相对目录名即可
       * @param compress 压缩比例 100是不压缩,值约小压缩率越高
       * @return 返回该文件
       */
      public static File saveBitmap(Bitmap mBitmap, String dir, int compress) {
  
          final String sdStatus = Environment.getExternalStorageState();
          // 检测sd是否可用
          if (!sdStatus.equals(Environment.MEDIA_MOUNTED)) {
              return null;
          }
          FileOutputStream fos = null;
          BufferedOutputStream bos = null;
          File fileName = createFileByTime(dir, "DOWN_LOAD", "jpg");
          try {
              fos = new FileOutputStream(fileName);
              bos = new BufferedOutputStream(fos);
              mBitmap.compress(Bitmap.CompressFormat.JPEG, compress, bos);// 把数据写入文件
          } catch (FileNotFoundException e) {
              e.printStackTrace();
          } finally {
              try {
  
                  if (bos != null) {
                      bos.flush();
                  }
                  if (bos != null) {
                      bos.close();
                  }
                  //关闭流
                  if (fos != null) {
                      fos.flush();
                  }
                  if (fos != null) {
                      fos.close();
                  }
              } catch (IOException e) {
                  e.printStackTrace();
              }
          }
          refreshDCIM();
  
          return fileName;
      }
  
      public static File writeToDisk(InputStream is, String dir, String name) {
          final File file = FileUtil.createFile(dir, name);
          BufferedInputStream bis = null;
          FileOutputStream fos = null;
          BufferedOutputStream bos = null;
  
          try {
              bis = new BufferedInputStream(is);
              fos = new FileOutputStream(file);
              bos = new BufferedOutputStream(fos);
  
              byte data[] = new byte[1024 * 4];
  
              int count;
              while ((count = bis.read(data)) != -1) {
                  bos.write(data, 0, count);
              }
  
              bos.flush();
              fos.flush();
  
  
          } catch (IOException e) {
              e.printStackTrace();
          } finally {
              try {
                  if (bos != null) {
                      bos.close();
                  }
                  if (fos != null) {
                      fos.close();
                  }
                  if (bis != null) {
                      bis.close();
                  }
                  is.close();
              } catch (IOException e) {
                  e.printStackTrace();
              }
          }
  
          return file;
      }
  
      public static File writeToDisk(InputStream is, String dir, String prefix, String extension) {
          final File file = FileUtil.createFileByTime(dir, prefix, extension);
          BufferedInputStream bis = null;
          FileOutputStream fos = null;
          BufferedOutputStream bos = null;
  
          try {
              bis = new BufferedInputStream(is);
              fos = new FileOutputStream(file);
              bos = new BufferedOutputStream(fos);
  
              byte data[] = new byte[1024 * 4];
  
              int count;
              while ((count = bis.read(data)) != -1) {
                  bos.write(data, 0, count);
              }
  
              bos.flush();
              fos.flush();
  
  
          } catch (IOException e) {
              e.printStackTrace();
          } finally {
              try {
                  if (bos != null) {
                      bos.close();
                  }
                  if (fos != null) {
                      fos.close();
                  }
                  if (bis != null) {
                      bis.close();
                  }
                  is.close();
              } catch (IOException e) {
                  e.printStackTrace();
              }
          }
  
          return file;
      }
  
      /**
       * 通知系统刷新系统相册，使照片展现出来
       */
      private static void refreshDCIM() {
          if (Build.VERSION.SDK_INT >= 19) {
              //兼容android4.4版本，只扫描存放照片的目录
              MediaScannerConnection.scanFile(Lottery.getApplicationContext(),
                      new String[]{Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DCIM).getPath()},
                      null, null);
          } else {
              //扫描整个SD卡来更新系统图库，当文件很多时用户体验不佳，且不适合4.4以上版本
              Lottery.getApplicationContext().sendBroadcast(new Intent(Intent.ACTION_MEDIA_MOUNTED, Uri.parse("file://" +
                      Environment.getExternalStorageDirectory())));
          }
      }
  
      /**
       * 读取raw目录中的文件,并返回为字符串
       */
      public static String getRawFile(int id) {
          final InputStream is = Lottery.getApplicationContext().getResources().openRawResource(id);
          final BufferedInputStream bis = new BufferedInputStream(is);
          final InputStreamReader isr = new InputStreamReader(bis);
          final BufferedReader br = new BufferedReader(isr);
          final StringBuilder stringBuilder = new StringBuilder();
          String str;
          try {
              while ((str = br.readLine()) != null) {
                  stringBuilder.append(str);
              }
          } catch (IOException e) {
              e.printStackTrace();
          } finally {
              try {
                  br.close();
                  isr.close();
                  bis.close();
                  is.close();
              } catch (IOException e) {
                  e.printStackTrace();
              }
          }
          return stringBuilder.toString();
      }
  
  
      public static void setIconFont(String path, TextView textView) {
          final Typeface typeface = Typeface.createFromAsset(Lottery.getApplicationContext().getAssets(), path);
          textView.setTypeface(typeface);
      }
  
      /**
       * 读取assets目录下的文件,并返回字符串
       */
      public static String getAssetsFile(String name) {
          InputStream is = null;
          BufferedInputStream bis = null;
          InputStreamReader isr = null;
          BufferedReader br = null;
          StringBuilder stringBuilder = null;
          final AssetManager assetManager = Lottery.getApplicationContext().getAssets();
          try {
              is = assetManager.open(name);
              bis = new BufferedInputStream(is);
              isr = new InputStreamReader(bis);
              br = new BufferedReader(isr);
              stringBuilder = new StringBuilder();
              String str;
              while ((str = br.readLine()) != null) {
                  stringBuilder.append(str);
              }
          } catch (IOException e) {
              e.printStackTrace();
          } finally {
              try {
                  if (br != null) {
                      br.close();
                  }
                  if (isr != null) {
                      isr.close();
                  }
                  if (bis != null) {
                      bis.close();
                  }
                  if (is != null) {
                      is.close();
                  }
                  assetManager.close();
              } catch (IOException e) {
                  e.printStackTrace();
              }
          }
          if (stringBuilder != null) {
              return stringBuilder.toString();
          } else {
              return null;
          }
      }
  
      public static String getRealFilePath(final Context context, final Uri uri) {
          if (null == uri) return null;
          final String scheme = uri.getScheme();
          String data = null;
          if (scheme == null)
              data = uri.getPath();
          else if (ContentResolver.SCHEME_FILE.equals(scheme)) {
              data = uri.getPath();
          } else if (ContentResolver.SCHEME_CONTENT.equals(scheme)) {
              final Cursor cursor = context.getContentResolver().query(uri, new String[]{MediaStore.Images.ImageColumns.DATA}, null, null, null);
              if (null != cursor) {
                  if (cursor.moveToFirst()) {
                      final int index = cursor.getColumnIndex(MediaStore.Images.ImageColumns.DATA);
                      if (index > -1) {
                          data = cursor.getString(index);
                      }
                  }
                  cursor.close();
              }
          }
          return data;
      }
  }
  ```

- `DownloadHandler`类

  ```java
  public class DownloadHandler {
  
      private final String URL;
      private static final HashMap<String, Object> PARAMS = RestCreator.getParams();
      private final RestCallback.IRequest REQUEST;
      private final RestCallback.ISuccess SUCCESS;
      private final RestCallback.IFailure FAILURE;
      private final RestCallback.IError ERROR;
      private final String DOWNLOAD_DIR;
      private final String EXTENSION;
      private final String NAME;
  
      public DownloadHandler(String url,
                             RestCallback.IRequest request,
                             RestCallback.ISuccess success,
                             RestCallback.IFailure failure,
                             RestCallback.IError error,
                             String downloadDir,
                             String extension,
                             String name
      ) {
          this.URL = url;
          this.REQUEST = request;
          this.SUCCESS = success;
          this.FAILURE = failure;
          this.ERROR = error;
          this.DOWNLOAD_DIR = downloadDir;
          this.EXTENSION = extension;
          this.NAME = name;
      }
  
      public final void handleDownload() {
          if (REQUEST != null) {
              REQUEST.onRequestStart();
          }
          RestCreator
                  .getRestService()
                  .download(URL, PARAMS)
                  .enqueue(new Callback<ResponseBody>() {
                      @Override
                      public void onResponse(Call<ResponseBody> call, Response<ResponseBody> response) {
                          if (response.isSuccessful()) {
                              final ResponseBody responseBody = response.body();
                              final SaveFileTask task = new SaveFileTask(REQUEST, SUCCESS, FAILURE, ERROR);
                              //以线程池的形式执行
                              task.executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR, DOWNLOAD_DIR, EXTENSION, NAME, responseBody);
                              //这里一定要注意判断，否则文件下载不全
                              if (task.isCancelled()) {
                                  if (REQUEST != null) {
                                      REQUEST.onRequestEnd();
                                  }
                              }
                          } else {
                              if (ERROR != null) {
                                  ERROR.onError(response.code(), response.message());
                              }
                          }
                          RestCreator.getParams().clear();
                      }
  
                      @Override
                      public void onFailure(Call<ResponseBody> call, Throwable t) {
                          if (FAILURE != null) {
                              FAILURE.onFailure();
                          }
                          RestCreator.getParams().clear();
                      }
                  });
      }
  }
  ```

- `SaveFileTask`类

  ```java
  public class SaveFileTask extends AsyncTask<Object, Void, File> {
  
      private final RestCallback.IRequest REQUEST;
      private final RestCallback.ISuccess SUCCESS;
      private final RestCallback.IFailure FAILURE;
      private final RestCallback.IError ERROR;
  
      public SaveFileTask(RestCallback.IRequest request,
                          RestCallback.ISuccess success,
                          RestCallback.IFailure failure,
                          RestCallback.IError error
      ) {
          this.REQUEST = request;
          this.SUCCESS = success;
          this.FAILURE = failure;
          this.ERROR = error;
      }
  
      @Override
      protected File doInBackground(Object... objects) {
          String downloadDir = (String) objects[0];
          String extension = (String) objects[1];
          final String name = (String) objects[2];
          final ResponseBody body = (ResponseBody) objects[3];
          final InputStream is = body.byteStream();
          if (downloadDir == null || downloadDir.equals("")) {
              downloadDir = "down_loads";
          }
          if (extension == null || extension.equals("")) {
              extension = "";
          }
          if (name == null || name.equals("")) {
              return FileUtil.writeToDisk(is, downloadDir, extension.toUpperCase(), extension);
          } else {
              return FileUtil.writeToDisk(is, downloadDir, name);
          }
      }
  
      @Override
      protected void onPostExecute(File file) {
          super.onPostExecute(file);
          if (SUCCESS != null) {
              SUCCESS.onSuccess(file.getPath());
          }
          if (REQUEST != null) {
              //执行请求结束后的操作
              REQUEST.onRequestEnd();
          }
          autoInstallApk(file);
      }
  
      /**
       * 自动安装安装包
       */
      private void autoInstallApk(File file) {
          if (FileUtil.getExtension(file.getPath()).equals("apk")) {
              final Intent install = new Intent();
              install.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
              install.setAction(Intent.ACTION_VIEW);
              install.setDataAndType(Uri.fromFile(file), "application/vnd.android.package-archive");
              Lottery.getApplicationContext().startActivity(install);
          }
      }
  }
  ```

### 8. 实现RestClient

```java
public class RestClient {

    //创建RestClient需要的一些参数
    private final String URL;
    //请求参数
    private static final HashMap<String, Object> PARAMS = RestCreator.getParams();
    //请求回调
    private final RestCallback.IRequest REQUEST;
    private final RestCallback.ISuccess SUCCESS;
    private final RestCallback.IFailure FAILURE;
    private final RestCallback.IError ERROR;
    private final RequestBody BODY;
    //文件上传参数
    private final File FILE;
    //文件下载参数
    private final String DOWNLOAD_DIR;
    private final String EXTENSION;
    private final String NAME;
    //Loader参数
    private final LoaderStyle LOADER_STYLE;
    private final Context CONTEXT;
    @ColorInt
    private final int COLOR;

    public RestClient(String url,
                      Map<String, Object> params,
                      RestCallback.IRequest request,
                      RestCallback.ISuccess success,
                      RestCallback.IFailure failure,
                      RestCallback.IError error,
                      RequestBody body,
                      File file,
                      String downloadDir,
                      String extension,
                      String name,
                      Context context,
                      LoaderStyle loaderStyle,
                      @ColorInt int color) {
        this.URL = url;
        PARAMS.putAll(params);
        this.REQUEST = request;
        this.SUCCESS = success;
        this.FAILURE = failure;
        this.ERROR = error;
        this.BODY = body;
        this.FILE = file;
        this.DOWNLOAD_DIR = downloadDir;
        this.EXTENSION = extension;
        this.NAME = name;
        this.CONTEXT = context;
        this.LOADER_STYLE = loaderStyle;
        this.COLOR = color;
    }

    //建造者模式构建RestClient
    public static RestClientBuilder builder() {
        return new RestClientBuilder();
    }

    //具体的请求，根据method判断
    private void request(HttpMethod method) {
        final RestService service = RestCreator.getRestService();
        Call<String> call = null;

        if (REQUEST != null) {
            REQUEST.onRequestStart();
        }

        if (LOADER_STYLE != null) {
            LotteryLoader.showLoading(CONTEXT, LOADER_STYLE, COLOR);
        }

        switch (method) {
            case GET:
                call = service.get(URL, PARAMS);
                break;
            case POST:
                call = service.post(URL, PARAMS);
                break;
            case POST_RAW:
                call = service.postRaw(URL, BODY);
                break;
            case PUT:
                call = service.put(URL, PARAMS);
                break;
            case PUT_RAW:
                call = service.putRaw(URL, BODY);
                break;
            case DELETE:
                call = service.delete(URL, PARAMS);
                break;
            case UPLOAD:
                //文件上传
                final RequestBody requestBody =
                        RequestBody.create(MediaType.parse(MultipartBody.FORM.toString()), FILE);
                final MultipartBody.Part body =
                        MultipartBody.Part.createFormData("file", FILE.getName(), requestBody);
                call = RestCreator.getRestService().upload(URL, body);
                break;
            default:
                break;
        }

        if (call != null) {
            call.enqueue(getRequestCallback());
        }
    }

    private Callback<String> getRequestCallback() {
        return new RequestCallbacks(
                REQUEST,
                SUCCESS,
                FAILURE,
                ERROR,
                LOADER_STYLE
        );
    }

    //GET请求
    public final void get() {
        request(HttpMethod.GET);
    }

    //GET请求
    public final void post() {
        if (BODY == null) {
            request(HttpMethod.POST);
        } else {
            if (!PARAMS.isEmpty()) {
                throw new RuntimeException("Params must be null!");
            }
            request(HttpMethod.POST_RAW);
        }
    }

    //GET请求
    public final void put() {
        if (BODY == null) {
            request(HttpMethod.PUT);
        } else {
            if (!PARAMS.isEmpty()) {
                throw new RuntimeException("Params must be null!");
            }
            request(HttpMethod.PUT_RAW);
        }

    }

    //GET请求
    public final void delete() {
        request(HttpMethod.DELETE);
    }

    //文件上传
    public final void upload() {
        request(HttpMethod.UPLOAD);
    }

    //文件下载
    public final void download() {
        new DownloadHandler(
                URL, REQUEST, SUCCESS, FAILURE,
                ERROR, DOWNLOAD_DIR, EXTENSION, NAME
        ).handleDownload();
    }
}
```

## 5. 使用RestClient进行网络请求

```java
RestClient.builder()
        //如果这里是完整的地址，那么Retrofit中的baseUrl就没有作用
        //如果这里不是完整地址，请求地址为baseUrl+url
        .url("http://10.0.2.2/Test/user_profile.json")
        //请求参数
        .params("name", "dankai")
        //请求成功回调
        .success(new RestCallback.ISuccess() {
            @Override
            public void onSuccess(String response) {
                //请求成功后的操作
            }
        })
        //请求错误回调
        .error(new RestCallback.IError() {
            @Override
            public void onError(int code, String msg) {
                //请求错误后的操作
                Toast.makeText(MainActivity.this, "code:" + code + "message" + msg, Toast.LENGTH_SHORT).show();
            }
        })
        //请求失败回调
        .failure(new RestCallback.IFailure() {
            @Override
            public void onFailure() {
                //请求失败后的操作
                Toast.makeText(MainActivity.this, "失败", Toast.LENGTH_SHORT).show();
            }
        })
        .build()
        .get();
```