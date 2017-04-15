---
title: HTTP工具类
date: 2017-03-21 08:00:00
categories: [HTTP]
tags: [HTTP, 封装]
---

GitHub代码：[HttpUtil.java](https://github.com/1CSH1/util/tree/master/src/main/java/com/example/util/http/HttpUtil.java)

> 摘要：本文介绍在项目中用HttpClient来发送HTTP请求时，对HtttpClient进行的封装，使得我们在项目中使用HttpClient时可以更简单易用。

# Maven项目中的pom.xml

添加HttpClient依赖

```
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.2</version>
</dependency>
```

# HttpUtil.java

实现了对HttpClient的封装，代码如下

```
package com.csh.util.http;



import org.apache.http.Header;
import org.apache.http.HttpEntity;
import org.apache.http.NameValuePair;
import org.apache.http.client.ClientProtocolException;
import org.apache.http.client.entity.UrlEncodedFormEntity;
import org.apache.http.client.methods.*;
import org.apache.http.client.utils.URIBuilder;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.impl.conn.PoolingHttpClientConnectionManager;
import org.apache.http.message.BasicNameValuePair;
import org.apache.http.util.EntityUtils;

import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.net.URISyntaxException;
import java.util.ArrayList;
import java.util.Map;

/**
 * @author James-CSH
 * @since 3/14/2017 08:00 AM
 */
public class HttpUtil {
    private PoolingHttpClientConnectionManager cm;
    private String EMPTY_STR = "";
    private String UTF_8 = "UTF-8";

    public HttpUtil() {
        init(20, 5);
    }

    public HttpUtil(int maxTotal, int maxPerRoute) {
        init(maxTotal, maxPerRoute);
    }

    /**
     * 初始化
     * @param maxTotal      最大连接数
     * @param maxPerRoute   每个路由最大连接数
     */
    private synchronized void init(int maxTotal, int maxPerRoute) {
        if (null == cm) {
            cm = new PoolingHttpClientConnectionManager();
            cm.setMaxTotal(maxTotal);// 整个连接池最大连接数
            cm.setDefaultMaxPerRoute(maxPerRoute);// 每路由最大连接数，默认值是2
        }
    }

    private CloseableHttpClient getHttpClient() {
        return HttpClients.custom().setConnectionManager(cm).build();
    }

    /**
     * 设置请求头
     * @param http      发送的HTTP请求
     * @param headers   请求头
     * @return          发送的HTTP请求
     */
    private HttpRequestBase setHeaders(HttpRequestBase http, Map<String, Object> headers) {
        if (null != headers) {
            for (Map.Entry<String, Object> param : headers.entrySet()) {
                http.addHeader(param.getKey(), String.valueOf(param.getValue()));
            }
        }
        return http;
    }

    /**
     * 将参数转为特定格式
     * @param params    参数
     * @return
     */
    private ArrayList<NameValuePair> covertParams2NVPS(Map<String, Object> params) {
        ArrayList<NameValuePair> pairs = new ArrayList<NameValuePair>();
        if (null != params) {
            for (Map.Entry<String, Object> param : params.entrySet()) {
                pairs.add(new BasicNameValuePair(param.getKey(), String.valueOf(param.getValue())));
            }
        }
        return pairs;
    }

    /**
     * 处理Http请求
     *
     * @param request
     * @return
     */
    private String getResult(HttpRequestBase request) {
        CloseableHttpClient httpClient = getHttpClient();
        CloseableHttpResponse response = null;
        try {
            response = httpClient.execute(request);
            HttpEntity entity = response.getEntity();
            if (entity != null) {
                String result = EntityUtils.toString(entity);
                response.close();
                return result;
            }
        } catch (ClientProtocolException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                response.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        return EMPTY_STR;
    }

    /**
     * some http request just return the headers. such as head
     * @param request   
     * @return
     */
    private Header[] getHeaders(HttpRequestBase request) {
        CloseableHttpClient httpClient = getHttpClient();
        CloseableHttpResponse response = null;
        try {
            response = httpClient.execute(request);
            return response.getAllHeaders();
        } catch (ClientProtocolException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                response.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return null;
    }

    /**
     *  发送GET请求
     * @param url  请求路径
     * @return      请求结果
     * @throws URISyntaxException
     */
    public String get(String url) throws URISyntaxException {
        return get(url, null);
    }

    /**
     *  发送GET请求
     * @param url       请求路径
     * @param params    请求参数
     * @return          请求结果
     * @throws URISyntaxException
     */
    public String get(String url, Map<String, Object> params) throws URISyntaxException {
        return get(url, null, params);
    }

    /**
     *  发送GET请求
     * @param url       请求路径
     * @param headers   请求头
     * @param params    请求参数
     * @return          请求结果
     * @throws URISyntaxException
     */
    public String get(String url, Map<String, Object> headers, Map<String, Object> params) throws URISyntaxException {
        URIBuilder ub = new URIBuilder();
        ub.setPath(url);
        //设置参数
        ub.setParameters(covertParams2NVPS(params));
        HttpGet httpGet = new HttpGet(ub.build());
        //设置请求头
        httpGet = (HttpGet) setHeaders(httpGet, headers);

        return getResult(httpGet);
    }

    /**
     * 发送POST请求
     * @param url   请求路径
     * @return      请求结果
     * @throws UnsupportedEncodingException
     */
    public String post(String url) throws UnsupportedEncodingException {
        return post(url, null);
    }

    /**
     * 发送POST请求
     * @param url       请求路径
     * @param params    请求参数
     * @return          请求结果
     * @throws UnsupportedEncodingException
     */
    public String post(String url, Map<String, Object> params) throws UnsupportedEncodingException {
        return post(url, null, params);
    }

    /**
     * 发送POST请求
     * @param url       请求路径
     * @param headers   请求头
     * @param params    请求参数
     * @return          请求结果
     * @throws UnsupportedEncodingException
     */
    public String post(String url, Map<String, Object> headers, Map<String, Object> params) throws UnsupportedEncodingException {
        HttpPost httpPost = new HttpPost(url);
        //设置参数
        ArrayList<NameValuePair> pairs = covertParams2NVPS(params);
        httpPost.setEntity(new UrlEncodedFormEntity(pairs, UTF_8));
        //设置请求头
        httpPost = (HttpPost) setHeaders(httpPost, headers);
        return getResult(httpPost);
    }

    /**
     * 发送PUT请求
     * @param url   请求路径
     * @return      请求结果
     * @throws UnsupportedEncodingException
     */
    public String put(String url) throws UnsupportedEncodingException {
        return put(url, null);
    }

    /**
     * 发送PUT请求
     * @param url       请求路径
     * @param params    请求参数
     * @return          请求结果
     * @throws UnsupportedEncodingException
     */
    public String put(String url, Map<String, Object> params) throws UnsupportedEncodingException {
        return put(url, null, params);
    }

    /**
     * 发送PUT请求
     * @param url       请求路径
     * @param headers   请求头
     * @param params    请求参数
     * @return          请求结果
     * @throws UnsupportedEncodingException
     */
    public String put(String url, Map<String, Object> headers, Map<String, Object> params) throws UnsupportedEncodingException {
        HttpPut httpPut = new HttpPut(url);
        //设置请求参数
        ArrayList<NameValuePair> pairs = covertParams2NVPS(params);
        httpPut.setEntity(new UrlEncodedFormEntity(pairs, UTF_8));
        //设置请求头
        httpPut = (HttpPut) setHeaders(httpPut, headers);

        return getResult(httpPut);
    }

    /**
     * 发送PATCH请求
     * @param url   请求路径
     * @return      请求结果
     * @throws UnsupportedEncodingException
     */
    public String patch(String url) throws UnsupportedEncodingException {
        return patch(url, null);
    }

    /**
     * 发送PATCH请求
     * @param url       请求路径
     * @param params    请求参数
     * @return          请求结果
     * @throws UnsupportedEncodingException
     */
    public String patch(String url, Map<String, Object> params) throws UnsupportedEncodingException {
        return patch(url, null, params);
    }

    /**
     * 发送PATCH请求
     * @param url       请求路径
     * @param headers   请求头
     * @param params    请求参数
     * @return          请求结果
     * @throws UnsupportedEncodingException
     */
    public String patch(String url, Map<String, Object> headers, Map<String, Object> params) throws UnsupportedEncodingException {
        HttpPatch httpPatch = new HttpPatch(url);
        //设置请求参数
        ArrayList<NameValuePair> pairs = covertParams2NVPS(params);
        httpPatch.setEntity(new UrlEncodedFormEntity(pairs, UTF_8));
        //设置请求头
        httpPatch = (HttpPatch) setHeaders(httpPatch, headers);

        return getResult(httpPatch);
    }

    /**
     * 发送DELETE请求
     * @param url   请求路径
     * @return      请求结果
     * @throws UnsupportedEncodingException
     */
    public String delete(String url) throws UnsupportedEncodingException, URISyntaxException {
        return delete(url, null);
    }

    /**
     * 发送DELETE请求
     * @param url       请求路径
     * @param params    请求参数
     * @return          请求结果
     * @throws UnsupportedEncodingException
     * @throws URISyntaxException
     */
    public String delete(String url, Map<String, Object> params) throws UnsupportedEncodingException, URISyntaxException {
        return delete(url, null, params);
    }

    /**
     * 发送DELETE请求
     * @param url       请求路径
     * @param headers   请求头
     * @param params    请求参数
     * @return          请求结果
     * @throws UnsupportedEncodingException
     */
    public String delete(String url, Map<String, Object> headers, Map<String, Object> params) throws UnsupportedEncodingException, URISyntaxException {
        URIBuilder ub = new URIBuilder();
        ub.setPath(url);
        //设置请求参数
        ub.addParameters(covertParams2NVPS(params));
        HttpDelete httpDelete = new HttpDelete(ub.build());
        //设置请求头
        httpDelete = (HttpDelete) setHeaders(httpDelete, headers);

        return getResult(httpDelete);
    }

    /**
     * 发送TRACE请求
     * 回显服务器收到的请求，主要用于测试或诊断。
     * @param url       请求路径
     * @return          请求结果
     * @throws UnsupportedEncodingException
     */
    public String trace(String url) throws UnsupportedEncodingException, URISyntaxException {
        return trace(url, null);
    }

    /**
     * 发送TRACE请求
     * 回显服务器收到的请求，主要用于测试或诊断
     * @param url       请求路径
     * @param params    请求参数
     * @return          请求结果
     * @throws UnsupportedEncodingException
     */
    public String trace(String url, Map<String, Object> params) throws UnsupportedEncodingException, URISyntaxException {
        return trace(url, null, params);
    }

    /**
     * 发送TRACE请求
     * 回显服务器收到的请求，主要用于测试或诊断
     * @param url       请求路径
     * @param headers   请求头
     * @param params    请求参数
     * @return          请求结果
     * @throws UnsupportedEncodingException
     */
    public String trace(String url, Map<String, Object> headers, Map<String, Object> params) throws UnsupportedEncodingException, URISyntaxException {
        URIBuilder ub = new URIBuilder();
        ub.setPath(url);
        //设置请求参数
        ub.setParameters(covertParams2NVPS(params));
        HttpTrace httpTrace = new HttpTrace(ub.build());
        //设置请求头
        httpTrace = (HttpTrace) setHeaders(httpTrace, headers);

        return getResult(httpTrace);
    }

    /**
     * 发送HEAD请求
     * 类似于get请求，只不过返回的响应中没有具体的内容，用于获取报头
     * @param url   请求路径
     * @return      请求结果
     * @throws UnsupportedEncodingException
     * @throws URISyntaxException
     */
    public Header[] head(String url) throws UnsupportedEncodingException, URISyntaxException {
        return head(url, null);
    }

    /**
     * 发送HEAD请求
     * 类似于get请求，只不过返回的响应中没有具体的内容，用于获取报头
     * @param url       请求路径
     * @param params    请求参数
     * @return          请求结果
     * @throws UnsupportedEncodingException
     * @throws URISyntaxException
     */
    public Header[] head(String url, Map<String, Object> params) throws UnsupportedEncodingException, URISyntaxException {
        return head(url, null, params);
    }

    /**
     * 发送HEAD请求
     * 类似于get请求，只不过返回的响应中没有具体的内容，用于获取报头
     * @param url       请求路径
     * @param headers   请求头
     * @param params    请求参数
     * @return          请求结果
     * @throws UnsupportedEncodingException
     * @throws URISyntaxException
     */
    public Header[] head(String url, Map<String, Object> headers, Map<String, Object> params) throws UnsupportedEncodingException, URISyntaxException {
        URIBuilder ub = new URIBuilder();
        ub.setPath(url);
        //设置请求参数
        ub.setParameters(covertParams2NVPS(params));
        HttpHead httpHead = new HttpHead(ub.build());
        //设置请求头
        httpHead = (HttpHead) setHeaders(httpHead, headers);

        return getHeaders(httpHead);
    }

    /**
     * 发送OPTIONS请求
     * 允许客户端查看服务器的性能
     * @param url   请求路径
     * @return      请求结果
     * @throws UnsupportedEncodingException
     * @throws URISyntaxException
     */
    public String options(String url) throws UnsupportedEncodingException, URISyntaxException {
        return options(url, null);
    }

    /**
     * 发送OPTIONS请求
     * 允许客户端查看服务器的性能
     * @param url       请求路径
     * @param params    请求参数
     * @return          请求结果
     * @throws UnsupportedEncodingException
     * @throws URISyntaxException
     */
    public String options(String url, Map<String, Object> params) throws UnsupportedEncodingException, URISyntaxException {
        return options(url, null, params);
    }

    /**
     * 发送OPTIONS请求
     * 允许客户端查看服务器的性能
     * @param url       请求路径
     * @param headers   请求头
     * @param params    请求参数
     * @return          请求结果
     * @throws UnsupportedEncodingException
     * @throws URISyntaxException
     */
    public String options(String url, Map<String, Object> headers, Map<String, Object> params) throws UnsupportedEncodingException, URISyntaxException {
        URIBuilder ub = new URIBuilder();
        ub.setPath(url);
        //设置参数
        ub.setParameters(covertParams2NVPS(params));
        HttpOptions httpOptions = new HttpOptions(ub.build());
        //设置请求头
        httpOptions = (HttpOptions) setHeaders(httpOptions, headers);

        return getResult(httpOptions);
    }

}

```


