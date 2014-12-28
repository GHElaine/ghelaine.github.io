---
layout : default
title: Linux高性能服务器编程-读书笔记-CH7
---
#Linux高性能服务器编程    
 > Edit by Elaine @2014.12.28
 
###第七章 Linux服务器程序规范   

​* 日志  
    Linux系统提供守护进程syslogd或rsyslogd来处理系统日志。服务器调试和维护都需要日志系统。  
    分别使用syslog、printk函数将用户输出消息和内核消息输出到指定文件，syslogd或rsyslogd函数通过读取指定文件再输出到指定日志文件，来记录系统日志。 
     
   * syslog函数  
   用于和rsyslogd守护进程通信：
     
	
        #include <syslog.h>
        void syslog( int priority, const char* message, ...);
        /* 采用可变参数来结构化输出；*/  
  
    
   * 日志掩码    
  用于过滤系统调试时的日志。
  
 	
        ​#include <syslog.h>
        ​int setlogmask( int maskpri);  
        ​/* 日志级别大于maskpri的日志信息将被系统忽略*/   
       
   ​
   * 关闭日志功能 
   
   
        ​#include <syslog.h> 
        ​void closelog();
   
   * 用户信息
     
	
    	​​ #include <sys/types.h>    
    	​​ ​#include <unistd.h>
    	 ​uid_t getuid();         ​    ​    ​/* 获取真实用户ID */
    	 ​​uid_t geteuid();    ​    ​        ​/* 获取有效用户ID */​
    	 ​​gid_t getgid();    ​    ​    ​    ​/* 获取真实组ID */
    	 ​​​git_t getegit();    ​    ​    ​    ​/* 获取有效组ID */
    	 ​​​​int setuid(uid_t uid);    ​    ​   /* 设置真实用户ID */
    	 ​​​​​int seteuid(uid_t uid);    ​    ​  /* 设置有效用户ID */
    	 ​​​​​​int setgid(git_t gid);    ​    ​     /* 设置真实组ID */
    	 ​​​​​​int setegid(git_t gid);    ​    ​    /* 设置有效组ID */

**一个进程拥有两个用户ID： UID和EUID。EUID为了方便资源访问，它可以使得用户拥有该程序的有效用户权限。**
