---
title: HTTP缓存策略
tags: [HTTP]
category: Frontend
filename: cache strategy of http static resource 
createdTime: 2019-12-12 19:38
---
一般来说，对于html, css, js以及图片等静态资源文件，浏览器会进行缓存，这样下次访问这些资源的时候就可以直接从缓存获取，从而提升页面的加载速度，但是浏览器的缓存机制遵循一定的策略.

### 什么情况下会缓存？
如果希望资源被浏览器缓存到本地，需要满足以下条件：
1. **http响应头Content-type必须标明当前资源是可被缓存的静态资源，例如text/html, application/javascript, text/css, image/png等等**
2. **http响应头Cache-Control指明了该资源允许被缓存**


### 与缓存有关的HTTP请求头和响应头？
![http缓存截图](/site_assets/http-cache-headers.png)
1. **响应头**
    1. Cache-Control: public max-age=3600, public代表资源可以被网络链路上的任何设备缓存，max-age指令代表缓存时长，例如max-age=3600表示浏览器应该缓存这份资源一个小时，Cache-Control请求头可以包含很多指令，详见[MDN解释](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control)
    2. Expires: Sat, 14 Dec 2019 02:07:08 GMT, 代表缓存资源的过期时间，Cache-Control的优先级比Expires高；
    3. Etag: W/"5df19f2c-626bc", 代表当前资源文件内容的签名；
    4. Last-Modified: Thu, 12 Dec 2019 02:00:12 GMT, 代表当前资源文件最近一次的改动时间
2. **请求头**
    1. If-Modified-Since: Thu, 12 Dec 2019 02:02:19 GMT, 代表当前资源文件最近一次的改动时间，询问服务器当前文件是否已经被更新；
    2. If-None-Match: W/"13a175-16ef7d7b473", 代表资源文件内容的签名，询问服务器当前文件是否已经被更新；

### 什么是强缓存？
所谓强缓存就是在浏览器的本地缓存里面找到了资源的副本并且副本未过期，不需要再向服务器发出请求，如图：
![local cache](/site_assets/strict-cache.png)
此时浏览器自动给你一个200 OK(from disk cache)的响应

### 什么是协商缓存？
所谓协商缓存就是在浏览器的本地缓存里面未找找到了副本但是副本已过期，此时浏览器会向服务器发出请求，并且带上对应的If-Modified-Since和If-None-Match请求头，再次和服务器确认缓存是否真正已过期，服务器根据If-Modified-Since或If-None-Match请求头的内容判断文件是否发生了更新，如果是，则说明浏览器缓存确实过期，返回正常的200 OK，响应体为最新的文件内容，反之，服务器返回304 Not Modified，告诉浏览器可以继续使用本地缓存，响应体为空，免去了文件内容的下载时间，如图：
![condition cache](/site_assets/condition-cache.png);

### 静态资源总体的加载流程？
综合以上分析，浏览器在加载静态资源的时候，大概会经历以下流程：
1. 检查本地缓存是否有副本，如果没有，直接请求server，流程结束，如果有，检查缓存是否未过期；
2. server通过Cache-Control的max-age指令指明资源的缓存时长，或者通过Expires的值指明资源的过期时刻，浏览器根据这些信息判断缓存是否还有效，如果有效，直接使用缓存文件，并返回200 OK(from disk cache)，如果已经过期，则进一步发出请求向server确定缓存是否真的已过期；
3. server在返回资源的时候会带上Etag和Last-Modified这俩响应头，这样浏览器在本地缓存已过期并向server确认的时候，可以带上If-None-Match（对应Etag）和If-Modified-Since（对应Last-Modified）这俩请求头，server会根据这些信息判断资源是否真的已发生更新，如果是，server返回200 OK，http响应体为更新后的资源，如果否，server返回304 Not Modified，http响应体为空，指示浏览器可以继续使用本地的缓存资源.


