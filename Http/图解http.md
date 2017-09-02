# 图解HTTP整理

## 目录

1. 了解Web及网络基础
2. 简单的HTTP协议
3. HTTP报文内的HTTP信息
4. **返回结果的HTTP状态码**
5. 与HTTP协作的Web服务器
6. **HTTP首部**
7. **确保Web安全的HTTPS**
8. 确认访问用户身份的认证
9. 基于HTTP的功能追加协议
10. 构建Web内容的技术
11. **Web的攻击技术**



## 了解Web及网络基础

HTTP是位于应用层的协议，目前使用版本基本上都是HTTP 1.1

URI(Uniform Resource Identifier)统一资源标识符，URL(Uniform Resource Location)统一资源定位符，前者侧重于一种抽象的，高层次的概念，突出的是一种标识符，而后者则侧重的是位置location的概念 。URL其实也可以看作是一种URI

URI格式` 协议方案名://登录信息(username:passwd)@服务器地址:端口号/带` 

## 简单的HTTP协议

请求报文

```
方法	URI	协议版本
请求报文首部字段
//空行
内容实体
```

响应报文

```
协议版本 状态码	状态码的原因短语
响应报文首部字段
//空行
内容实体
```

HTTP是一种不保存状态，自身不对请求和响应之间的通信状态进行保存，HTTP这个级别，协议对于发送过的请求和响应都不做持久化处理

HTTP协议使用URI定位互联网上的资源，可以通过完整的URI来定位`http://hacker.jp/index.html`也可以通过首部字段Host中写明网络域名或者IP地址`/index.html `,`Host: hacker.jp`

HTTP方法

- GET获取资源，告知服务器该客户端请求某个资源，后面接所需的URI
- POST传输实体主体，主要作用是告知服务器实体的内容
- PUT传输文件，把实体的内容传输到URI的位置，由于存在安全性问题(无验证)，一般的Web网站不采用这个方法
- HEAD获得报文首部，不获取报文主体部分，只是返回报文头
- DELETE，删除服务器上的URI对应的文件
- OPTIONS，询问服务器支持哪些方法
- TRACE，追踪路径，和tracert方法有点类似，都是维护一个变量，设个初始值，每经过一个代理服务器就减一，并记录服务器信息，为0时就返回
- CONNECT，告知服务器会用隧道协议进行连接代理，CONNECT方法只对代理服务器有效，用得最多的是SSL和TLS

持久连接

- 持久连接，HTTP是不保存状态的，这样也就意味着每一次使用HTTP协议都要进行3次TCP握手，这样做是很划不来的，所以现在的HTTP协议基本上都实现了默认持久连接，建立TCP连接之后，一定时间内不需要再次建立连接了
- 管线化，本来HTTP协议，发送请求之后都需要等待并收到相应才能发送下一个请求，现在可以在上一个请求还没有得到相应的时候就发送另一个请求。



## HTTP报文内的HTTP信息

HTTP报文结构

```
请求行(客户端)/相应行(服务器)
报文首部
空行(CRLF)
报文信息
```

发送多种数据

- multipart/form-data

  主要用来请求不同的文件

  ```
  Content-type: multipart/form-data; boundary:AAA
  -- AAA
  Content-Depostion:form-data;name = 'field1'
  主要内容

  -- AAA
  Content-Depostion:form-data;name = "field2"
  主要内容
  ```

  ​

- multipart/byteranges

  主要是同一个文件中的不同部分

  ```
  Content-type : multipart/byterangers; boundary: BOUNDARY

  -- BOUNDARY
  Content-type : application/pdf
  Content-Range : bytes 500-800/1000

  --BOUNDARY
  Content-type : application / pdf
  Content-Range : bytes 100-200/1000

  ```

  ​

## 返回结果的HTTP状态码

- 2XX

  执行成功

  - 200 OK，执行成功
  - 204 No Content，没有返回结果，但执行成功
  - 206 Partial Content，当get请求的是部分文件时，成功返回这部分文件

- 3XX

  需要重定向

  - 301 Moved Permantly，永久性重定向，资源发生了永久转移，如果书签里面存有原来的地址需要转成现在的地址
  - 302 Found，临时性重定向，资源发生了临时性的重定向，但不会改书签地址，因为这个地址后面可能还会发生改变
  - 303 See Other，和302类似
  - 304 Not Modified，和重定向无关，一般是Get请求中有条件，但是服务器找到了资源却没有符合条件，就返回这个状态
  - 307 Temporary Redirect，和302类似

- 4XX

  访问错误

  - 400 Bad Request，有语法错误
  - 401 Unauthorized，没有认证，如果之前有认证过就说明认证失败了
  - 403 Forbidden，不允许访问该资源
  - 404 Not Found，没有该资源

- 5XX

  服务器错误

  - 500 Server Internal Error，服务器执行查找时发生错误
  - 503 Server Unavailable

## 与HTTP协作的Web服务器

- 单台虚拟机实现多个域名

  一台虚拟机可以实现多个域名，所以在发请求时需要明确URI

- 通信数据转发程序: 代理、网关、隧道

  - 代理

    代理的作用是在客户端和服务器之间起到一个中转的作用，客户端发送请求到代理服务器，这个代理服务器再发送给下一个代理服务器，直到发送到目的服务器。每次代理服务器转发时，不会改变URI，只会增加转发服务器的记录。代理服务器的分类主要有两种，缓存代理和透明代理，缓存代理中会暂时存储资源，当再次请求资源时就会直接从代理服务器上获取，而透明代理则不会

  - 网关

    网关是内网和外网连接通信的服务器， 内网传输数据时会把数据先发送给网关，由网关来进行转发

  - 隧道

    让客户端和服务器直接相连，同时使用SSL等方式来进行加密通信，隧道不会对数据进行处理，而是直接进行数据传输

- 保存资源的缓存

  - 缓存的有效期限

    获取资源时，需要判断缓存的有效性。

  - 客户端的缓存

## HTTP首部

HTTP首部由字段名和字段值组成，两者之间由":"隔开

四种首部字段

- 通用字段
  - Cache-Control：控制缓存操作，如no-cache，强制向源服务器再次验证，no-store不换存请求等
  - Connection，主要有两个作用
    - 控制不再转发的首部字段，如upgrade，客户端中的upgrade字段不会再被服务器转发了
    - 管理持久连接，如close, keep-alive
  - Date：日期
  - Pragma：历史遗留问题和no-catch的功能相似，是1.0的用法
  - Trailer：说明报文主体的后面还有字段
  - Transfer-Encoding：规定传输报文主体时采用的编码方式
  - Upgrade：用来检测HTTP协议以及其他协议是否有可用的更高版本的进行通信
  - Via：代理服务器转发的记录
  - Warning：`Warning:[警告码][警告主机：端口][警告内容][时间]`，警告码有100 警告已过期,111再验证失败,112断开连接
- 请求字段
  - Accept：声明客户端可以接受的资源种类
  - Accept-Charset：可接受的字符集，iso-8859-5, unicode-1-1
  - Accept-Encoding：可接受的编码类型，gzip, compress, deflate
  - Accept-Language: 可接受的语言，如: zh-cn, zh;q = 0.7, en-us, en; q = 0.3
  - Authorization: 认证信息
  - Expect：期望得到的action
  - From：告知服务器用用户代理的用户的电子邮件地址
  - Host：发送给的主机
  - If-Match：一串字符串用来和ETag匹配
  - If-Modified-Since：一个时间变量，判断这个时间之后不再变化
  - If-None-Match
  - If-Range：指定一个Range值，ETag或者时间，有匹配的就返回Range所规定的资源，否则返回全部资源
  - If-Unmodified-Since：自从某个时间起，没有修改过
  - Max-Forwards：最多的转发次数，次数到达时，返回响应结果
  - Proxy-Authorizatoin：代理认证
  - Range：部分资源
  - Referer：告知服务器，请求的资源所在的完整的URI
  - TE：告诉服务器编码优先级
  - User-Agent：用户的信息，如HTTP版本，浏览器，操作系统等数据，如`User-Agent: Mozilla/5.0 (Windows NT 6.1;WOW64;rv;13.0) Gecko/20100101 Firefox/13.0.1`
- 响应字段
  - Accept-Ranges：告知客户端能否处理范围请求
  - Age：告知客户端源服务器在什么时候建立了相应
  - ETag：服务器会为每一份资源分配一个对应的ETag值
  - Location：资源地址
  - Proxy-Authenticate：告知客户端服务器所需的认证
  - Retry-After：告知客户端应该再多久之后再次重新发送请求
  - Server：服务器信息
  - Vary：
  - WWW-Authenticate：告知客户端适用于访问URI所需要的认证方法
- 实体字段
  - Allow：通知客户端能够支持指定URI资源的全部方法
  - Content-Encoding：实体的编码方式
  - Content-Language：实体的语言
  - Content-Length：实体的长度
  - Content-Location：返回报文主体资源的URI
  - Content-MD5
  - Content-Range
  - Content-Type
  - Expires
  - Last-Modified
- Cookie首部字段
  - Set-Cookie：事先告知各种信息：`Set-Cookie:name = value;expires = sdfdsfsd ; path = / ; domain = localhost;Secure;HttpOnly`
  - Cookie


- 其他首部字段
  - X-Frame-Options
  - X-XSS-Protection
  - DNT
  - P3P

## 确保Web安全的HTTPS

HTTP的缺点

- 报文没有加密
- 没有身份认证
- 无法验证报文完整性

HTTPS

- 加密，SSL加密
- 认证，交换密钥的认证方式
- 确保数据完整性
- 占内存，速度比较慢



---

剩下的感觉不算特别重要，先不写了

