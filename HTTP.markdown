# 参考资料
* [wikipedia：统一资源标志符](https://zh.wikipedia.org/wiki/%E7%BB%9F%E4%B8%80%E8%B5%84%E6%BA%90%E6%A0%87%E5%BF%97%E7%AC%A6)
* [wikipedia: URL](https://en.wikipedia.org/wiki/URL)
* [rfc2616：3.2.2 http URL](https://www.w3.org/Protocols/rfc2616/rfc2616-sec3.html#sec3.2.2)
* [What is the difference between a URI, a URL and a URN?](https://stackoverflow.com/questions/176264/what-is-the-difference-between-a-uri-a-url-and-a-urn)
* [rfc2616：9 Method Definitions](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html)
* [MDN : HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP)
* [HTTP/2 简介](https://developers.google.com/web/fundamentals/performance/http2/?hl=zh-cn)
* [htmlspecialchars](http://php.net/manual/zh/function.htmlspecialchars.php)
* [Difference between file URI and URL in java](http://java2db.com/java-io/how-to-get-and-the-difference-between-file-uri-and-url-in-java)
* [How to Fix SQL Injection Using Java PreparedStatement & CallableStatement](https://software-security.sans.org/developer-how-to/fix-sql-injection-in-java-using-prepared-callable-statement)
* [浅谈 HTTP 中 Get 与 Post 的区别](https://www.cnblogs.com/hyddd/archive/2009/03/31/1426026.html)
* [Are http:// and www really necessary?](https://www.webdancers.com/are-http-and-www-necesary/)
* [HTTP (HyperText Transfer Protocol)](https://www.ntu.edu.sg/home/ehchua/programming/webprogramming/HTTP_Basics.html)
* [Web-VPN: Secure Proxies with SPDY & Chrome](https://www.igvita.com/2011/12/01/web-vpn-secure-proxies-with-spdy-chrome/)
* [File:HTTP persistent connection.svg](http://en.wikipedia.org/wiki/File:HTTP_persistent_connection.svg)
* [Proxy server](https://en.wikipedia.org/wiki/Proxy_server)
* [What Is This HTTPS/SSL Thing And Why Should You Care?](https://www.x-cart.com/blog/what-is-https-and-ssl.html)
* [What is SSL Offloading?](https://securebox.comodo.com/ssl-sniffing/ssl-offloading/)
* [Sun Directory Server Enterprise Edition 7.0 Reference - Key Encryption](https://docs.oracle.com/cd/E19424-01/820-4811/6ng8i26bn/index.html)
* [An Introduction to Mutual SSL Authentication](https://www.codeproject.com/Articles/326574/An-Introduction-to-Mutual-SSL-Authentication)
* [The Difference Between URLs and URIs](https://danielmiessler.com/study/url-uri/)
* [Cookie 与 Session 的区别](https://juejin.im/entry/5766c29d6be3ff006a31b84e#comment)
* [COOKIE 和 SESSION 有什么区别](https://www.zhihu.com/question/19786827)
* [Cookie/Session 的机制与安全](https://harttle.land/2015/08/10/cookie-session.html)
* [HTTPS 证书原理](https://shijianan.com/2017/06/11/https/)
* [What is the difference between a URI, a URL and a URN?](https://stackoverflow.com/questions/176264/what-is-the-difference-between-a-uri-a-url-and-a-urn)
* [XMLHttpRequest](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest)
* [XMLHttpRequest (XHR) Uses Multiple Packets for HTTP POST?](https://blog.josephscott.org/2009/08/27/xmlhttprequest-xhr-uses-multiple-packets-for-http-post/)
* [Symmetric vs. Asymmetric Encryption – What are differences?](https://www.ssl2buy.com/wiki/symmetric-vs-asymmetric-encryption-what-are-differences)
* [Web 性能优化与 HTTP/2](https://www.kancloud.cn/digest/web-performance-http2)
* [HTTP/2 简介](https://developers.google.com/web/fundamentals/performance/http2/?hl=zh-cn)


# 基础概念
## 请求和响应报文
请求报文结构：
* 首行包括请求类型、URL和协议版本
* 后续多行为请求首部 Header，每个首部都有一个首部名称，以及对应值
* 空行用于分割首部和请求主体 Body

      GET http://www.example.com/ HTTP/1.1
      Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
      Accept-Encoding: gzip, deflate
      Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
      Cache-Control: max-age=0
      Host: www.example.com
      If-Modified-Since: Thu, 17 Oct 2019 07:18:26 GMT
      If-None-Match: "3147526947+gzip"
      Proxy-Connection: keep-alive
      Upgrade-Insecure-Requests: 1
      User-Agent: Mozilla/5.0 xxx

      param1=1&param2=2

响应报文结构：
* 首行包含协议版本，状态码及描述
* 后续多行为首部内容
* 空行分割首部和内容主体

      HTTP/1.1 200 OK
      Age: 529651
      Cache-Control: max-age=604800
      Connection: keep-alive
      Content-Encoding: gzip
      Content-Length: 648
      Content-Type: text/html; charset=UTF-8
      Date: Mon, 02 Nov 2020 17:53:39 GMT
      Etag: "3147526947+ident+gzip"
      Expires: Mon, 09 Nov 2020 17:53:39 GMT
      Keep-Alive: timeout=4
      Last-Modified: Thu, 17 Oct 2019 07:18:26 GMT
      Proxy-Connection: keep-alive
      Server: ECS (sjc/16DF)
      Vary: Accept-Encoding
      X-Cache: HIT

      <!doctype html>
      <html>
        <head>
          <title>Example Domain</title>
	         // 省略...
          </body>
      </html>

## URL (Uniform Resource Locator, 统一资源定位符)
URL 是 URI (Uniform Resource Identifier, 统一资源标识符) 的子集，在 URI 的基础上增加了定位能力。URI 还包含 URN (Uniform Resource Name，统一资源名称)，URN 用来定义资源名称但是无法定位资源


# HTTP 方法
## GET
> 获取资源，绝大部分网络请求都是 GET 请求

## HEAD
> 和 GET 类似，但是返回主体部分，用于确认 URL 有效性和资源更新的日期等

## POST
> 传输实体主体，GET 主要是获取，POST 则是双向传输

## PUT
> 上传文件，自身不带验证机制，有安全隐患，一般不用

## PATCH
> 对资源进行部分修改

## DELETE
> 删除文件，也不代表验证机制

## OPTIONS
> 查询指定 URL 支持的方法类型

## CONNECT
> 要求与代理服务器通信时要建立隧道，使用 SSL (Secure Sockets Layer,安全套接层) 和 TLS (Transport Layer Security, 传输层安全) 协议把通信内容加密后经过网络隧道传输。

## TRACE
> 追踪路径，服务器将通信路径返回给客户端，需要在 Max-Forwards 首部字段中填入数值，经过一个服务器 -1 ， 数值为 0 停止传输。   
通常不用，容易受到 XST (Cross-Site Tracing，跨站追踪) 攻击。


# HTTP 状态
      2xx：成功
      200  OK  请求成功，后面是对GET或POST请求的应答文档
      201  Created  请求被创建完成，新的资源文件已被创建
      202  Accepted  请求已被接受，但是处理未完成
      203  Non Information  返回文档的拷贝，应答头可能不正确。
      204  Non Content  没有新文档，浏览器应该继续显示原来的文档。如果用户定期刷新页面，而Servlet确定用户目前的文档足够新，则使用该响应。
      205  Reset Content  没有新文档，但是浏览器要重置现实的内容。用来强制浏览器清除表单输入内容。
      206  Partial Content  客户发送了一个带有Range头的Get请求，服务器满足了他。

      3xx：重定向
      300  Multiple Choices  多重选择，用户可以选择某链接到目标，最多五个选择
      301  Moved permanently  页面已被转移到新的url
      302  Found  页面暂时转移至新的url
      303  See Other  页面可在别的url被找到
      304  Not Modified  未按预期修改文档。客户端缓冲的文档并发出了一个条件性请求（一般是提供if-modified-since头表示客户只想比指定日期更新的文档），服务器告诉用户缓冲的文档还可以使用。
      305  Use Proxy  客户请求的文档应该通过Location头所指的代理服务器提取
      306  Unused  协议上已经不再使用，但是代码还在
      307  Temporary Redirect  被请求的页面已经临时转移至新的url。

      4xx：客户端错误
      400  Bad Request  服务器未能理解请求
      401  Unauthorized  被请求的页面需要用户名和密码
      402  Payment Required  尚无法使用
      403  Forbidden  对被请求页面的访问被禁止
      404  Not Found  服务器无法找到被请求的页面
      405  Method Not Allowed  请求中指定的方法不被允许
      406  Not Acceptable  服务器生成的相应无法被客户端所接受
      407  Proxy Authentication Required  用户必须首先使用代理服务器执行验证后，请求才会被处理
      408  Request Timeout  请求超出了服务器的等待时间
      409  Conflict  冲突导致请求无法完成
      410  Gone  请求页面不可用
      411  Length Required  Content-Length未定义，如果没有此内容，服务器不会接受请求。
      412  Precondition Failed  请求的前提条件被服务器评估为失败
      413  Request Entity Too Large  由于请求的实体太大，服务器不会接受请求
      414  Request-url Too Long  由于url太长，服务器不会接受请求。当post请求转换为get请求时可能会出现这种情况
      415  Unsupported Media Type  由于媒介类型不被支持，服务器不会接受请求
      416  Requested Range Not Satisfiable  服务器不能满足客户在请求中指定的Range头
      417  Expectation Failed  执行失败
      423  锁定的错误

      5xx：服务器错误
      500  Internal Server Error  请求未完成。服务器遇到不可预测的问题
      501  Not Implemented  请求未完成。服务器不支持所请求的功能
      502  Bad GetWay  请求未完成。服务器从上有服务器接受到一个无效的响应
      503  Service Unavaliable  请求未完成。服务器临时过载或宕机
      504  Getway Timeout  网关超时
      505  Http  Version Not Supported  服务器不支持请求中指明的HTTP协议版本


# HTTP 首部
有 4 种首部字段：通用首部字段、请求首部字段、响应首部字段 和 实体首部字段

## 通用首部字段
名称|说明
:--:|:-:
Cache-Control|	控制缓存的行为
Connection|	控制不再转发给代理的首部字段、管理持久连接
Date|	创建报文的日期时间
Pragma|	报文指令
Trailer|	报文末端的首部一览
Transfer-Encoding|	指定报文主体的传输编码方式
Upgrade|	升级为其他协议
Via|	代理服务器的相关信息
Warning|	错误通知

## 请求首部字段
名称|说明
:--:|:-:
Accept|	用户代理可处理的媒体类型
Accept-Charset|	优先的字符集
Accept-Encoding|	优先的内容编码
Accept-Language|	优先的语言（自然语言）
Authorization	Web| 认证信息
Expect|	期待服务器的特定行为
From|	用户的电子邮箱地址
Host|	请求资源所在服务器
If-Match|	比较实体标记（ETag）
If-Modified-Since|	比较资源的更新时间
If-None-Match|	比较实体标记（与 If-Match 相反）
If-Range|	资源未更新时发送实体 Byte 的范围请求
If-Unmodified-Since|	比较资源的更新时间（与 If-Modified-Since 相反）
Max-Forwards|	最大传输逐跳数
Proxy-Authorization|	代理服务器要求客户端的认证信息
Range|	实体的字节范围请求
Referer|	对请求中 URI 的原始获取方
TE|	传输编码的优先级
User-Agent|	HTTP 客户端程序的信息

## 响应首部字段
名称|说明
:--:|:-:
Accept-Ranges||	是否接受字节范围请求
Age|	推算资源创建经过时间
ETag|	资源的匹配信息
Location|	令客户端重定向至指定 URI
Proxy-Authenticate|	代理服务器对客户端的认证信息
Retry-After|	对再次发起请求的时机要求
Server|	HTTP 服务器的安装信息
Vary|	代理服务器缓存的管理信息
WWW-Authenticate|	服务器对客户端的认证信息

## 实体首部字段
名称|说明
:--:|:-:
Allow|	资源可支持的 HTTP 方法
Content-Encoding|	实体主体适用的编码方式
Content-Language|	实体主体的自然语言
Content-Length|	实体主体的大小
Content-Location|	替代对应资源的 URI
Content-MD5|	实体主体的报文摘要
Content-Range|	实体主体的位置范围
Content-Type|	实体主体的媒体类型
Expires|	实体主体过期的日期时间
Last-Modified|	资源的最后修改日期时间

# 具体应用
## 连接管理

![](assets/HTTP-4278eff6.png)

### 1.短连接和场链接
当浏览器访问一个包含多张图片的 HTML 时，除了请求页面还要请求图片资源。如果获取一次资源就建立一次 TCP 连接，开销很大。    
长连接只需要进行一次 TCP 连接就能进行多次 HTTP 通信。
* 1.1 之后默认是长连接，如果断开连接需要服务器或客户端主动断开，使用 Connection:close;
* 1.1 之前默认是短连接，如果使用长连接，使用 Conncection:keep-Alive

### 2.流水线
流水线是指同一条长连接上连续发出请求，而不用等待响应返回，这样可以减少延迟。

## Cookie
HTTP 协议是无状态的。 HTTP/1.1 引入 Cookie 来保存状态信息。   
Cookie 是服务器发送到用户浏览器并保存在本地的一小块数据，它会在浏览器之后向同一服务器再次发起请求时被携带上，用于告知服务端两个请求是否来自同一浏览器。由于每次请求都携带 cookie 会带来额外的性能开销。   
Cookie 层一度用于客户端数据的存储，因为当时并没有其他合适的存储办法而作为唯一的存储手段，但现在新的浏览器 API 已经允许开发者将数据存储在本地，如使用 Web storage API（本地存储和会话存储） 或 IndexedDB

### 1.用途
* 会话状态管理
* 个性化设置
* 浏览器行为追踪

### 2.创建过程
服务器发送的响应报文包含 Set-Cookie 首部字段，客户端得到响应报文后把 Cookie 内容保存到浏览器中。

      HTTP/1.0 200 OK
      Content-type: text/html
      Set-Cookie: yummy_cookie=choco
      Set-Cookie: tasty_cookie=strawberry

      [page content]

### 3.分类
* 会话器 Cookie：浏览器关闭之后他会被自动删除，仅在会话期有效
* 持久性 Cookie：指定过期时间 (Expires) 后有效期 (max-age) 之后就成为了持久性 Cookie。

      Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT;

### 4.作用域
**Domain** 标识指定了那些主机可以接收 Cookie。如果不指定默默认为当前文档的主机 (不包含子域名)。如果指定了，则一般包含子域名。   
**Path** 标识指定了哪些路径可以接收 Cookie (改 URL 路径必须存在于请求 URL 中)。 以字符 %x2F("/") 作为分隔符，子路径也会被匹配。

### 5.JavaScript
浏览器通过 document.cookie 属性可创建新的 Cookie，也可通过该属性访问非 HttpOnly 标记的 Cookie。

      document.cookie = "yummy_cookie=choco";
      document.cookie = "tasty_cookie=strawberry";
      console.log(document.cookie);

### 6.HttpOnly
HttpOnly 标识代表 Cookie 不能被 JavaScript 脚本调用。跨站脚本攻击 (XSS) 常使用 JavaScript 的 document.cookie API 剽窃用户的 Cookie 信息。

      Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT; Secure; HttpOnly

### 7.Secure
标记为 Secure 的 Cookie 只能通过被 Https 协议加密过的请求。但即使加密了，也不应该通过 Cookie 传递敏感信息。

### 8.Session
除了 Cookie 存储信息到浏览器， 也可以使用 Session 存储信息到服务器，相比之下 Session 更加安全。   
Session 既可以存在服务器的物理文件上，也可以存在 Redis 这种内存型数据库上，效率更高。    
使用 Session 维护用户登录状态的过程如下：
* 用户登录时，用户提交包含账号密码的表单，放入 HTTP 请求中
* 服务器验证账号密码，通过后将账户信息存储到 Redis 中，在 Redis 中的 key 被称为 sessionId
* 服务器返回的响应报文的 Set-Cookie 首部字段包含了这个 sessionId ,客户端收到响应报文后将 Cookie 传入浏览器
* 客户端之后对同一个服务器进行请求时会包含该 Cookie，服务器收到后提取出 sessionId，从 Redis 中取出账户信息，继续业务操作。

### 9.浏览器禁用 Cookie
此时无法使用 Cookie 存储用户信息，而 Session 可以存储任何类型的数据。不能用 Cookie 传递 SessionId 转用 URL 重写技术，将 SessionId 作为 URL 的参数进行传递。

### 10.Cookie 和 Session的选择
* Cookie 只能存储 ASCII 码字符串，Session 可以存储任何类型数据，考虑数据复杂性，首选 Session
* Cookie 存在浏览器，容易被恶意获取，非要存在 Cookie 可以将 Cookie 加密，放到服务器解密
* 所有用户数据都存在 Session 对于大型网站开销太大，因此不建议这么做


## 缓存
### 1.优点
* 缓解服务器压力
* 降低客户端延迟

### 2.实现
* 让代理服务器缓存
* 让客户端自行缓存

### 3.Cache-Control
HTTP/1.1 通过 Cache-Control 首部字段来控制缓存
#### 3.1 禁止缓存
no-store 指令规定不能对请求或响应任何一部分进行缓存

#### 3.2 强制确认缓存
no-cache 指令规定缓存服务器要向源服务器验证缓存资源的有效性，只有当缓存有效才能对请求进行响应   

#### 3.3 私有缓存和公共缓存
private 指令规定了资源作为私有缓存，只能被单独用户使用，一般存储在用户浏览器中。    
public 指令规定了资源作为公共缓存，可以被多个用户使用，一般存储在代理服务器中。

#### 3.4 缓存过期机制
max-age 指令出现在请求报文，并且缓存资源的缓存时间小于该指令指定的时间，那么就能接受该缓存。    
max-age 指令出现在响应报文，表示缓存资源在缓存服务器中保存的时间。   
Expires 首部字段也可以用于告知缓存服务器该资源什么时候过期。
* 在 1.1 中，优先处理 max-age 指令
* 在 1.0 中，max-age指令会被忽略掉

#### 4.缓存验证
ETag 字段是资源的唯一标识。URL 不能唯一表示资源，例如 www.google.com 有中文 和 英文 两个资源，只有 ETag 才能对两个资源进行唯一标识。   
可以将缓存资源的 ETag 值放入 If-None-Match 首部，服务器收到该请求后，判断缓存资源的 ETag 值和资源的最新 ETag 值是否一致，一致则表示缓存资源有效，返回 304 。   
Last-Modified 首部字段也可用于缓存验证，它包含在源服务器发送的响应报文中，指示源服务器对资源最后修改时间。但是精确度只能到秒级别，如果响应中包含该字段，客户端可以在请求中加上 If-Modifieed-Since 来验证缓存。服务器只在所请求的资源在给定的日期后，对内容进行过修改才会返回资源，否则返回一个不带实体的 304 响应。


## 内容协商
通过内容协商返回最合适的内容，例如根据浏览器的默认语言选择返回中文界面还是英文界面。
### 1. 类型
#### 1.1 服务端驱动型

客户端设置特定的 HTTP 首部字段，例如 Accept、Accept-Charset、Accept-Encoding、Accept-Language，服务器根据这些字段返回特定的资源。

它存在以下问题：

* 服务器很难知道客户端浏览器的全部信息；
* 客户端提供的信息相当冗长（HTTP/2 协议的首部压缩机制缓解了这个问题），并且存在隐私风险（HTTP 指纹识别技术）；
* 给定的资源需要返回不同的展现形式，共享缓存的效率会降低，而服务器端的实现会越来越复杂。


#### 1.2 代理驱动型
服务器返回 300 Multiple Choices 或者 406 Not Acceptable，客户端从中选出最合适的那个资源。

### 2. Vary
>Vary: Accept-Language

在使用内容协商的情况下，只有当缓存服务器中的缓存满足内容协商条件时，才能使用该缓存，否则应该向源服务器请求该资源。

例如，一个客户端发送了一个包含 Accept-Language 首部字段的请求之后，源服务器返回的响应包含 Vary: Accept-Language 内容，缓存服务器对这个响应进行缓存之后，在客户端下一次访问同一个 URL 资源，并且 Accept-Language 与缓存中的对应的值相同时才会返回该缓存。

## 内容编码
内容编码将实体主体进行压缩，从而减少传输的数据量。

常用的内容编码有：gzip、compress、deflate、identity。

浏览器发送 Accept-Encoding 首部，其中包含有它所支持的压缩算法，以及各自的优先级。服务器则从中选择一种，使用该算法对响应的消息主体进行压缩，并且发送 Content-Encoding 首部来告知浏览器它选择了哪一种算法。由于该内容协商过程是基于编码类型来选择资源的展现形式的，响应报文的 Vary 首部字段至少要包含 Content-Encoding。

## 范围请求
如果网络出现中断，服务器只发送了一部分数据，范围请求可以使得客户端只请求服务器未发送的那部分数据，从而避免服务器重新发送所有数据。

### 1. Range
在请求报文中添加 Range 首部字段指定请求的范围。

      GET /z4d4kWk.jpg HTTP/1.1
      Host: i.imgur.com
      Range: bytes=0-1023
      请求成功的话服务器返回的响应包含 206 Partial Content 状态码。

      HTTP/1.1 206 Partial Content
      Content-Range: bytes 0-1023/146515
      Content-Length: 1024
      ...
      (binary content)


### 2. Accept-Ranges
响应首部字段 Accept-Ranges 用于告知客户端是否能处理范围请求，可以处理使用 bytes，否则使用 none。
>Accept-Ranges: bytes

### 3. 响应状态码
* 在请求成功的情况下，服务器会返回 206 Partial Content 状态码。
* 在请求的范围越界的情况下，服务器会返回 416 Requested Range Not Satisfiable 状态码。
* 在不支持范围请求的情况下，服务器会返回 200 OK 状态码。


## 分块传输编码
Chunked Transfer Encoding，可以把数据分割成多块，让浏览器逐步显示页面。

## 多部分对象集合
一份报文主体内可含有多种类型的实体同时发送，每个部分之间用 boundary 字段定义的分隔符进行分隔，每个部分都可以有首部字段。

例如，上传多个表单时可以使用如下方式：

      Content-Type: multipart/form-data; boundary=AaB03x

      --AaB03x
      Content-Disposition: form-data; name="submit-name"

      Larry
      --AaB03x
      Content-Disposition: form-data; name="files"; filename="file1.txt"
      Content-Type: text/plain

      ... contents of file1.txt ...
      --AaB03x--


## 虚拟主机
HTTP/1.1 使用虚拟主机技术，使得一台服务器有多个域名，可以看做多个服务器

## 通信数据转发
### 1. 代理
代理服务器接收客户端的请求，并且转发给其他服务器，目的是：
* 缓存
* 负载均衡
* 网络访问控制
* 访问日志记录

代理服务器分为正向和反向代理两种：
* 用户察觉得到正向代理的存在

![](assets/HTTP-935d068a.png)

* 而反向代理一般位于内部网络中，用户察觉不到

![](assets/HTTP-8efb4b8c.png)

### 2.网关
与代理服务器不同的是，网管服务器是将 HTTP 转换成其他协议进行通信，从而请求其它非 HTTP 服务器的服务。

### 3.隧道
使用 SSL 等加密手段，在客户端和服务器建立安全的通信线路


# HTTPS
HTTP 具有以下安全问题:
* 使用明文进行通信，内容会被窃听
* 不验证通信方身份，通信方身份可能被伪装
* 无法证明报文的完整性，报文可能被篡改

HTTPS 不是新协议，是 HTTP 先与 SSL 通信，再经由 SSL 与 TCP 进行通信。

![](assets/HTTP-1cdc7b22.png)

## 加密
### 1.对称密钥加密
该加密方法，加密解密用的是一套密钥
* 优点：运算速度快
* 缺点：无法安全地将密钥传输给通信方

![](assets/HTTP-17f2d72b.png)

### 2.非对称密钥加密
又称公开密钥加密，加密和解密使用不同的密钥。    
公开密钥所有人都能获得，发送发获得接收方提供的公钥后就可以使用公钥进行加密，接收方收到后通过私有密钥进行解密。   
该加密方式还可用来进行签名，因为私有密钥无法被他人获取，发送方可以使用私钥进行签名，接收方通过发送方传递的公钥进行解密，判断签名是否正确。   
* 优点：安全传输
* 缺点：传输速度慢

![](assets/HTTP-05d68510.png)

### 3.HTTPS 加密
HTTPS 采用混合加密机制，利用非对称密钥加密将对称密钥传输给对方。
* 使用非对称加密方式对对称密钥加密，保证安全
* 获取对策密钥后，使用对称密钥进行传输，保证效率

![](assets/HTTP-bfa20882.png)

## 认证
通过使用 证书 来对通信双方认证。   
数字证书认证机构 (CA, Certificate Authority) 是客户和服务器都信赖的第三方机构。    
服务器向 CA 发起公开密钥申请， CA 判断申请者身份后，对已申请的公开密钥做数字签名，然后分配这个已签名的公开密钥，将公开密钥放入证书中绑定在一起。    
进行 HTTPS 通信时，服务器会将证书发送给客户端。客户端取得证书中的密钥后，先用数字签名进行认证，认证通过则可以通信。

![](assets/HTTP-70735c9f.png)

## 完整性保护
SSL 提供报文摘要进行完整性保护。    
HTTP 也提供了 MD5 报文摘要功能，不够安全，内容如果被篡改同时重新计算 MD5 的值， 接收方不会感知到      
HTTPS 摘要有加密和认证两个操作，相对安全

## HTTPS 缺点
* 因为要加密和解密，速度相对较慢
* 证书认证需要额外费用

## HTTPS通信整体流程

![](assets/HTTP-73ba6d39.png)



# HTTP/2.0
## HTTP/1.x 缺陷
* 客户端使用多个连接才能并发和缩短延迟
* 不会压缩请求和响应的首部，从而导致不必要的流量开销
* 不支持资源的优先级，导致 TCP 连接利用率低下

## 二进制分帧层
HTTP/2.0 将报文分成 HEADERS 帧和 DATA 帧，他们都是二进制格式。

![](assets/HTTP-f8968215.png)

通信过程中，只有一个 TCP 连接存在，它承载了任意数量的双向数据流 (Stream) 。
* 一个数据流 (Stream) 都有一个唯一标识符和可选的优先级信息，用于承载双向信息
* 消息 (Message) 是与逻辑请求或相应对应的完整的一系列帧
* 帧 (Frame) 是最小的通信单位，来自不同数据流的帧可以交错发送，然后根据每个帧头的数据流标识符重新组装

![](assets/HTTP-ee22ecfb.png)

## 服务端推送
HTTP/2.0 在客户端请求一个资源时，会把相关资源一起发送给客户端，例如请求 HTML 页面，服务端会将 JS 文件和 CSS 文件 和 JPG 文件等等一起发送给客户端。

![](assets/HTTP-62f9ab6d.png)

## 首部压缩
HTTP/2.0 要求客户端和服务器同时维护和更新一个包含之前见过的首部字段表，避免重复传输。   
不仅如此，还通过 Huffman 编码对首部字段进行压缩。   
![](assets/HTTP-49f7b21f.png)

# HTTP/1.1 新特性

* 默认长连接
* 支持流水线
* 支持多个 TCP 连接
* 支持虚拟主机
* 新增状态码 100
* 支持分块传输编码
* 新增缓存处理指令 max-age


# GET 和 POST 比较
## 作用
GET 用于获取资源，而 POST 用于传输实体主体。

## 参数
GET 和 POST 的请求都能使用额外的参数。但 GET 是显式拼接在 URL 中 ， POST 存储在实体主体里。 POST 请求可以通过抓包工具 (类如 Fillder) 查看。    
因为 URL 只支持 ASCII 码，因此 GET 请求的参数需要进行编码。

## 安全
安全的 HTTP 方法不会改变服务器状态，它是只读的。   
GET 是安全的，POST 不是，因为 POST 传送实体主体类容，如果 POST 传送了表单数据，那服务器可能要把数据存入数据库中，状态则发生了变化。    
安全的方法还包括 HEAD、OPTIONS。
不安全的方法包括 PUT、DELETE。

## 幂等性
幂等的 HTTP 方法执行一次和连续执行多次的效果是一样的。    
所有安全方法都是幂等的。    
正确实现下 PUT 和 DELETE 也是幂等的，而 POST 不是。   
例如 POST /add_row HTTP/1.1 不是幂等的，如果调用多次，就会增加多行记录：

    POST /add_row HTTP/1.1   -> Adds a 1nd row
    POST /add_row HTTP/1.1   -> Adds a 2nd row
    POST /add_row HTTP/1.1   -> Adds a 3rd row

DELETE /idX/delete HTTP/1.1 是幂等的，即使不同的请求接收到的状态码不一样：

    DELETE /idX/delete HTTP/1.1   -> Returns 200 if idX exists
    DELETE /idX/delete HTTP/1.1   -> Returns 404 as it just got deleted
    DELETE /idX/delete HTTP/1.1   -> Returns 404

## 可缓存
如果对响应进行缓存，需要满足以下条件：
* 请求报文的 HTTP 方法本身可缓存，除了 POST 和 DELETE。 (少数情况下可缓存 POST)
* 响应报文的状态码是可缓存的，包括： 200，203，204，206，300，304，301，404，405，410，414，501
* 响应报文的 Cache-Control 首部字段没有指定不进行缓存。


## XMLHttpRequest
为了阐述 POST 和 GET 的另一个区别，需要先了解 XMLHttpRequest：

>XMLHttpRequest 是一个 API，它为客户端提供了在客户端和服务器之间传输数据的功能。它提供了一个通过 URL 来获取数据的简单方式，并且不会使整个页面刷新。这使得网页只更新一部分页面而不会打扰到用户。XMLHttpRequest 在 AJAX 中被大量使用。

* 在使用 XMLHttpRequest 的 POST 方法时，浏览器会先发送 Header 再发送 Data。但并不是所有浏览器会这么做，例如火狐就不会。
* 而 GET 方法 Header 和 Data 会一起发送。
