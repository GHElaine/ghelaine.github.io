---
layout : default
title  : Linux高性能服务器编程-读书笔记-CH4
---
#Linux高性能服务器编程    
 > Edit by Elaine @2014.12.2
 
###第4章 TCP/IP通信案例：访问Internet上的web服务器
       
####一些概念：  
* 正向代理服务器：要求客户端自己设置代理服务器的地址。每次请求都将直接发送到该代理服务器，并由代理服务器来请求目标资源。    
* 反向代理服务器：设置在服务器端，客户端无须进行任何设置。用代理服务器来接收Internet上的连接请求，将请求转发给内部网络上的服务器，并将从内部服务器上得到的结果返回给客户端。   
 
	>正向代理服务器与客户端在同一个逻辑网络中，反向代理服务器与服务器在同一个逻辑网络中。    

* 透明代理服务器：只能设置在网关上。可看做正向代理的一种特殊情况。  

	>代理服务器通常还提供缓存目标资源的功能。    

####HTTP服务 通信过程   
* 访问DNS服务器     
* 本地名称查询    
* HTTP通信    

  HTTP请求：  
    1. 短连接：请求处理完后立即关闭连接。同一个客户的多个HTTP请求不能共用一个TCP连接。  
    2. 长连接：多个请求可以使用同一个TCP连接。极大减少了网络上由于建立TCP连接导致的负荷，减少处理时间。

  HTTP应答：    
  
  >100 continue  通知客户端继续发送数据；      
  >200 OK 请求成功；   
  >3xx 重定向；  
  >4xx 客户端错误；   
  >5xx 服务器错误；

  Cookie:  
    HTTP的无状态特性（无上下文），使得交互程序不方便承上启下。cookie是服务器发送给客户端的特殊信息，客户端每次发送给服务器的请求都需要带上这些信息，服务器就可以区分不同客户，基于浏览器的自动登录就是用cookie实现的。   