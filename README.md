# 浏览器HTTP缓存

### 场景

在前端项目中，访问服务器获取数据都是很常见的事情，但是如果相同的数据被重复请求了不止一次，那么多余的请求次数必然会浪费网络带宽，以及浏览器渲染视图的延迟，从而影响用户的使用体验。

如果用户使用的是按量计费的方式访问网络，那么多余的请求还会隐性地增加用户的网络流量资费。

**因此使用缓存技术，通过资源复用，减少了客户端等待时间和网络流量，同时也能缓解服务器端的压力。可以显著的提升我们网站的用户体验**。



### 定义

{% hint style="info" %}
HTTP缓存指的是: 当客户端向服务器请求资源时，会先抵达浏览器缓存，如果浏览器具有“要请求资源”的副本，就可以直接从浏览器缓存中获取而不是从服务器中获取这个资源。
{% endhint %}

~~****~~

### 缓存的种类

缓存的技术种类有很多，比如代理缓存、浏览器缓存、网关缓存、负载均衡器及内容分发网络等

**浏览器HTTP 缓存应该算是前端开发中最常接触的缓存机制之一，它又可细分为强制缓存与协商缓存**。

**二者最大的区别在于判断缓存命中时，浏览器是否需要向服务器端进行询问**。进而判断是否需要就响应内容进行重新请求。

**强制缓存命中的话不会发请求到服务器（比如chrome中的200 from memory cache），协商缓存一定会发请求到服务器（304 not modified）**。

再通过资源的请求首部字段验证资源是否命中协商缓存，如果协商缓存命中，服务器会将这个请求返回，但是不会返回这个资源的实体，而是通知客户端可以从缓存中加载这个资源（304 not modified）

