---
layout : default
title : Socket API 编程实践（1）
---
# Socket API 编程实践（1） 

### 这集副标题是 *菜鸟的丢人史*  
> Edit by Elaine @ 2014-12-9   

这几天看论文看的有些头痛，想着还是写写代码轻松一下，没想到还是遇到了各种问题。作为一个初学Linux网络编程的菜鸟，表示有些东西，理论背的再熟练，都不及动手写一个代码长记性。   

这段时间，《Linux高性能服务器编程》看到了第五章，深感唯有动手编码才能领会各种协议和Socket的精髓。抄了书上几个例子，竟然有因为抄错代码运行不出正确结果而纠结了一天的荒唐事发生，真是无颜号称自己是计算机专业的学生。究其原因，无非是自己只是把书上的理论看过了，却没能实际理解，抄了书上的代码，却没有逐句去琢磨每行代码的作用。   

今晚，想着从书上给的TCP的发送/接收数据的例子稍微扩展一下，写个UDP的玩玩，本想着完全没有难度，因为Socket API几乎已经完成了绝绝大多数工作，只需要把逻辑理清楚就行了。结果......啊唏......  
 
先说我理解的**TCP发送和接收数据**的过程：
  
* 发送端：  

	1.读取服务器（目的主机）IP，Port，转换为`sockaddr_in`。这里注意三个字段：`sin_family`, `sin_addr`, `sin_port`: TCP/IP协议族的`sin_family`指定为`AF_INET`； `sin_addr`才是真正存放转换后的IP地址的字段； `sin_port`存放转换后的端口号。（**转换指转为网络字节序**）   
	2.建立Socket。`socket`系统调用，第二个参数指定为`SOCK_STREAM`时为TCP协议，数据流。 返回值非负时成功建立socket。  
	3.请求连接。`connect`系统调用，返回值小于0时，连接失败。否则即可发送数据，**此时三次握手已经完成**。  
	4.发送数据。`send`系统调用，可发送`normal_data`（正常数据）和`oob_data`（TCP带外数据，紧急数据）。
	5.数据发完，关闭连接。  
 
* 接收端：  
	1.读取服务器程序IP，Port，转换为`sockadd_in`。注意字段同发送端。  
	2.建立socket，`socket`系统调用。   
	3.绑定地址。`bind`系统调用，绑定`sockfd`与服务器程序地址（`sockadd_in`类型的地址）。  
	4.监听端口。`listen`系统调用。监听服务器程序所在端口（根据`sockfd`）。
	5.接受连接。`accept`系统调用。**此时三次握手至少已完成前两次握手**。  
	6.接受数据。`recv`系统调用。第三个参数可选`MSG_OOB`表示带外紧急数据，0表示普通数据。   
	7.数据传送结束，关闭连接。    
	
今天写的UDP传送数据的过程与TCP过程有相似之处，但有巨大的差异，建立socket的参数不同，发送接收的系统调用函数不同，不再赘述。  
两个协议虽然逻辑都不复杂，差异也很明显：**TCP面向连接，数据可靠，基于字节流；而UDP无连接，不可靠，基于数据报**。但是写起代码来，居然忘了这些，只一味模仿TCP，简直汗颜。直到代码运行后，始终在listen之后提示错误，半天才意识到，UDP是无连接的，不用侦听端口，只需要绑定地址之后就开始传送数据了，自然也不存在接收端的接受连接，和发送端的请求连接。  
   
为了提醒自己UDP无连接的特性，还是将代码中错误的部分保留，仅用注释标记：   
		
		// SEND 
		#include <sys/types.h>
		#include <sys/socket.h>
		#include <libgen.h>
		#include <assert.h>
		#include <errno.h>
		#include <stdlib.h>
		#include <stdio.h>
		#include <netinet/in.h>
		#include <arpa/inet.h>
		#include <unistd.h>
		#include <string.h>

		#define BUFFER_SIZE 1024

		int main(int argc, char* argv[]) {
			if (argc <= 2) {
				printf ("usage: %s ip_address, port number.\n", basename(argv[0]));
				return 1;
			}
			
			int port = atoi(argv[2]);
			const char* ip = argv[1];

			struct sockaddr_in address;
			bzero(&address, sizeof(address));
			address.sin_family = AF_INET;
			inet_pton(AF_INET, ip, &address.sin_addr);
			address.sin_port = htons(port);

			int sockfd = socket(PF_INET, SOCK_DGRAM, 0);
			assert(sockfd >= 0);

			
			/* if (connect(sockfd, (struct sockaddr*)&address, sizeof(address)) < 0) {
			 	printf("connect failed\n");
			 	return -1;
			 } 
			else {
				const char* buffer = "hello Elaine";
				sendto(sockfd, buffer, sizeof(buffer), 0, (sockaddr*)&address, sizeof(address));
			}*/
			close(sockfd);
			return 0;

		}   
	
---

		// RECV  
		#include <sys/socket.h>
		#include <sys/types.h>
		#include <netinet/in.h>
		#include <arpa/inet.h>
		#include <assert.h>
		#include <stdio.h>
		#include <unistd.h>
		#include <stdlib.h>
		#include <errno.h>
		#include <string.h>
		#include <libgen.h>
		#include <iostream>
		using namespace std;

		#define BUFFER_SIZE 1024

		int main(int argc, char* argv[]) {
			if (argc <= 2) {
				printf("usage:%s ip_address, port number\n", basename(argv[0]));
				return 1;
			}
			const char* ip = argv[1];
			int port = atoi(argv[2]);

			struct sockaddr_in address;
			bzero(&address, sizeof(address));
			address.sin_family = AF_INET;
			inet_pton(AF_INET, ip , &address.sin_addr);
			address.sin_port = htons(port);			int sock = socket(AF_INET, SOCK_DGRAM, 0);
			assert(sock >= 0);

			int ret = bind(sock, (struct sockaddr*)&address, sizeof(address));
			assert(ret != -1);

			//ret = listen(sock, 5);

			struct sockaddr_in client;
			socklen_t client_addrlength = sizeof(client);
		
			/* int connfd = accept(sock, (struct sockaddr*)&client, &client_addrlength);
			 if (connfd < 0) {
			 	printf("errno is %d\n", errno);
			 }
			 else {
				char buffer[BUFFER_SIZE];
				memset(buffer,'\0',sizeof(buffer));
				ret = recvfrom(sock, buffer, BUFFER_SIZE-1, 0, (struct sockaddr*)&client, &client_addrlength);
				printf("got %d bytes of udp data '%s' \n", ret, buffer);
			}*/
			close(sock);
			return 0;
		}
		
---

##就当我在卖萌吧...