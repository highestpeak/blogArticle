# 状态码

[HTTP 响应代码](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status)

五类状态码（举例了一些常见的状态码）

<u>*todo: 针对举出的例子，详细了解出现的原因和情形，进行解释*</u>

- 信息响应(100–199)
- 成功响应(200–299)
  - 200 - 请求成功
  - 204 - 请求已经成功处理，但是返回的响应报文不包含实体的主体部分。一般在只需要从客户端往服务器发送信息，而不需要返回数据时使用
  - 206 - 表示客户端进行了范围请求，响应报文包含由 Content-Range 指定范围的实体内容。
- 重定向(300–399)
  - 301 - 永久性重定向
  - 302 - 临时性重定向，表示资源临时被分配了新的 URL
  - 304 - Not Modified ：如果请求报文首部包含一些条件，例如：If-Match，If-Modified-Since，If-None-Match，If-Range，If-Unmodified-Since，如果不满足条件，则服务器会返回 304 状态码
- 客户端错误(400–499)
  - 400 - 请求报文存在语法错误
  - 401 - 表示发送的请求需要有通过 HTTP 认证的认证信息
  - 403 - 表示对请求资源的访问被服务器拒绝
  - 404 - 请求的资源（网页等）不存在
- 服务器错误 (500–599)
  - 500 - 内部服务器端在执行请求时发生了错误
  - 503 - 服务器暂时处于超负载或正在进行停机维护，现在无法处理请求

# 首部字段

<u>*todo: 补充这里的表格字段，移除下面大段的文字列表，改为另外一篇文章或者页面显示。*</u>

<u>*todo: 针对下面的标注✨的部分进行进一步阐述*</u>

<u>*todo: 不同首部字段的具体应用：数据转发、内容协商、内容编码、范围请求、分块传输编码、多部分对象集合、虚拟主机等等*</u>

有 4 种类型的首部字段：通用首部字段、请求首部字段、响应首部字段和实体首部字段。

<u>*todo：几种JSON接收发数据的首部字段*</u>

通用首部字段（General Header Fields）：请求报文和响应报文两方都会使用的首部

- Cache-Control 控制缓存 ✨
  - todo：单独条目，字段的取值，
  - 缓存实现：
    - 让代理服务器进行缓存；
    - 让客户端浏览器进行缓存。
  - 优点：
    - 缓解服务器压力；
    - 降低客户端获取资源的延迟：缓存通常位于内存中，读取缓存的速度更快。并且缓存服务器在地理位置上也有可能比源服务器来得近，例如浏览器缓存。
  - todo: 缓存过期机制，缓存验证
- Connection 连接管理、逐条首部 ✨
- Upgrade 升级为其他协议
- via 代理服务器的相关信息
- Wraning 错误和警告通知
- Transfor-Encoding 报文主体的传输编码格式 ✨
- Trailer 报文末端的首部一览
- Pragma 报文指令
- Date 创建报文的日期

请求首部字段（Reauest Header Fields）:客户端向服务器发送请求的报文时使用的首部

- Accept 客户端或者代理能够处理的媒体类型 ✨
- Accept-Encoding 优先可处理的编码格式
- Accept-Language 优先可处理的自然语言
- Accept-Charset 优先可以处理的字符集
- If-Match 比较实体标记（ETage） ✨
- If-None-Match 比较实体标记（ETage）与 If-Match相反 ✨
- If-Modified-Since 比较资源更新时间（Last-Modified）✨
- If-Unmodified-Since比较资源更新时间（Last-Modified），与 If-Modified-Since相反 ✨
- If-Rnages 资源未更新时发送实体byte的范围请求
- Range 实体的字节范围请求 ✨
- Authorization web的认证信息 ✨
- Proxy-Authorization 代理服务器要求web认证信息
- Host 请求资源所在服务器 ✨
- From 用户的邮箱地址
- User-Agent 客户端程序信息 ✨
- Max-Forwrads 最大的逐跳次数
- TE 传输编码的优先级
- Referer 请求原始放的url
- Expect 期待服务器的特定行为

响应首部字段（Response Header Fields）:从服务器向客户端响应时使用的字段

- Accept-Ranges 能接受的字节范围
- Age 推算资源创建经过时间
- Location 令客户端重定向的URI ✨
- vary 代理服务器的缓存信息
- ETag 能够表示资源唯一资源的字符串 ✨
- WWW-Authenticate 服务器要求客户端的验证信息
- Proxy-Authenticate 代理服务器要求客户端的验证信息
- Server 服务器的信息 ✨
- Retry-After 和状态码503 一起使用的首部字段，表示下次请求服务器的时间

实体首部字段（Entiy Header Fields）:针对请求报文和响应报文的实体部分使用首部

- Allow 资源可支持http请求的方法 ✨
- Content-Language 实体的资源语言
- Content-Encoding 实体的编码格式
- Content-Length 实体的大小（字节）
- Content-Type 实体媒体类型
- Content-MD5 实体报文的摘要
- Content-Location 代替资源的yri
- Content-Rnages 实体主体的位置返回
- Last-Modified 资源最后的修改资源 ✨
- Expires 实体主体的过期资源 ✨

> Connection: keep-alive

在早期的HTTP/1.0中，每次http请求都要创建一个连接，而创建连接的过程需要消耗资源和时间，为了减少资源消耗，缩短响应时间，就需要重用连接。

在后来的HTTP/1.0中以及HTTP/1.1中，引入了重用连接的机制，就是在http请求头中加入Connection: keep-alive来告诉对方这个请求响应完成后不要关闭，下一次咱们还用这个请求继续交流。

keep-alive的优点：

- 较少的CPU和内存的使用（由于同时打开的连接的减少了）
- 允许请求和应答的HTTP管线化
- 降低拥塞控制 （TCP连接减少了）
- 减少了后续请求的延迟（无需再进行握手）
- 报告错误无需关闭TCP连接