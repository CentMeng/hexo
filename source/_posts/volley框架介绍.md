---
title: volley框架介绍
date: 2016-08-11 21:44:07
categories:
- Android技术
tags:
- 框架
---
<img src="/img/code.jpg" />

### volley介绍
开发android应用很多时候都要涉及网络操作，Android SDK中提供了HttpClient 和 HttpUrlConnection两种方式用来处理网络操作，但当应用比较复杂的时候需要我们编写大量的代码处理很多东西：图像缓存，请求的调度等等；

而Volley框架就是为解决这些而生的，它与2013年Google I/O大会上被提出：使得Android应用网络操作更方便更快捷；抽象了底层Http Client等实现的细节，让开发者更专注与产生RESTful Request。另外，Volley在不同的线程上异步执行所有请求而避免了阻塞主线程。

目前的HTTP通信框架，AsyncHttpClient，网络图片加载框架，Universal-Image-Loader，而Volley是将以上两个框架集于一身。

<img src="/img/android/volley/1.jpg" />

### volley下载

```
git clone https://android.googlesource.com/platform/frameworks/volley  

```
或者使用我的[地址](https://github.com/CentMeng/volley)，很久没有更新了，请见谅,解决了SSL证书问题以及做了简单的缓存

### volley特点

- 自动调度网络请求
- 多个并发的网络连接
- 通过使用标准的HTTP缓存机制保持磁盘和内存响应的一致
- 支持请求优先级
- 支持取消请求的强大API，可以取消单个请求或多个
- 易于定制
- 健壮性：便于正确的更新UI和获取数据
- 包含调试和追踪工具

### volley使用

我一般都是使用封装好的方法
#### 创建volleyHttpClient对象
封装了返回Gson和String的get,post请求以及相应需要增加oauth验证的方法

```

public class VolleyHttpClient {

    public static final int GET = 1;

    public static final int GET_AOUTH = 2;

    public static final int POST = 3;

    public static final int POST_AOUTH = 4;

    public static final int GETSTRING = 5;

    public static final int GETSTRING_AOUTH = 6;

    public static final int POSTSTRING_AOUTH = 7;

    private static final String TAG = "VolleyHttpClient";

    private static HttpService httpService;
    private final static Gson gson = new Gson();

    private static BaseActivity activity;


    public VolleyHttpClient(HttpService httpService) {
        this.httpService = httpService;
    }

    /**
     * 传入初始化好的httpService 实例
     *
     * @param httpService
     * @return
     */
    public static VolleyHttpClient newInstance(HttpService httpService, Context context) {
        activity = (BaseActivity) context;
        if (httpService != null) {
            return new VolleyHttpClient(httpService);
        }
        return null;
    }

    public void post(ApiRequest apiRequest) {
        doNetTask(POST, apiRequest, DataCacheType.NO_CACHE);
    }

    public void postWithOAuth(ApiRequest apiRequest) {
        doNetTask(POST_AOUTH, apiRequest, DataCacheType.NO_CACHE);
    }

    public void get(ApiRequest apiRequest) {
        doNetTask(GET, apiRequest, DataCacheType.NO_CACHE);
    }

    public void getWithOAuth(ApiRequest apiRequest) {
        doNetTask(GET_AOUTH, apiRequest, DataCacheType.NO_CACHE);
    }

    public void doNetTask(int method, ApiRequest apiRequest) {
        doNetTask(method, apiRequest, DataCacheType.NO_CACHE);
    }

    public void doNetTask(int method, ApiRequest apiRequest, DataCacheType cacheType) {
        doNetTask(method, apiRequest, cacheType, false, null);
    }


    public void doNetTask(int method, ApiRequest apiRequest, DataCacheType cacheType,
                          boolean needLogin, BaseActivity activity) {
        doNetTask(method, apiRequest, cacheType, needLogin, activity, false);
    }


    public void doNetTask(int method, ApiRequest apiRequest, DataCacheType cacheType,
                          boolean needLogin, BaseActivity activity, boolean needFinish) {
        if (needLogin && activity.app.getUserParam() == null) {
            try {
                activity.cancelLoadingDialog();
            } catch (Exception e) {
                e.printStackTrace();
            }
            Intent intent = new Intent(activity, LoginActivity_.class);
            activity.startActivity(intent);
            if (needFinish) {
                activity.finish();
            }
            return;
        }
        switch (method) {
            case GET:
                //USER_OLD_CACHE有缓存先加载缓存数据，然后网络请求只是保存数据
                //TEMP_CACHE有缓存先加载缓存数据，不进行网络请求
                get(apiRequest, false, cacheType);
                break;
            case GET_AOUTH:
                get(apiRequest, true, cacheType);
                break;
            case POST:
                post(apiRequest, false, cacheType);
                break;
            case POST_AOUTH:
                post(apiRequest, true, cacheType);
                break;
            case GETSTRING:
                getByStringRequest(apiRequest, false, cacheType);
                break;
            case GETSTRING_AOUTH:
                getByStringRequest(apiRequest, true, cacheType);
                break;
            case POSTSTRING_AOUTH:
                postByStringRequest(apiRequest, true, cacheType);
                break;
        }
    }


    /**
     * 不用传header以及accesstoken的GET请求
     *
     * @param apiRequest
     */
    public void get(ApiRequest apiRequest, boolean needOauth, DataCacheType cacheType) {


        Map<String, String> header = new HashMap<String, String>();
        if (apiRequest.getHeaders() != null) {
            header.putAll(apiRequest.getHeaders());
        }
        header.put("code", ApiSettings.VERSION_CODE);
        header.put("osType", ApiSettings.OSTYPE);
        if (needOauth) {

            String accessToken = GlobalPhone.token;

            if(TextUtils.isEmpty(accessToken)){
                accessToken = DataCache.getDataCache()
                        .queryCache("access_token");
            }
            if (!TextUtils.isEmpty(accessToken)) {
                LogUtils.e("access_token", accessToken);
            }
//        header.put("Authorization", "Bearer" + " " + accessToken);
            header.put("token", accessToken);
        }
        String url = apiRequest.getUrl(Method.GET);
        Listener listener = apiRequest.getListener();
        LogUtils.e("------cacheType---------", cacheType.name());
        switch (cacheType) {
            case USE_OLD_CACHE:
            case TEMP_CACHE:
                String cacheStr ="";
                if(cacheType == DataCacheType.TEMP_CACHE){
                    cacheStr = DataCache.getDataCache().queryTempCache(url);
                }else {
                    cacheStr = DataCache.getDataCache().queryCache(url);
                }
                if (!TextUtils.isEmpty(cacheStr)) {
                    try {
                        if (apiRequest.getResponseType() != null) {
                            //有缓存先加载缓存数据
                            listener.onResponse(gson.fromJson(cacheStr, apiRequest.getResponseType()));
                            //有缓存先加载缓存数据，不进行网络请求
                            if (cacheType == DataCacheType.TEMP_CACHE) {
                                return;
                            }
                        } else {
                            listener.onResponse(cacheStr);
                            if (cacheType == DataCacheType.TEMP_CACHE) {
                                return;
                            }
                        }

                    } catch (JsonSyntaxException e) {
                        e.printStackTrace();
                    }
                }
                break;
            default:
                break;
        }
        GsonRequest request = new GsonRequest(Method.GET, url,
                apiRequest.getResponseType(), header, null,
                listener, apiRequest.getErrorlistener(), cacheType);
        request.setShouldCache(false);
        //网络错误并且使用TEMP_CACHE和use old cache还有待考察
        if (!useCache(apiRequest.getResponseType(), url, listener)) {
            httpService.addToRequestQueue(request);
        }

    }


    public void post(ApiRequest apiRequest, boolean needOauth, DataCacheType cacheType) {

        Map<String, String> header = new HashMap<String, String>();
        if (apiRequest.getHeaders() != null) {
            header.putAll(apiRequest.getHeaders());
        }
        header.put("code", ApiSettings.VERSION_CODE);
        header.put("osType", ApiSettings.OSTYPE);
        if (needOauth) {
            String accessToken = GlobalPhone.token;

            if(TextUtils.isEmpty(accessToken)){
                accessToken = DataCache.getDataCache()
                        .queryCache("access_token");
            }
            if (!TextUtils.isEmpty(accessToken)) {
                LogUtils.e("access_token", accessToken);
            }
//        header.put("Authorization", "Bearer" + " " + accessToken);
            header.put("token", accessToken);
        }

        LogUtils.e("------cacheType---------", cacheType.name());
        String url = apiRequest.getUrl(Request.Method.POST);
        Listener listener = apiRequest.getListener();
        if (apiRequest.getParams() != null && apiRequest.getParams().size() > 0) {
            //post有参数时候请求不建议使用use_old_cache方式，因为根据路径存储，post参数不在路径中
            if (cacheType == DataCacheType.USE_OLD_CACHE) {
                cacheType = DataCacheType.CACHE;
            }
        }
        switch (cacheType) {
            case USE_OLD_CACHE:
            case TEMP_CACHE:
                String cacheStr = DataCache.getDataCache().queryCache(url);
                if (!TextUtils.isEmpty(cacheStr)) {
                    try {
                        if (apiRequest.getResponseType() != null) {
                            //有缓存先加载缓存数据
                            listener.onResponse(gson.fromJson(cacheStr, apiRequest.getResponseType()));
                            //有缓存先加载缓存数据，不进行网络请求
                            if (cacheType == DataCacheType.TEMP_CACHE) {
                                return;
                            }
                        } else {
                            listener.onResponse(cacheStr);
                            if (cacheType == DataCacheType.TEMP_CACHE) {
                                return;
                            }
                        }

                    } catch (JsonSyntaxException e) {
                        e.printStackTrace();
                    }
                }
                break;
            default:
                break;
        }
        GsonRequest request = new GsonRequest(Request.Method.POST, url,
                apiRequest.getResponseType(), header,
                apiRequest.getParams(), listener,
                apiRequest.getErrorlistener(), cacheType);
        request.setShouldCache(false);
        //网络错误并且使用TEMP_CACHE和use old cache还有待考察
        if (!useCache(apiRequest.getResponseType(), url, listener)) {
            httpService.addToRequestQueue(request);
        }

    }

    /**
     * 图片请求
     */
    public ImageLoader.ImageContainer loadingImage(ImageView imageView, String url, int loadingResid,
                                                   int errorResid, BitmapCache mCache, ImageLoader.ImageListener listener) {
        if (listener == null) {
            listener = ImageLoader.getImageListener(imageView,
                    loadingResid, errorResid);
        }
        if (httpService.httpQueue != null) {
            ImageLoader imageLoader = new ImageLoader(httpService.httpQueue,
                    mCache);
            return imageLoader.get(url, listener);
        }

        return null;
    }

    public void loadingImage(ImageView imageView, String url, int loadingResid,
                             int errorResid, BitmapCache mCache) {
        loadingImage(imageView, url, loadingResid, errorResid, mCache, null);
    }

    /**
     * StringRequest
     */

    public void getByStringRequest(ApiRequest apiRequest, boolean needOauth, DataCacheType cacheType) {
        Map<String, String> header = new HashMap<String, String>();
        if (apiRequest.getHeaders() != null) {
            header.putAll(apiRequest.getHeaders());
        }
        header.put("code", ApiSettings.VERSION_CODE);
        header.put("osType", ApiSettings.OSTYPE);
        if (needOauth) {
            String accessToken = GlobalPhone.token;

            if(TextUtils.isEmpty(accessToken)){
                accessToken = DataCache.getDataCache()
                        .queryCache("access_token");
            }
            if (!TextUtils.isEmpty(accessToken)) {
                LogUtils.e("access_token", accessToken);
            }
//        header.put("Authorization", "Bearer" + " " + accessToken);
            header.put("token", accessToken);
        }
        String url = apiRequest.getUrl(Request.Method.GET);
        Listener listener = apiRequest.getListener();

        StringRequest request = new StringRequest(Request.Method.GET, url,
                header, null, listener, apiRequest.getErrorlistener(), cacheType);
        request.setShouldCache(false);
        if (!useCache(null, url, listener)) {
            httpService.addToRequestQueue(request);
        }

    }

    /**
     * StringRequest
     */
    public void postByStringRequest(ApiRequest apiRequest,
                                    boolean needOauth, DataCacheType cacheType) {
        Map<String, String> header = new HashMap<String, String>();
        if (apiRequest.getHeaders() != null) {
            header.putAll(apiRequest.getHeaders());
        }
        header.put("code", ApiSettings.VERSION_CODE);
        header.put("osType", ApiSettings.OSTYPE);
        String url = apiRequest.getUrl(Request.Method.POST);
        Listener listener = apiRequest.getListener();

        if (needOauth) {
            String accessToken = GlobalPhone.token;

            if(TextUtils.isEmpty(accessToken)){
                accessToken = DataCache.getDataCache()
                        .queryCache("access_token");
            }
            if (!TextUtils.isEmpty(accessToken)) {
                LogUtils.e("access_token", accessToken);
            }
//        header.put("Authorization", "Bearer" + " " + accessToken);
            header.put("token", accessToken);
        }
        Map<String, String> params = new HashMap<String, String>();
        for (String key : apiRequest.getParams().keySet()) {
            params.put(key, (String) apiRequest.getParams().get(key));
        }
        StringRequest request = new StringRequest(Request.Method.POST, url,
                header, params, listener,
                apiRequest.getErrorlistener(), cacheType);
        request.setShouldCache(false);
        if (!useCache(null, url, listener)) {
            httpService.addToRequestQueue(request);
        }
    }

    /**
     * 断网情况使用缓存处理
     *
     * @param clazz
     * @param url
     * @param listener
     * @return
     */
    private boolean useCache(Class clazz, String url, Listener listener) {
        HttpService.httpQueue.getCache().invalidate(url, true);
        if (!NetWorkUtils.detect(httpService.getContext())) {
            String cacheStr = DataCache.getDataCache().queryCache(url);
            if (cacheStr != null) {
                try {
                    if (clazz != null) {
                        listener.onResponse(gson.fromJson(cacheStr, clazz));
                    } else {
                        listener.onResponse(cacheStr);
                    }
                    return true;

                } catch (JsonSyntaxException e) {
                    e.printStackTrace();
                    return false;
                }
            }
            return false;
        } else {
            return false;
        }
    }

    public void getExtWithOAuth(com.core.api.event.request.Request apiRequest) {
        getExtWithOAuth(apiRequest, DataCacheType.NO_CACHE);
    }

    public void getExtWithOAuth(com.core.api.event.request.Request apiRequest, DataCacheType cacheType) {
        String url = apiRequest.getUrl(Request.Method.GET) + "?" + apiRequest.httpParam.encodeParametersToString("UTF-8");
        Listener listener = apiRequest.getListener();

        Map<String, String> header = new HashMap<String, String>();
        if (apiRequest.getHeaders() != null) {
            header.putAll(apiRequest.getHeaders());
        }
        header.put("code", ApiSettings.VERSION_CODE);
        header.put("osType", ApiSettings.OSTYPE);

        String accessToken = GlobalPhone.token;

        if(TextUtils.isEmpty(accessToken)){
            accessToken = DataCache.getDataCache()
                    .queryCache("access_token");
        }
        header.put("token", accessToken);

        GsonRequestExt request = new GsonRequestExt(Request.Method.GET, url,
                apiRequest.getResponseType(), header,
                apiRequest.httpParam, listener,
                apiRequest.getErrorlistener(), cacheType);
        request.setShouldCache(false);
        if (!useCache(apiRequest.getResponseType(), url, listener)) {
            httpService.addToRequestQueue(request);
        }
    }


    public void getExt(com.core.api.event.request.Request apiRequest, DataCacheType cacheType) {
        Map<String, String> header = new HashMap<String, String>();
        if (apiRequest.getHeaders() != null) {
            header.putAll(apiRequest.getHeaders());
        }
        header.put("code", ApiSettings.VERSION_CODE);
        header.put("osType", ApiSettings.OSTYPE);
        String url = apiRequest.getUrl(Request.Method.GET) + "?" + apiRequest.httpParam.encodeParametersToString("UTF-8");
        Listener listener = apiRequest.getListener();
        GsonRequestExt request = new GsonRequestExt(Request.Method.GET, url,
                apiRequest.getResponseType(), header,
                apiRequest.httpParam, listener,
                apiRequest.getErrorlistener(), cacheType);
        request.setShouldCache(false);
        if (!useCache(apiRequest.getResponseType(), url, listener)) {
            httpService.addToRequestQueue(request);
        }
    }


    public void postExt(com.core.api.event.request.Request apiRequest, DataCacheType cacheType) {
        Map<String, String> header = new HashMap<String, String>();
        if (apiRequest.getHeaders() != null) {
            header.putAll(apiRequest.getHeaders());
        }
        header.put("code", ApiSettings.VERSION_CODE);
        header.put("osType", ApiSettings.OSTYPE);
        String accessToken = GlobalPhone.token;

        if(TextUtils.isEmpty(accessToken)){
            accessToken = DataCache.getDataCache()
                    .queryCache("access_token");
        }

        if (!TextUtils.isEmpty(accessToken)) {
            header.put("token", accessToken);
        }

        String url = apiRequest.getUrl(Request.Method.POST);
        Listener listener = apiRequest.getListener();
        GsonRequestExt request = new GsonRequestExt(Request.Method.POST, url,
                apiRequest.getResponseType(), header,
                apiRequest.httpParam, listener,
                apiRequest.getErrorlistener(), cacheType);
        request.setShouldCache(false);
        if (!useCache(apiRequest.getResponseType(), url, listener)) {
            httpService.addToRequestQueue(request);
        }
    }

    // volley框架缓存
    // if (HttpService.httpQueue.getCache().get(url) != null) {
    // String cacheStr = new String(HttpService.httpQueue.getCache()
    // .get(url).data);
    //
    // if (cacheStr != null) {
    //
    // try {
    // listener.onResponse(cacheStr);
    //
    // } catch (JsonSyntaxException e) {
    // e.printStackTrace();
    // }
    //
    // return;
    // }
    //
    // }

}


```

#### 使用

- 先创建基础Request，所有Request的继承他

```
public class ApiRequest {

    public String url = "";

    protected Map<String, Object> params = null;

    private Class<?> clazz;

    private Response.Listener<?> listener;

    private Response.ErrorListener errorlistener;

    protected Map<String, String> headers;

    public ApiRequest(String values, Class<?> clazz) {
        this(values, clazz, false);
    }

    public ApiRequest(String values, Class<?> clazz, boolean isChonggou) {
        if (isChonggou) {
            this.url = values;
        } else {
            this.url = String.format("%1$s%2$s", ApiSettings.URL_BASE, values);
        }
        this.clazz = clazz;
        this.params = new HashMap<String, Object>();
//        params.put("versionCode",ApiSettings.VERSION_CODE);
        this.headers = new HashMap<String, String>();
    }


    public String getUrl(int method) {
        switch (method) {
            case Method.GET:
                return urlWithParameters(url, params);
            case Method.POST:
                return url;
        }
        return url;
    }

    public Map<String, Object> getParams() {
        if (params.isEmpty()) {
            Log.e("REQUEST_PARAMS", "null");
            return null;
        }
        return params;
    }

    public void setParams(Map<String, Object> params) {
        this.params = params;
    }

    public Class getResponseType() {
        return clazz;
    }

    public Response.Listener<?> getListener() {
        return listener;
    }

    public void setListener(Response.Listener<?> listener) {
        this.listener = listener;
    }

    public Response.ErrorListener getErrorlistener() {
        return errorlistener;
    }

    public void setErrorlistener(Response.ErrorListener errorlistener) {
        this.errorlistener = errorlistener;
    }

    public Map<String, String> getHeaders() {
        if (headers.isEmpty()) {
            Log.e("REQUEST_Header", "null");
            return null;
        }
        return headers;
    }

    public void setHeaders(Map<String, String> headers) {
        this.headers = headers;
    }

    public void addParams(Map<String, Object> params) {
        if (this.params != null) {
            this.params.putAll(params);
        } else {
            this.params = new HashMap<String, Object>();
            this.params.putAll(params);
        }
    }


    public static String urlWithParameters(String url,
                                           Map<String, Object> params) {
        StringBuilder builder = new StringBuilder();

        if (params != null) {
            for (HashMap.Entry<String, Object> entry : params.entrySet()) {
                if (builder.length() == 0) {
                    if (url.contains("?")) {
                        builder.append("&");
                    } else {
                        builder.append("?");
                    }
                } else {
                    builder.append("&");
                }
                builder.append(entry.getKey()).append("=")
                        .append(entry.getValue());
            }
        }

        return url + builder.toString();
    }


}


```

- 创建相应的request,ApiConstans中存放所有请求的路径

```
public class AnswerListRequest extends ApiRequest implements ApiConstants {

    public AnswerListRequest(int page,int pageSize) {
        super(API_ANSWERLIST, AnswerListResponse.class);
        params.put("page",page);
        params.put("pageSize",pageSize);
    }
}

```

当然，所有请求可以放到一个library包，做抽离



