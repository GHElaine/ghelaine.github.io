---
layout : default
title: 深入理解计算机系统——读书笔记
---
#深入理解计算机系统--读书笔记
>  Edit by Elaine 2014-09-14

##CH1   
    
1.2   hello.c程序的执行过程：  
           Hello.c -> 预处理-> hello.i -> 编译器 -> hello.s -> 汇编器 -> hello.o -> 链接器 -> hello (可执行程序)
        链接是在机器指令生成之前，汇编阶段结束之后开始的。   
        
1.7  ***操作系统的两个功能：***  
 
  * 1）防止硬件被失控的应用程序滥用；    
  * 2）在控制复杂而又通常广泛不同的低级硬件设备方面，为应用程序提供简单一致的方法。  
  
1.7.1 ***进程:***操作系统对运行程序的一种抽象。    
 
  * 1. 并发：一个系统上同时可以运行多个程序；    
  * 2. 上下文切换：实现不同进程指令交替执行；  
  * 3. 上下文：操作系统保存的进程运行时所需的状态信息；  
 
1.7.2 ***线程：*** 一个进程可以由多个线程组成。每个线程都运行在进程的上下文中，并共享同样的代码和全局数据。多线程比多进程更容易共享数据，也比进程更高效。  

1.7.3 ***虚拟存储器：*** 每个进程看到的存储器都是一致的，称为虚拟地址空间。  
从下到上，依次：**程序代码和数据区**（可执行文件直接初始化）；**堆**（Malloc，free等操作运行时加载）；**共享区**（C标准库以及数学库加载需要）；**栈**（运行时加载，动态增长和缩减）；**内核虚拟存储器**（地址空间顶部四分之一留给内核常驻，程序不允许读取这部分内容）



##CH2   
2.1.2 字长决定虚拟地址空间的大小。  

##CH3   
3.1 ***摩尔定律***：每过18个月，芯片上集成的晶体管的数量翻倍。  
3.6 switch语句是通过跳转表来索引跳转程序段的，这种结构使switch case语句的执行效率远高于if else，并且与case 的数量无关。编译器会在case较多（大于等于4个），且跨度较小时，使用**跳转表**。  
    *** 栈 ***：向着地址变小的方向增长，从顶向底压栈。

##CH4  
程序指令执行的过程：取指，解析，执行，访存，回写  

##CH5  
5．1编译器优化的限制：  
 
* 1）不能改变正确的程序行为；    
* 2）对程序行为和使用环境的了解有限；      
* 3）需要快速完成编译工作。另外：不可知的函数调用。  

优化的方面：   

 * 1） 消除循环的低效率  
 * 2） 减少过程调用    
 * 3） 消除不必要的存储器引用   
  
 >处理器性能受三类约束限制：  
 
  * 1）程序中数据相关性迫使一些操作延迟直到他们的操作数被计算出来。因为功能单元有一个或多个周期的执行时间，这久设置了一个给定的操作序列执行周期数的下限。  
  * 2）资源约束限制了在任意给定时刻能够执行多少个操作。  
  * 3）分支预测逻辑限制处理器能够在指令流中超前工作以保持执行单元繁忙的程度。预测错误时，处理器从正确位置重新开始都会引起很大的延迟。

#CH7 链接  
> 链接：将不同部分的代码和数据收集和组合成为一个单一文件的过程这根文件可被加载（或拷贝）到存储器并执行。链接可以执行于编译时，即源代码被翻译成机器代码时，也可以执行于加载时，就是程序被加载器加载到存储器并执行时，甚至执行于运行时，由应用程序来执行。    
 
##静态链接   
> 静态链接器：以一组可重定位目标文件和命令行参数作为输入，生成一个完全链接的可以加载和运行的可执行目标文件作为输入。输入的可重定位目标文件由各种不同的代码和数据节组成。指令在一个节中，初始化的全局变量在另一个节中，而未初始化的变量又在另一个节中。
   
* 符号解析：目标文件定义和引用符号。目的是将每个符号引用和一个符号定义联系起来。    
> 连接器解析符号引用的方法是将每个引用与它输入的可重定位目标文件的符号表中的一个确定的符号定义联系起来。对那些和引用定义在相同模块中的本地符号引用，符号解析是非常简单明了的。编译器只允许每个模块中的每个本地符号只有一个定义。编译器确保静态本地变量，他们也会有本地链接符号，拥有唯一的名字。  

    解析全局符号。的那个编译器遇到一个不是在当前模块中定义的符号时，会假设该符号在其他某个模块中定义的，生成一个连接器符号表表木并把它交给链接器处理。如果链接器在它的任何输入模块都找不到这个被引用的符号，就输出一条错误信息并终止。相同符号会被多个目标文件定义，这种情况，链接器要么标志一个错误，要么以某种方法选出一个定义并抛弃其他定义。  
   > 如何解析多处定义的全局符号：函数和已初始化的全局变量是强符号，未初始化的全局变量是若符号。处理多处定义的规则：  
    >1. 不允许有多个强符号；   
    >2. 如果一个强符号和多个若符号，则选择强符号；   
    >3. 如果有多个若符号，则从若符号中任意选择一个。  
   ***规则2和规则3引起注意：不同的程序模块中定义相同符号时，一强一弱的情况下，链接器会静默选择强符号，尤其在重复的符号定义为不同类型时，会引起不易察觉的严重错误。例如一个double型符号会可能会覆盖两个int型符号。***  
   ***另一个经验是，注意查看warning***

* 重定位：编译器和汇编器生成从地址零开始的代码和数据节。连接器通过把每个符号定义与一个存储器位置联系起来，然后修改所有对这些符号的引用，使得它们指向这个存储器位置，而重定位这些节。    
> **与静态库链接**   静态库：将所有目标模块打包成一个单独的文件，可以作为链接器的输入。当连接器构造一个输出的可执行文件时，只拷贝静态库里被应用程序引用的目标模块。    

7.3 目标文件    

* 可重定位目标文件：包含二进制代码和数据，可以在编译时与其他可重定位目标文件合并起来，创建一个可执行目标文件。  
* 可执行目标文件：包含二进制代码和数据，其形式可以被直接拷贝到存储器并执行。  
* 共享目标文件：特殊类型的可重定位目标文件，在加载或运行时被动态加载到存储器并链接。    

7.9 加载可执行目标文件 
 

|	内存位置 | 备注 |
|:----------	 |:------------|  
| 内核虚拟存储器|用户代码不可见|  
| 用户栈       |运行时创建    |  
| 共享库的存储器映射区域 |			|  
| 运行时堆| 由malloc创建|   
| 读写段（.data, .bss） | 从可执行文件中加载|  
| 只读段（.init, .text, .rodata）| 从可执行文件中加载|  
| 未使用||    

7.10 动态链接共享库  

共享库：在运行时，可以加载到任意存储器地址，并在存储器中和一个程序链接起来。即为由动态链接器程序执行的动态链接。  


#CH8 异常控制流  

###8.1 异常  

| 类别 | 原因 | 异步/同步 | 返回行为 |  
|:-----|:-----|:--------|:--------|   
| 中断 | 来自I/O设备的信号 | 异步 | 总是返回到下一条指令 |   
| 陷阱 | 有意的异常  | 同步 | 总是返回到下一条指令 |   
| 故障 | 潜在可恢复的错误 | 同步 | 可能返回到下一条指令 |   
| 终止 | 不可恢复的错误 | 同步 | 不会返回 |   
  
###8.4 进程控制   

父进程调用fork函数创建一个新的运行子进程。  

*	fork函数一次调用，两次返回。一次返回到父进程，一次返回到新建的子进程。
* 	父子进程并发执行，独立运行。内核以任意方式交替执行。  
* 	父子进程具有相同但独立的地址空间。相同的用户栈、本地变量值、堆、全局变量值，以及相同的代码，但都有独自的私有地址空间，任何对变量的修改都是独立的，不会反映在另一个进程的存储器中。  
*  父子进程共享文件。子进程继承了父进程所有的文件。   

回收子进程  
当一个进程由于某种原因终止时，进程保持在终止状态，直到被父进程回收。一个终止但未被回收的进程称为僵死进程。    
父进程未回收子进程就终止的情况，由init进程来回收。


















