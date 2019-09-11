# 前端硬核知识
这里列出了一些前端的知识，可以从底层去看看前端的那些东西

## 从系统角度去看页面渲染

### 1.页面渲染流程（HTML/CSS/JS）
HTML解析过程的输入由一串代码点(一个Unicode代码点并表示为一个四到六个十六进制数)组成，这些代码点通过标记化阶段，然后是树构造阶段，最终输出一个Document对象。

![parsing-model-overview](https://raw.githubusercontent.com/luobinbinchina/FE-Technology/master/image/parsing-model-overview.jpg)

#### 1.1 浏览器渲染一个页面是按照
1. 自上而下
2. 遇CSS/JS阻塞
3. 边解析边构建

#### 1.2 DOMTree解析
样式（link、style）与脚本文件（script）加载并解析完成后，DOMTree基本完成并触发DOMContentLoaded事件，此时可通过JS操作DOM节点。

#### 1.3 构建RenderTree(呈现树)
DOMTree解析后开始构建RenderTree(呈现树)，目的是将CSS样式注入到DOM节点上（webkit内核将该过程称作附着，其他浏览器概念不尽相同），RenderTree的每一个节点都有为之相对应的DOM节点的CSS框，框的的类型取决于DOM节点的display（block/inline等），但DOM节点不太一定有与之相对应的RenderTree节点且位置也不一定相同。

#### 1.4 计算布局
DOMTree、RenderTree构造完后，开始进行布局并计算RenderTree节点的大小和位置，简单来说布局的过程是浏览器根据窗口（包括显示器）实际大小来计算RenderTree节点实际的显示大小和位置。

#### 1.5 页面绘制
布局完成后就开始进行RenderTree节点绘制，最终完成页面的渲染流程

### 2.一个页面，四个进程
如图，当打开 `https://www/baidu.com` 网页，可以看到任务管理器中一共存在四个进程，浏览器、GPU进程、实用程序NetworkService、标签页，为什么有这些进程，负责哪些功能模块？

![1page4process](https://raw.githubusercontent.com/luobinbinchina/FE-Technology/master/image/1page4process.jpg?w=100)

#### 2.1 进程 —— 浏览器
浏览器主进程，负责浏览器页面的显示，各个页面的管理，负责其他进程的创建和销毁，保证UI进程（主线程）不会被任何其他操作（如本地文件读写，socket读写，数据库操作）阻碍。

#### 2.2 进程 —— GPU
GPU进程主要做页面的3D绘制，由Render进程提需求给浏览器进程通过IPC下达给GPU进程进行绘制，有且只有一个GPU进程，不论打开多少个页面

#### 2.3 进程 —— 实用程序NetworkService
实用程序NetworkService是网络进程，负责页面的网络资源加载，可以看到当有网络请求时，该进程就会
发起网络请求

#### 2.4 进程 —— 标签页
标签页是渲染进程，任务是讲HTML/CSS/JS转换为用户可以操作的页面，Blink和V8都在该进程运行，为了安全考虑，每个页面都会创建一个渲染进程且在沙箱模式下运行

#### 2.5 进程 —— 其他
其他进程会根据用户的设置，如果装了相关插件并启用的时候会触发插件进程等

### 3. HTML文档流的传送
当用户访问一个网页时，服务端会把处理好的HTML文档流传送浏览器，再由浏览器进行渲染。这里看一下服务器是如何把HTML文档完整的传递给浏览器(PS：这里就对网络架构OSI，TCP/IP，三次握手四次挥手及HTTP报文结构的解释)

#### 3.1 传输协议 —— TCP
传输控制协议（TCP，Transmission Control Protocol）是一种面向连接的、可靠的、基于字节流的传输层通信协议，由IETF的RFC 793定义，该协议是为了在不可靠的互联网络上提供可靠的端到端字节流而专门设计的一个传输协议。

当应用层向TCP层发送用于网间传输的、用8位字节表示的数据流，TCP则把数据流分割成适当长度的报文段，最大传输段大小（MSS）通常受该计算机连接的网络的数据链路层的最大传送单元（MTU）限制。之后TCP把数据包传给IP层，由它来通过网络将包传送给接收端实体的TCP层。

#### 3.1.1 网络架构（五、七层框架）
![osi](/image/osi.jpg)

`五层因特网协议栈`: 这里简单概括一下就是从应用层的发送http请求，到传输层通过三次握手建立tcp/ip连接，再到网络层的ip寻址，再到数据链路层的封装成帧，最后到物理层的利用物理介质传输的过程。

1. 应用层(dns,http) DNS解析成IP并发送http请求
2. 传输层(tcp,udp) 建立tcp连接（三次握手）
3. 网络层(IP,ARP) IP寻址 
4. 数据链路层(PPP) 封装成帧 
5. 物理层(利用物理介质传输比特流) 物理传输（然后传输的时候通过双绞线，电磁波等各种介质）

`完整的OSI七层框`:  物理层、 数据链路层、 网络层、 传输层、 会话层、 表示层、 应用层，相比五层协议多了表示层和会话层

- 表示层：主要处理两个通信系统中交换信息的表示方式，包括数据格式交换，数据加密与解密，数据压缩与终端类型转换等
- 会话层：它具体管理不同用户和进程之间的对话，如控制登陆和注销过程

#### 3.2 传送流程

`URL解析`: 浏览器会根据protocol、host、port、path、query、fragment(#后的hash值，用来做锚点)

`DNS解析`: 如果输入的是域名，需要进行dns解析成IP，浏览器会按照自身缓存、本机缓存、本地host、DNS域名服务器来查对应的IP地址

`三次握手`: “三次握手”即对每次发送的数据量是怎样跟踪进行协商使数据段的发送和接收同步，根据所接收到的数据量而确定的数据确认数及数据发送、接收完毕后何时撤消联系，并建立虚连接。

![threehands](/image/threehands.jpg)

`四次挥手`: 由于TCP连接是全双工的，因此每个方向都必须单独进行关闭。这原则是当一方完成它的数据发送任务后就能发送一个FIN来终止这个方向的连接。收到一个 FIN只意味着这一方向上没有数据流动，一个TCP连接在收到一个FIN后仍能发送数据。

![threehands](/image/fourhands.jpg)

### 3.3 HTTP报文结构

#### 3.3.1 通用头部
通用头部就是我们经常见到的几个信息：

- `Request Url`: 请求的web服务器地址
- `Request Method`: 请求方式（Get、POST、OPTIONS、PUT、HEAD、DELETE、CONNECT、TRACE）
- `Status Code`: 请求的返回状态码，如200代表成功
- `Remote Address`: 请求的远程服务器地址（会转为IP）

#### 3.3.2 Request Headers 请求头部
请求是我们做分析时用到的，常用的有：

- `Accept`: 接收类型，表示浏览器支持的MIME类型（对标服务端返回的Content-Type）
- `Accept-Encoding`：浏览器支持的压缩类型,如gzip等,超出类型不能接收
- `Content-Type`：客户端发送出去实体内容的类型
- `Cache-Control`: 指定请求和响应遵循的缓存机制，如no-cache
- `If-Modified-Since`：对应服务端的Last-Modified，用来匹配看文件是否变动，只能精确到1s之内，http1.0中
- `Expires`：缓存控制，在这个时间内不会请求，直接使用缓存，http1.0，而且是服务端时间
- `Max-age`：代表资源在本地缓存多少秒，有效时间内不会请求，而是使用缓存，http1.1中
- `If-None-Match`：对应服务端的ETag，用来匹配文件内容是否改变（非常精确），http1.1中
- `Cookie`：有cookie并且同域访问时会自动带上
- `Connection`：当浏览器与服务器通信时对于长连接如何进行处理,如keep-alive
- `Host`：请求的服务器URL
- `Origin`：最初的请求是从哪里发起的（只会精确到端口）,Origin比Referer更尊重隐私
- `Referer`：该页面的来源URL(适用于所有类型的请求，会精确到详细页面地址，csrf拦截常用到这个字段)
- `User-Agent`：用户客户端的一些必要信息，如UA头部等

