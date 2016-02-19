title: Volley在项目中的使用
date: 2016-02-19 
tags: [android]
categories: [android]
toc: true
---


## Volley在项目中的使用*（volley+okhttp+gson）*

>使用`volley`、`okhttp`、`gson `进行网络层的架构是当前Android端流行的网络框架之一。对Volley进行二次封装，以便适用于`RESTful API`的接口请求，并提高开发效率。这篇文章结合项目从`Volley `，`okhttp`,`restful api`等方面讲解如何进行二次封装。

### Volley
-------------------
关于Volley的介绍及基本使用可以参考这篇文章[Volley源码解析](http://a.codekk.com/detail/Android/grumoon/Volley%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90).
可以简单总结为Volley适用于数据量小、请求频繁的网络场景（市面上大部分应用都适用）,异步请求网络支持restful api。

使用方法如下：

- 获取RequestQueue。

		//public static RequestQueue newRequestQueue(Context context)
		//public static RequestQueue newRequestQueue(Context context, HttpStack stack)

		RequestQueue mRequestQueue = Volley.newRequestQueue(context,stack));
	
- 创建一个Request。
	Volley为我们提供了`StringRequest `、`JsonRequest `、`JsonObjectRequest `、`JsonArrayRequest `、`ImageRequest `这几种网络请求类以获取Request。对于简单的网络请求及简单的数据交互而言这些方法组员满足需求。但随着业务的复杂化这些原生请求会耗费很多精力去处理数据解析以及参数传递的事情。

- 将创建的请求加入队列，即完成请求。

		mRequestQueue.add(Request)
- 我们还可以对请求设置Tag，及retry.

		request.setTag(SilverConstant.RequestTag.REQUEST_TAG_PUT_JPUSH);
		request.setRetryPolicy(new DefaultRetryPolicy(20 * 1000, 1, 1.0f));
		
### okhttp
-----------
网上重写httpstack的例子很多，下面是我扣下的一个加以注释。
	
	package com.silversnet.cloudpos.http;
	
	import com.android.volley.AuthFailureError;
	import com.android.volley.Request;
	import com.android.volley.toolbox.HttpStack;
	import com.squareup.okhttp.Call;
	import com.squareup.okhttp.Headers;
	import com.squareup.okhttp.MediaType;
	import com.squareup.okhttp.OkHttpClient;
	import com.squareup.okhttp.Protocol;
	import com.squareup.okhttp.RequestBody;
	import com.squareup.okhttp.Response;
	import com.squareup.okhttp.ResponseBody;
	
	import org.apache.http.HttpEntity;
	import org.apache.http.HttpResponse;
	import org.apache.http.ProtocolVersion;
	import org.apache.http.StatusLine;
	import org.apache.http.entity.BasicHttpEntity;
	import org.apache.http.message.BasicHeader;
	import org.apache.http.message.BasicHttpResponse;
	import org.apache.http.message.BasicStatusLine;
	
	import java.io.IOException;
	import java.util.Map;
	import java.util.concurrent.TimeUnit;
	
	public class OkHttpStack implements HttpStack {
	    private final OkHttpClient mClient;
	
	    public OkHttpStack(OkHttpClient client) {
	        this.mClient = client;
	    }
	
	    /**
	     * 执行请求,获取请求结果
	     *
	     * @param request           the request to perform
	     * @param additionalHeaders additional headers to be sent together with
	     *                          {@link Request#getHeaders()}
	     * @return
	     * @throws IOException
	     * @throws AuthFailureError
	     */
	    @Override
	    public HttpResponse performRequest(Request<?> request, Map<String, String> additionalHeaders) throws IOException, AuthFailureError {
	
	        OkHttpClient client = mClient.clone();
	        int timeoutMs = request.getTimeoutMs();
	        client.setConnectTimeout(timeoutMs, TimeUnit.MILLISECONDS);
	        client.setReadTimeout(timeoutMs, TimeUnit.MILLISECONDS);
	        client.setWriteTimeout(timeoutMs, TimeUnit.MILLISECONDS);
	
	        com.squareup.okhttp.Request.Builder okHttpRequestBuilder = new com.squareup.okhttp.Request.Builder();
	        okHttpRequestBuilder.url(request.getUrl());
	
	        Map<String, String> headers = request.getHeaders();
	        for (final String name : headers.keySet()) {
	            okHttpRequestBuilder.addHeader(name, headers.get(name));
	        }
	        for (final String name : additionalHeaders.keySet()) {
	            //附加头信息
	            okHttpRequestBuilder.addHeader(name, additionalHeaders.get(name));
	        }
	
	        setConnectionParametersForRequest(okHttpRequestBuilder, request);
	
	        //创建一条完整的okhttp请求
	        com.squareup.okhttp.Request okHttpRequest = okHttpRequestBuilder.build();
	        Call okHttpCall = client.newCall(okHttpRequest);
	        //执行请求，获取返回结果
	        Response okHttpResponse = okHttpCall.execute();
	
	        //由返回码，获取返回状态
	        StatusLine responseStatus = new BasicStatusLine(parseProtocol(okHttpResponse.protocol()), okHttpResponse.code(), okHttpResponse.message());
	        BasicHttpResponse response = new BasicHttpResponse(responseStatus);
	        response.setEntity(entityFromOkHttpResponse(okHttpResponse));
	
	        Headers responseHeaders = okHttpResponse.headers();
	        for (int i = 0, len = responseHeaders.size(); i < len; i++) {
	            final String name = responseHeaders.name(i), value = responseHeaders.value(i);
	            if (name != null) {
	                response.addHeader(new BasicHeader(name, value));
	            }
	        }
	
	        return response;
	    }
	
	    /**
	     * HttpEntity
	     *
	     * @param r Response
	     * @return
	     * @throws IOException
	     */
	    private static HttpEntity entityFromOkHttpResponse(Response r) throws IOException {
	        BasicHttpEntity entity = new BasicHttpEntity();
	        ResponseBody body = r.body();
	
	        entity.setContent(body.byteStream());
	        entity.setContentLength(body.contentLength());
	        entity.setContentEncoding(r.header("Content-Encoding"));
	
	        if (body.contentType() != null) {
	            entity.setContentType(body.contentType().type());
	        }
	        return entity;
	    }
	
	    /**
	     * 设置请求的连接参数（请求类型，请求体）
	     *
	     * @param builder
	     * @param request
	     * @throws IOException
	     * @throws AuthFailureError
	     */
	    @SuppressWarnings("deprecation")
	    private static void setConnectionParametersForRequest(com.squareup.okhttp.Request.Builder builder, Request<?> request) throws IOException, AuthFailureError {
	        switch (request.getMethod()) {
	            case Request.Method.DEPRECATED_GET_OR_POST:
	                // Ensure backwards compatibility.  Volley assumes a request with a null body is a GET.
	                byte[] postBody = request.getPostBody();
	                if (postBody != null) {
	                    builder.post(RequestBody.create(MediaType.parse(request.getPostBodyContentType()), postBody));
	                }
	                break;
	            case Request.Method.GET:
	                builder.get();
	                break;
	            case Request.Method.DELETE:
	                builder.delete();
	                break;
	            case Request.Method.POST:
	                builder.post(createRequestBody(request));
	                break;
	            case Request.Method.PUT:
	                builder.put(createRequestBody(request));
	                break;
	            case Request.Method.HEAD:
	                builder.head();
	                break;
	            case Request.Method.OPTIONS:
	                builder.method("OPTIONS", null);
	                break;
	            case Request.Method.TRACE:
	                builder.method("TRACE", null);
	                break;
	            case Request.Method.PATCH:
	                builder.patch(createRequestBody(request));
	                break;
	            default:
	                throw new IllegalStateException("Unknown method type.");
	        }
	    }
	
	    private static ProtocolVersion parseProtocol(final Protocol p) {
	        switch (p) {
	            case HTTP_1_0:
	                return new ProtocolVersion("HTTP", 1, 0);
	            case HTTP_1_1:
	                return new ProtocolVersion("HTTP", 1, 1);
	            case SPDY_3:
	                return new ProtocolVersion("SPDY", 3, 1);
	            case HTTP_2:
	                return new ProtocolVersion("HTTP", 2, 0);
	        }
	
	        throw new IllegalAccessError("Unkwown protocol");
	    }
	
	    private static RequestBody createRequestBody(Request r) throws AuthFailureError {
	        final byte[] body = r.getBody();
	        if (body == null)
	            return null;
	
	        return RequestBody.create(MediaType.parse(r.getBodyContentType()), body);
	    }
	}

### RESTful API
----------------
RESTful API就是一种接口定义风格，以`resource`为主体，充分使用Http协议的`post `、`put `、`get `、`delete `对resource进行CRUD操作，从形式上约束url格式。
可以参考[理解RESTful架构](http://www.ruanyifeng.com/blog/2011/09/restful)、[RESTful API 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)进行进一步理解.
### RequestQueue的封装
------------------
由于一个应用对应一个请求队列，那么我们的RequestQueue应该也是唯一的。只需要在application初始化时进行初始化就行。代码如下：

	package com.silversnet.cloudpos.http;
	
	import android.content.Context;
	
	import com.android.volley.Request;
	import com.android.volley.RequestQueue;
	import com.android.volley.toolbox.Volley;
	import com.squareup.okhttp.OkHttpClient;
	
	/**
	 * Author：zhongshan
	 * Date：
	 * Description：
	 */
	public class VolleyHelper {
	    private static VolleyHelper mInstance;
	    public RequestQueue mRequestQueue;
	
	    private VolleyHelper() {
	    }
	
	
	    public void init(Context context){
	        if (mRequestQueue == null ) {
	        	//这里我们可以看出我们使用的HttpStack为自定义的OkhttpStack
	            mRequestQueue = Volley.newRequestQueue(context, new OkHttpStack(new OkHttpClient()));
	        }
	    }
	
	    public static synchronized VolleyHelper getInstance() {
	        if (mInstance == null) {
	            mInstance = new VolleyHelper();
	        }
	        return mInstance;
	    }
	
	    /**
	     * Returns a Volley request queue for creating network requests
	     *
	     * @return {@link RequestQueue}
	     */
	    public RequestQueue getRequestQueue() {
	        if (null != mRequestQueue) {
	            return mRequestQueue;
	        } else {
	            throw new IllegalArgumentException("RequestQueue is not initialized.");
	        }
	    }
	
	    /**
	     * Adds a request to the Volley request queue
	     *
	     * @param request is the request to add to the Volley queue
	     */
	    public <T> void addRequest(Request<T> request) {
	        getRequestQueue().add(request);
	    }
	    /**
	     * Adds a request to the Volley request queue
	     *
	     * @param request is the request to add to the Volley queuest
	     * @param tag is the tag identifying the request
	     */
	    public <T> void addRequest(Request<T> request, String tag) {
	        request.setTag(tag);
	        getRequestQueue().add(request);
	    }
	    /**
	     * Cancels all the request in the Volley queue for a given tag
	     *
	     * @param tag associated with the Volley requests to be cancelled
	     */
	    public void cancelAllRequests(String tag) {
	        if (getRequestQueue() != null) {
	            getRequestQueue().cancelAll(tag);
	        }
	    }
	}

我们只需要在Application中进行 VolleyHelper.getInstance().init(this);，在请求时调用VolleyHelper.getInstance().add()即可。

### Request 的封装
这部分是最核心部分，由代码可以看出是`Request<T> `的子类，使用`Builder模式`传入参数。使用`HttpSecurityManager`去处理URL中的参数.
这里的参数分为两类：path variables和query parameters：
	
	/api/resource/{p1}/subresource/{p2}?q1=1&q2=2
	
- path variables: p1, p2
- query parameters: q1, q2

封装Request的主要组成类为`Gson `(处理解析)、`HttpSecurityManager`和`SecurityUtil `处理url参数。

代码如下

	package com.silversnet.cloudpos.http;
	
	import android.util.Log;
	
	import com.android.volley.AuthFailureError;
	import com.android.volley.NetworkResponse;
	import com.android.volley.ParseError;
	import com.android.volley.Request;
	import com.android.volley.Response;
	import com.android.volley.toolbox.HttpHeaderParser;
	import com.google.gson.Gson;
	import com.google.gson.JsonSyntaxException;
	import com.silversnet.cloudpos.SilverApplication;
	import com.silversnet.cloudpos.exception.SilverException;
	import com.silversnet.cloudpos.manager.PaymentManager;
	import com.silversnet.cloudpos.util.DateUtil;
	import com.silversnet.cloudpos.util.SecurityUtil;
	
	import java.io.UnsupportedEncodingException;
	import java.util.HashMap;
	import java.util.Map;
	
	/**
	 * Author：zhongshan
	 * Date：
	 * Description：
	 */
	public class GsonRequest<T> extends Request<T> {

    private static final String CLASSNAME = "GsonRequest";

    private final Gson gson = new Gson();
    private final Class<T> clazz;
    private final Map<String, String> headers;
    private final Response.Listener<T> listener;
    private String params;
    private Map<String, String> pathVars;
    private Map<String, String> queryParams;
    private int statusCode;

    /**
     * Make a GET request and return a parsed object from JSON.
     *
     * @param url    URL of the request to make
     * @param clazz  Relevant class object, for Gson's reflection
     * @param params Map of request params
     */
    public GsonRequest(String url, Class<T> clazz, String params, Response.Listener<T> listener, Response.ErrorListener errorListener) {
        super(Method.GET, url, errorListener);
        this.clazz = clazz;
        this.headers = null;
        this.params = params;
        this.listener = listener;
    }

    /**
     * Make a request and return a parsed object from JSON.
     *
     * @param url     URL of the request to make
     * @param clazz   Relevant class object, for Gson's reflection
     * @param headers Map of request headers
     */
    public GsonRequest(int method, String url, Class<T> clazz, Map<String, String> headers, String params, Response.Listener<T> listener, Response.ErrorListener errorListener) {
        super(method, url, errorListener);
        this.clazz = clazz;
        this.headers = headers;
        this.params = params;
        this.listener = listener;
    }

    /**
     * @param builder requestBuilder
     */
    public GsonRequest(RequestBuilder builder) {
        super(builder.method, builder.getUrl(), builder.errorListener);
        clazz = builder.clazz;
        if (builder.headers.size() > 0) {
            headers = builder.headers;
        } else {
            headers = builder.httpSecurityManager.getHeaders();
        }
        listener = builder.successListener;
        params = builder.params;
        this.pathVars = builder.httpSecurityManager.getPathVars();
        this.queryParams = builder.httpSecurityManager.getQueryParams();
    }

    @Override
    public Map<String, String> getHeaders() throws AuthFailureError {
        try {
            long timestamp = DateUtil.timestamp();
            // TODO: consider getting appKey from database per cposId
            String appKey = SilverApplication.props.getProperty("cpos.appKey");
            String hash = SecurityUtil.md5(getPathVarStr() + getParamStr() + timestamp + appKey);
            headers.put("cposId", PaymentManager.getInstance().getCposId());
            headers.put("timestamp", String.valueOf(timestamp));
            headers.put("hash", hash);
        } catch (SilverException e) {
            Log.e(CLASSNAME, "Failed to generate request headers", e);
            throw new AuthFailureError();
        }

        return headers;
    }

    @Override
    protected void deliverResponse(T response) {
        listener.onResponse(response);
    }

    /**
     * 请求参数组装：重写getBody方法
     *
     * @throws AuthFailureError
     */
    @Override
    public byte[] getBody() throws AuthFailureError {
        return params == null ? super.getBody() : params.getBytes();
    }

    public int getStatusCode() {
        return statusCode;
    }

    @Override
    protected Response<T> parseNetworkResponse(NetworkResponse response) {
        String parsed;
        statusCode = response.statusCode;
        Log.e("StatusCode： ", statusCode + "");
        try {
            parsed = new String(response.data, "UTF-8");
            Log.e("Response： ", parsed);
            if (clazz == null) {
                return (Response<T>) Response.success(parsed, HttpHeaderParser.parseCacheHeaders(response));
            } else {
                return Response.success(gson.fromJson(parsed, clazz), HttpHeaderParser.parseCacheHeaders(response));
            }
        } catch (UnsupportedEncodingException e) {
            return Response.error(new ParseError(e));
        } catch (JsonSyntaxException e) {
            return Response.error(new ParseError(e));
        } catch (Exception e) {
            return Response.error(new ParseError(e));
        }
    }

    public static class RequestBuilder {
        private SecurityManager securityManager = new SecurityManager();
        private int method = Method.GET;
        private String url;
        private String domain;
        private Class clazz;
        private Response.Listener successListener;
        private Response.ErrorListener errorListener;
        private Map<String, String> headers = new HashMap<>();
        private String params;
        private HttpSecurityManager httpSecurityManager = new HttpSecurityManager();

        /**
         * 添加域名
         */
        public RequestBuilder domain(String domain) {
            this.domain = domain;
            return this;
        }

        /**
         * 添加class
         */
        public RequestBuilder clazz(Class clazz) {
            this.clazz = clazz;
            return this;
        }

        /**
         * request success
         */
        public RequestBuilder successListener(Response.Listener successListener) {
            this.successListener = successListener;
            return this;
        }

        /**
         * request error
         */
        public RequestBuilder errorListener(Response.ErrorListener errorListener) {
            this.errorListener = errorListener;
            return this;
        }

        /**
         * get request
         */
        public RequestBuilder get(String url) {
            this.url = url;
            this.method = Method.GET;
            return this;
        }

        /**
         * post request
         */
        public RequestBuilder post(String url) {
            this.url = url;
            this.method = Method.POST;
            return this;
        }

        /**
         * put request
         */
        public RequestBuilder put(String url) {
            this.url = url;
            this.method = Method.PUT;
            return this;
        }

        /**
         * delete request
         */
        public RequestBuilder delete(String url) {
            this.url = url;
            this.method = Method.DELETE;
            return this;
        }

        /**
         * add request headers
         */
        public RequestBuilder headers(Map<String, String> headers) {
            this.headers = headers;
            return this;
        }

        /**
         * add pathVars
         */
        public RequestBuilder addPathVar(String key, String value) {
            httpSecurityManager.addPathVar(key, value);
            return this;
        }

        /**
         * add queryParams
         */
        public RequestBuilder addQueryParam(String key, String value) {
            httpSecurityManager.addQueryParam(key, value);
            return this;
        }

        /**
         * params
         */
        public RequestBuilder params(String params) {
            this.params = params != null ? params : null;
            return this;
        }

        private String getUrl() {
            String url = this.domain + this.url;
            Map<String, String> paths = httpSecurityManager.getPathVars();
            /**替换{pathkey}-path variables*/
            for (String pathKey : paths.keySet()) {
                String pathVal = paths.get(pathKey);
                url = url.replace("{" + pathKey + "}", pathVal);
            }
            Map<String, String> params = httpSecurityManager.getQueryParams();
            /**替换?q1=1&q2=2-query parameters*/
            url += SecurityUtil.generateQueryString(params);
            return url;
        }

        public GsonRequest build() {
            return new GsonRequest(this);
        }
    }

    private String getPathVarStr() {
        String pathVarString = SecurityUtil.pathVarString(pathVars);
        return pathVarString;
    }

    private String getParamStr() {
        String paramString = SecurityUtil.sortedValString(queryParams);
        return paramString;
    }

    @Override
    public String getBodyContentType() {
        return "application/json; charset=utf-8";
    }
	}

其中我们无法通过header去控制ContentType，如果不重写getBodyContentType()方法，会默认为表单提交。
创建形式为：
	
	GsonRequest requestPaymentTypeList = new GsonRequest.RequestBuilder().domain(SilverConstant
	                .BASE_REQUEST_DOMAIN)
	                .get(SilverConstant.RequestPath.GET_PAYMENT_TYPE_LIST)
	                .errorListener(new Response.ErrorListener() {
	                    @Override
	                    public void onErrorResponse(VolleyError error) {
	                        syncSuccess = false;
	                        verifyRequestsComplete();
	                    }
	                })
	                .successListener(new Response.Listener() {
	                    @Override
	                    public void onResponse(Object response) {
	                        if (response != null) {
	                            PaymentTypeDao.getInstance(context).save((String) response);
	                        }
	                        verifyRequestsComplete();
	                    }
	                })
	                .build();
	        requestPaymentTypeList .setTag(SilverConstant.RequestTag.REQUEST_TAG_GET_PAYMENT_TYPE_LIST);
	        requestPaymentTypeList .setRetryPolicy(new DefaultRetryPolicy(20 * 1000, 1, 1.0f));
	        VolleyHelper.getInstance().addRequest(requestPaymentTypeList);
	        
其实我们可以对请求进行进一步封装，并将异步请求的结果通过`ResponseListener`接口处理。

	public interface ResponseListener {
	    void ResponseSuccess(Object response, String tag);
	    void ResponseError(VolleyError error, String tag);
	}

进一步封装如下：
	
	public static GsonRequest doRequest(String url, int method, Map<String, String> pathVars,
	      Map<String, String> queryParams, String params, Class clazz,
	      final ResponseListener responseListener, final String tag) {
	
	    Map<String, String> headers = getHeaders(pathVars, queryParams);
	
	    BaseRequest request = new BaseRequest.RequestBuilder().domain(ApiConstants.BASE_REQUEST_DOMAIN)
	        .url(url)
	        .method(method)
	        .headers(headers)
	        .addPaths(pathVars)
	        .addParams(queryParams)
	        .params(params)
	        .clazz(clazz)
	        .successListener(new Response.Listener() {
	          @Override public void onResponse(Object response) {
	            Object msgObject = null;
	            try {
	              msgObject = ResponseParseFactory.createMsgObject(tag, response);
	            }catch (Exception e){
	              e.printStackTrace();
	            }
	            responseListener.ResponseSuccess(msgObject, tag);
	          }
	        })
	        .errorListener(new Response.ErrorListener() {
	          @Override public void onErrorResponse(VolleyError error) {
	            responseListener.ResponseError(error, tag);
	          }
	        })
	        .build();
	    request.setTag(tag);
	    request.setRetryPolicy(new DefaultRetryPolicy(20 * 1000, 1, 1.0f));
	    HttpRequestHelper.getInstance().addRequest(request);
	    return request;
	  }
	  
### 最后
---
项目中使用Volley已经有一段时间了，今天把代码贴了出来，也算是回顾吧，虽然使用了更适合RESTful api并且效率更高的Retrofit。O(∩_∩)O~













