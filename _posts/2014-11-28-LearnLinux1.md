---
layout : default
title: Linux高性能服务器编程-读书笔记-CH2
---
#Linux高性能服务器编程    
 > Edit by Elaine @2014.11.28     
 
###第二章 IP协议详解     
* IP服务的特点   
    **无状态**（通信双方不同步传输数据的状态信息）可能是乱序、重复的数据       
        优点：简单高效
    **无连接**（通信双方不长久维持对方的任何信息）每次发送都必须指明IP    
    **不可靠**（不能保证IP数据报准确的到达接收端）上层需要自己实现数据确认，超时重传等      
* IP转发  
    * 检查数据报头部TTL值，若为0，丢弃数据报；
    * 查看路由选择新安乡，若设置了该选项，则检查数据报的目标IP地址是否为本机的某个IP地址，若不是，则发送ICMP源站选路失败报文给发送端；  
    * 若有必要，则发源端一个ICMP重定向报文，告诉它一个更合理的下一跳路由器；   
    * TTL值-1；  
    * 处理IP头部选项；   
    * 若有必要，执行IP分片操作；   
   
```
IPV6并不是IPV4的简单扩展，而是完全不同的协议， 除了解决IP不够用的问题，还增加了多播和流的功能，引入了自动配置功能，增加专门的网络安全功能等。   
```