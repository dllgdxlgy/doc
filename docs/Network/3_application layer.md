## 应用层

### 一、DNS解析过程

+ **主机**向**本地域名服务器**的查询一般都是采用递归查询
+ **本地域名服务器**向**根域名**服务器查询一般都是采用迭代查询。

[DNS原理总结及其解析过程详解](https://www.cnblogs.com/idmask/p/4485836.html)

讲一下DNS过程。给一个网址www.bytedance.com，DNS服务器如何逐级解析的？

[下面内容的网址](https://www.zhihu.com/question/23042131)

1、在浏览器中输入www.qq.com 域名，操作系统会先检查自己本地的hosts文件是否有这个网址映射关系，如果有，就先调用这个IP地址映射，完成域名解析。 

2、如果hosts里没有这个域名的映射，则查找本地DNS解析器缓存，是否有这个网址映射关系，如果有，直接返回，完成域名解析。 

3、如果hosts与本地DNS解析器缓存都没有相应的网址映射关系，首先会找TCP/ip参数中设置的首选DNS服务器，在此我们叫它本地DNS服务器，此服务器收到查询时，如果要查询的域名，包含在本地配置区域资源中，则返回解析结果给客户机，完成域名解析，此解析具有权威性。 

4、如果要查询的域名，不由本地DNS服务器区域解析，但该服务器已缓存了此网址映射关系，则调用这个IP地址映射，完成域名解析，此解析不具有权威性。 

5、如果本地DNS服务器本地区域文件与缓存解析都失效，则根据本地DNS服务器的设置（是否设置转发器）进行查询，如果未用转发模式，本地DNS就把请求发至13台根DNS，根DNS服务器收到请求后会判断这个域名(.com)是谁来授权管理，并会返回一个负责该顶级域名服务器的一个IP。本地DNS服务器收到IP信息后，将会联系负责.com域的这台服务器。这台负责.com域的服务器收到请求后，如果自己无法解析，它就会找一个管理.com域的下一级DNS服务器地址([http://qq.com](https://link.zhihu.com/?target=http%3A//qq.com))给本地DNS服务器。当本地DNS服务器收到这个地址后，就会找[http://qq.com](https://link.zhihu.com/?target=http%3A//qq.com)域服务器，重复上面的动作，进行查询，直至找到www.qq.com主机。 

6、如果用的是转发模式，此DNS服务器就会把请求转发至上一级DNS服务器，由上一级服务器进行解析，上一级服务器如果不能解析，或找根DNS或把转请求转至上上级，以此循环。不管是本地DNS服务器用是是转发，还是根提示，最后都是把结果返回给本地DNS服务器，由此DNS服务器再返回给客户机。 

​    从客户端到本地DNS服务器是属于递归查询，而DNS服务器之间就是的交互查询就是迭代查询。

### 二、Http与Https的区别

![image-20211103203402124](https://gitee.com/dlutlgy/window_typora/raw/master/images/image-20211103203402124.png)

### 三：浏览器输入URL发生了什么？



### 四、HTTP状态码及301和302之间的区别

- [1xx消息](https://zh.wikipedia.org/wiki/HTTP状态码#1xx消息)——请求已被服务器接收，继续处理。

- [2xx成功](https://zh.wikipedia.org/wiki/HTTP状态码#2xx成功)——请求已成功被服务器接收、理解、并接受。

  - 200: 正常返回信息

- [3xx重定向](https://zh.wikipedia.org/wiki/HTTP状态码#3xx重定向)——需要后续操作才能完成这一请求。

  - 300 Multiple Choices
  - **301 Moved Permanently：永久性重定向**。表示请求的资源已经被分配了新的 URI，以后应使用资源现在所指向的 URI。通常会发送 HTTP Location 来重定向到正确的新位置。这是将 HTTP 迁移到 HTTPS 的极佳方法。
  - **302 Found：临时重定向**。表示请求的资源已被分配了新的 URI，希望用户**(本次)**能使用新的 URI 访问。
  - 303 See Other：临时重定向。表示由于请求对应的资源存在着另一个 URI，应使用 **GET** 方法定向获取请求的资源。
  - **304 Not Modified：**表示客户端发送附带条件的请求时，服务器端允许请求访问资源，但未满足条件的情况。304 状态码返回时，不包含任何响应的主体部分。304 虽然被划分在 3XX 类别中，但是和重定向没有关系。

  > 附带条件的请求是指采用 GET 方法的请求报文中包含 If-Match，If-Modified- Since，If-None-Match，If-Range，If-Unmodified-Since 中任一首部。

- [4xx请求错误](https://zh.wikipedia.org/wiki/HTTP状态码#4xx请求错误)——请求含有词法错误或者无法被执行，客户端错误。

  - 401 Unauthorized：请求未经授权，即用户没有权限执行这个请求。
  - 403 Forbidden：服务器收到请求，但是拒绝提供服务
  - 404 Not Found：请求资源不存在。最常见的就是URL错误

- [5xx服务器错误](https://zh.wikipedia.org/wiki/HTTP状态码#5xx服务器错误)——服务器在处理某个正确请求时发生错误

  关于服务器500 501 502 503 504 505 的一些详解：https://jingyan.baidu.com/article/63f2362812cc600208ab3dce.html

  - 500 Internal Server Error：表明服务器端在执行请求时发生了错误，有可能是 Web 应用存在的 bug 或某些临时的故障。
  - 502 Bad Gateway：作为网关或者代理工作的服务器尝试执行请求时，从上游服务器接收了无效的响应。
  - 503 Service Unavailable：服务器暂时处于超负载或正在进行停机维护，现在无法处理请求。
  - 504 Gateway Time-out 这是代表着网关超时是现象出现了。是指服务器作为网关或代理，但是没有及时从上游服务器收到请求。







