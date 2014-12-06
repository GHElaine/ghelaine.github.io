---
layout : default
title: Linux高性能服务器编程-读书笔记-CH5
---
#Linux高性能服务器编程    
 > Edit by Elaine @2014.12.6
 
###第五章 Linux网络编程基础API    
* socket地址API    
    >* **字节序**：大端：高位存在低地址；小端：低位存在低地址  ；
    >* 大多数PC采用小端字节序，因此小端字节序又称为主机字节序；   
    >* 网络发送端总是把数据转换为大端字节序，接收端机器根据自己的字节序决定是否转换。大端字节序又称为网络字节序。   
    >* JAVA虚拟机采用大端字节序。   


* 创建Socket      
    使用Linux系统调用即可，注意参数。    
    
        #include <sys/types.h>      
        int socket( int domain, int type, int protocol) ;     
        /* domain: 协议族 ；  
         * type: 服务类型；  
         * protocol: 具体的协议;   */

* 命名 Socket    
    将一个socket与socket地址绑定称为给socket命名。
    服务器程序中，只有命名后，客户端才知道如何连接它。    
    客户端不需要命名socket而是匿名，操作系统会自动分配socket地址。  

        #include <sys/types.h>   
        #include <sys/socket.h>   
        int bind( int sockfd, const struct sockaddr* my_addr, socklen_t addrlen );     
        /* 将my_addr所指的socket地址分配给未命名的sockfd文件描述符，addrlen参数指出该socket地址的长度。     
        * 失败时返回值：  EACCES: 被绑定的地址是受保护地址，进超级用户能够访问。EADDRINUSE：被绑定的地址正在使用中。*/        

* 监听 Socket 
   
        #include <sys/socket.h>  
        int listen ( int sockfd, int backlog ) ;    
        /* sockfd: 被监听的socket    
         * backlog: 提示内核监听队列的最大长度  */     


* 接受连接  

        #include  <sys/types.h>   
        #include <sys/socket.h>   
        int accept ( int sockfd, struct sockaddr *addr, socklen_t *addrlen)   
        /* sockfd：执行通过listen系统调用的监听socket;   
         * addr: 用来获取被接受连接的远端socket地址；   
         * 成功时返回一个新的连接socket，唯一标识了被接收的这个连接，服务器可通过读写该socket来与被接受连接对应的客户端通信。*/       

* 发起连接  
 
        #include <sys/types.h>  
        #include <sys/socket.h>    
        int connect (int sockfd, const struct sockaddr *serv_addr, socklen_t addrlen);    
        /* sockfd: 由socket系统调用返回一个socket；   
         * serv_addr:服务器监听的socket地址；   
         * addrlen: 指定地址的长度；  */

* 关闭连接  
 
         #include <unistd.h>   
         int close( int fd);   
         /* 每次调用，将fd引用次数减一；
          * 直到为0时，真正关闭连接 */     
           
 ---    
 
         #include <sys/socket.h>   
         int shutdown( int sockfd, int howto );    
         /* 立即关闭连接。howto决定shutdown行为 */       

* 数据读写   
    TCP数据读写   
  
        #include <sys/types.h> 
        #include <sys/socket.h>  
        ssize_t recv( int sockfd, void *buf, size_t len, int flags ); 
        ssize_t send( int sockfd, const void *buf, size_t len, int flag );     

    通用数据读写  
     
        #include <sys/socket.h>   
        ssize_t recvmsg(int sockfd, struct msghdr* msg,  int flags) 
        ssize_t sendmsg(int sockfd, struct msghdr* msg, int flags)     

---
**Socket API太多了，不再赘述。其中很多参数必须要靠真正的编程实践才能体会其作用，等我的老联想本本的ubuntu装起了，好好写点代码实验下。有种老家伙迎来第二春的快感，毕竟已经废弃它了几个月了。**
