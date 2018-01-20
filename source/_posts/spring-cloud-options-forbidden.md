---
title: Spring Cloud 项目出现 Options Forbidden 403 问题解决
date: 2017-11-06 21:00:00
categories: [Spring Cloud, Cors]
tags: [Spring Cloud, Cors, Forbidden]
---


> 摘要：本文简述了博主在开发过程中，需要跨域调试的时候，出现了 OPTIONS 请求 Forbidden 的问题，以及解决方法。

### 问题

![options-forbidden](https://1csh1.github.io/img/spring-cloud-options-forbidden/options-forbidden.png)

&emsp;&emsp;在使用 Spring Cloud 的项目中，本地跨域调试发现 POST 请求转为了 OPTIONS 请求，并且服务端拒绝访问，其实是 CORS 请求的问题。

&emsp;&emsp;CORS 请求分为2类： 简单请求 和 非简单请求。两者主要的区分点在于：
 * 1: 请求方法为 HEAD, GET, POST; 
 * 2: HTTP 头信息为以下几个： Accept， Accept-Language，Content-Language， Last-Event-ID，Content-Type （值为 application/x-www-form-urlencoded、multipart/form-data、text/plain）。

&emsp;&emsp;只要满足以上两点，则为简单请求；否则为非简单请求。

&emsp;&emsp;简单请求的处理方式是浏览器直接发送 CORS 请求。非简单请求的处理方式是浏览器发送预检请求，表示询问服务器当前的域名是否可以访问正常服务器，如果可以访问，则发送正常的请求到服务器；否则报错。

&emsp;&emsp;现在确定遇到的问题就是在 CORS 请求预检的时候发现域名不在服务器端的白名单里面，所以需要修改服务端的请求返回报文。

### 解决方案

在网关中添加下面的过滤器，在每次请求返回报文中添加报文头，即可正常访问

```
@Component
public class CorsFilter implements Filter {

    @Override
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException, ServletException {
        HttpServletResponse response = (HttpServletResponse) res;
        response.setHeader("Access-Control-Allow-Origin", "*");
        response.setHeader("Access-Control-Allow-Methods", "POST, GET, PUT, OPTIONS, DELETE, PATCH");
        response.setHeader("Access-Control-Max-Age", "3600");
        response.setHeader("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
        response.setHeader("Access-Control-Expose-Headers", "Location");
        chain.doFilter(req, res);
    }

    @Override
    public void init(FilterConfig filterConfig) {}

    @Override
    public void destroy() {}

}
```

参考文章：
[跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html)
