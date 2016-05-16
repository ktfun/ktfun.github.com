---
layout: post
title:  "CGI、FastCGI和WSGI"
date:   2016-05-12 09:39:00 +0800
categories: ktfun update
---

## 转发
* 转自http://www.biaodianfu.com/cgi-fastcgi-wsgi.html

## 注意
* 原文被删减过，格式也被重新整理,图片都没有保存。
* 本人并未详细考证原文，可能存在理解上的偏差，所谓尽信书不如无书。

## 内容

### CGI
CGI即通用网关接口(Common Gateway Interface)，是外部应用程序（CGI程序）与Web服务器之间的接口标准，是在CGI程序和Web服务器之间传递信息的规程。CGI规范允许Web服务器执行外部程序，并将它们的输出发送给Web浏览器，CGI将Web的一组简单的静态超媒体文档变成一个完整的新的交互式媒体。通俗的讲CGI就像是一座桥，把网页和WEB服务器中的执行程序连接起来，它把HTML接收的指令传递给服务器的执行程序，再把服务器执行程序的结果返还给HTML页。CGI 的跨平台性能极佳，几乎可以在任何操作系统上实现。  
CGI方式在遇到连接请求（用户请求）先要创建cgi的子进程，激活一个CGI进程，然后处理请求，处理完后结束这个子进程。这就是fork-and-execute模式。所以用cgi方式的服务器有多少连接请求就会有多少cgi子进程，子进程反复加载是cgi性能低下的主要原因。当用户请求数量非常多时，会大量挤占系统的资源如内存，CPU时间等，造成效能低下。  

CGI脚本工作流程：  

1. 浏览器通过HTML表单或超链接请求指向一个CGI应用程序的URL。
1. 服务器收发到请求。
1. 服务器执行所指定的CGI应用程序。
1. CGI应用程序执行所需要的操作，通常是基于浏览者输入的内容。
1. CGI应用程序把结果格式化为网络服务器和浏览器能够理解的文档（通常是HTML网页）。
1. 网络服务器把结果返回到浏览器中。
  

### FastCGI
FastCGI是一个可伸缩地、高速地在HTTP server和动态脚本语言间通信的接口。多数流行的HTTP server都支持FastCGI，包括Apache、Nginx和lighttpd等，同时，FastCGI也被许多脚本语言所支持，其中就有PHP。  
FastCGI是从CGI发展改进而来的。传统CGI接口方式的主要缺点是性能很差，因为每次HTTP服务器遇到动态程序时都需要重新启动脚本解析器来执行解析，然后结果被返回给HTTP服务器。这在处理高并发访问时，几乎是不可用的。FastCGI像是一个常驻(long-live)型的CGI，它可以一直执行着，只要激活后，不会每次都要花费时间去fork一次(这是CGI最为人诟病的fork-and-execute 模式)。CGI 就是所谓的短生存期应用程序，FastCGI 就是所谓的长生存期应用程序。由于 FastCGI 程序并不需要不断的产生新进程，可以大大降低服务器的压力并且产生较高的应用效率。它的速度效率最少要比CGI 技术提高 5 倍以上。它还支持分布式的运算, 即 FastCGI 程序可以在网站服务器以外的主机上执行并且接受来自其它网站服务器来的请求。  
FastCGI是语言无关的、可伸缩架构的CGI开放扩展，其主要行为是将CGI解释器进程保持在内存中并因此获得较高的性能。众所周知，CGI解释器的反复加载是CGI性能低下的主要原因，如果CGI解释器保持在内存中并接受FastCGI进程管理器调度，则可以提供良好的性能、伸缩性、Fail-Over特性等等。FastCGI接口方式采用C/S结构，可以将HTTP服务器和脚本解析服务器分开，同时在脚本解析服务器上启动一个或者多个脚本解析守护进程。当HTTP服务器每次遇到动态程序时，可以将其直接交付给FastCGI进程来执行，然后将得到的结果返回给浏览器。这种方式可以让HTTP服务器专一地处理静态请求或者将动态脚本服务器的结果返回给客户端，这在很大程度上提高了整个应用系统的性能。
  
FastCGI的工作流程:

1. Web Server启动时载入FastCGI进程管理器（PHP-CGI或者PHP-FPM或者spawn-cgi)
1. FastCGI进程管理器自身初始化，启动多个CGI解释器进程(可见多个php-cgi)并等待来自Web Server的连接。
1. 当客户端请求到达Web Server时，FastCGI进程管理器选择并连接到一个CGI解释器。Web server将CGI环境变量和标准输入发送到FastCGI子进程php-cgi。
1. FastCGI子进程完成处理后将标准输出和错误信息从同一连接返回Web Server。当FastCGI子进程关闭连接时，请求便告处理完成。FastCGI子进程接着等待并处理来自FastCGI进程管理器(运行在Web Server中)的下一个连接。 在CGI模式中，php-cgi在此便退出。
FastCGI 的特点
1. 打破传统页面处理技术。传统的页面处理技术，程序必须与 Web 服务器或 Application 服务器处于同一台服务器中。这种历史已经早N年被FastCGI技术所打破，FastCGI技术的应用程序可以被安装在服务器群中的任何一台服务器，而通过 TCP/IP 协议与 Web 服务器通讯，这样做既适合开发大型分布式 Web 群，也适合高效数据库控制。
1. 明确的请求模式。CGI 技术没有一个明确的角色，在 FastCGI 程序中，程序被赋予明确的角色（响应器角色、认证器角色、过滤器角色）。

### PHP-FPM
PHP-FPM是一个PHP FastCGI管理器，是只用于PHP的,可以在 http://php-fpm.org/download下载得到。PHP-FPM其实是PHP源代码的一个补丁，旨在将FastCGI进程管理整合进PHP包中。必须将它patch到你的PHP源代码中，在编译安装PHP后才可以使用。FPM（FastCGI 进程管理器）用于替换 PHP-CGI 的大部分附加功能，对于高负载网站是非常有用的。

它的功能包括：

1. 支持平滑停止/启动的高级进程管理功能；
1. 可以工作于不同的 uid/gid/chroot 环境下，并监听不同的端口和使用不同的 php.ini 配置文件（可取代 safe_mode 的设置）；
1. stdout 和 stderr 日志记录;
1. 在发生意外情况的时候能够重新启动并缓存被破坏的 opcode;
1. 文件上传优化支持;
1. “慢日志” – 记录脚本（不仅记录文件名，还记录 PHP backtrace 信息，可以使用 ptrace或者类似工具读取和分析远程进程的运行数据）运行所导致的异常缓慢;
1. fastcgi_finish_request() – 特殊功能：用于在请求完成和刷新数据后，继续在后台执行耗时的工作（录入视频转换、统计处理等）；
1. 动态／静态子进程产生；
1. 基本 SAPI 运行状态信息（类似Apache的 mod_status）；
1. 基于 php.ini 的配置文件。

### WSGI
Web服务器网关接口（Python Web Server Gateway Interface，缩写为WSGI）是为Python语言定义的Web服务器和Web应用程序或框架之间的一种简单而通用的接口。自从WSGI被开发出来以后，许多其它语言中也出现了类似接口。WSGI是作为Web服务器与Web应用程序或应用框架之间的一种低级别的接口，以提升可移植Web应用开发的共同点。WSGI是基于现存的CGI标准而设计的。  
WSGI区分为两个部份：一为“服务器”或“网关”，另一为“应用程序”或“应用框架”。在处理一个WSGI请求时，服务器会为应用程序提供环境资讯及一个回呼函数（Callback Function）。当应用程序完成处理请求后，透过前述的回呼函数，将结果回传给服务器。所谓的 WSGI 中间件同时实现了API的两方，因此可以在WSGI服务和WSGI应用之间起调解作用：从WSGI服务器的角度来说，中间件扮演应用程序，而从应用程序的角度来说，中间件扮演服务器。  

“中间件”组件可以执行以下功能：  

1. 重写环境变量后，根据目标URL，将请求消息路由到不同的应用对象。
1. 允许在一个进程中同时运行多个应用程序或应用框架。
1. 负载均衡和远程处理，通过在网络上转发请求和响应消息。
1. 进行内容后处理，例如应用XSLT样式表。

以前，如何选择合适的Web应用程序框架成为困扰Python初学者的一个问题，这是因为，一般而言，Web应用框架的选择将限制可用的Web服务器的选择，反之亦然。那时的Python应用程序通常是为CGI，FastCGI，mod_python中的一个而设计，甚至是为特定Web服务器的自定义的API接口而设计的。WSGI没有官方的实现, 因为WSGI更像一个协议。只要遵照这些协议,WSGI应用(Application)都可以在任何服务器(Server)上运行, 反之亦然。WSGI就是Python的CGI包装，相对于Fastcgi是PHP的CGI包装。  
WSGI将 web 组件分为三类： web服务器，web中间件,web应用程序， wsgi基本处理模式为 ： WSGI Server -> (WSGI Middleware)* -> WSGI Application 。  

#### WSGI Server/gateway
wsgi server可以理解为一个符合wsgi规范的web server，接收request请求，封装一系列环境变量，按照wsgi规范调用注册的wsgi app，最后将response返回给客户端。文字很难解释清楚wsgi server到底是什么东西，以及做些什么事情，最直观的方式还是看wsgi server的实现代码。以python自带的wsgiref为例，wsgiref是按照wsgi规范实现的一个简单wsgi server。它的代码也不复杂。
1. 服务器创建socket，监听端口，等待客户端连接。
1. 当有请求来时，服务器解析客户端信息放到环境变量environ中，并调用绑定的handler来处理请求。
1. handler解析这个http请求，将请求信息例如method，path等放到environ中。
1. wsgi handler再将一些服务器端信息也放到environ中，最后服务器信息，客户端信息，本次请求信息全部都保存到了环境变量environ中。
1. wsgi handler 调用注册的wsgi app，并将environ和回调函数传给wsgi app
1. wsgi app 将reponse header/status/body 回传给wsgi handler
1. 最终handler还是通过socket将response信息塞回给客户端。

#### WSGI Application
wsgi application就是一个普通的callable对象，当有请求到来时，wsgi server会调用这个wsgi app。这个对象接收两个参数，通常为environ,start_response。environ就像前面介绍的，可以理解为环境变量，跟一次请求相关的所有信息都保存在了这个环境变量中，包括服务器信息，客户端信息，请求信息。start_response是一个callback函数，wsgi application通过调用start_response，将response headers/status 返回给wsgi server。此外这个wsgi app会return 一个iterator对象 ，这个iterator就是response body。这么空讲感觉很虚，对着下面这个简单的例子看就明白很多了。

#### WSGI MiddleWare
有些功能可能介于服务器程序和应用程序之间，例如，服务器拿到了客户端请求的URL, 不同的URL需要交由不同的函数处理，这个功能叫做 URL Routing，这个功能就可以放在二者中间实现，这个中间层就是 middleware。middleware对服务器程序和应用是透明的，也就是说，服务器程序以为它就是应用程序，而应用程序以为它就是服务器。这就告诉我们，middleware需要把自己伪装成一个服务器，接受应用程序，调用它，同时middleware还需要把自己伪装成一个应用程序，传给服务器程序。
其实无论是服务器程序，middleware 还是应用程序，都在服务端，为客户端提供服务，之所以把他们抽象成不同层，就是为了控制复杂度，使得每一次都不太复杂，各司其职。

参考资料：

* http://blog.csdn.net/on_1y/article/details/18803563
* http://blog.kenshinx.me/blog/wsgi-research/
* http://blog.ez2learn.com/2010/01/27/introduction-to-wsgi/