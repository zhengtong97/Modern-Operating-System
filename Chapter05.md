# Chapter05
# 第五章 输入/输出 习题
- - - -
## 知识点小记
- - - -
1. I/O设备可分为：块设备和字符设备
* 块设备：把信息存储在固定大小的块种妹妹个快有自己的地址。基本特征是每个块都能独立于其他块而读写。如硬盘、蓝光光盘和USB盘。
* 字符设备：以字符为单位发送或接受一个字符流，而不考虑任何块结构。字符设备是不可存值得，也没有任何寻道操作。如打印机、网络接口、鼠标？？？，以及大多属与磁盘不用的设备都可看做字符设备。
2. I/O设备一般有机械部件和电子部件两部分组成。
* 电子部件称作设备控制器或适配器。常以主板上的芯片的形式出现，或者以插入扩展槽中的印刷电路板的形式出现。
* 机械部件是设备本身。
3. 控制器的任务是把串行的位流转换为字节块，并进行必要的错误矫正工作。
4. CPU有两种方法与设备的控制寄存器和数据缓冲区进行通信。
* 第一种方法：I/O端口号
	* 每个控制寄存器被分配一个I_O端口号（8位或16位），所有I_O端口形成I/O端口空间，并且受到保护似的普通的用户程序不能对其进行访问，只有操作系统可以访问。
	* 使用特殊的I/O指令`IN REG, PORT`，CPU可以读取控制寄存器PORT的内容并将结果存入到CPU寄存器REG中。类似地，使用`OUT PORT, REG` ，CPU可以将REG的内容写入到控制寄存器中。
	* 这种方案中，内存地址空间和I/O地址空间是不同的。
* 第二种方法：内存映射I/O
	* 将所有的控制寄存器映射到内存空间中，每个控制寄存器被分配唯一的一个内存地址，并且不会有内存被分配这一地址。
	* 优点：
		* ①I/O设备驱动程序可以完全用C语言编写
		* ②不需要特殊的保护机制来阻止用户程序执行I/O操作
		* ③可以引用内存的每一条指令，也可以引用控制寄存器
	* 缺点：
	* ①硬件必须能够针对每个页面有选择性的禁用高速缓存
	* ②所有的内存模块和所有I/O设备都必须检查所有的内存引用，以便了解有谁做出相应。
5. 直接存储器存取DMA
* 第一步：
	* CPU通过设置DMA控制器的寄存器对它进行编程，使DMA控制器知道讲什么数据送到什么地方。
	* DMA控制器还要向磁盘控制器发出一个命令，通知它从磁盘读数据到其内部的缓冲区，并对校验和进行检验。
* 第二步：
	* DMA控制器通过在总线上发出一个读请求到磁盘控制器而发起DMA传送。
* 第三步：
	* 磁盘控制器从其内部缓冲区中赌侠一个字，写到内存。
* 第四步：
	* 写操作完成时，磁盘控制器在总线上发出一个应答信号到DMA控制器。
	* DMA控制器步增要使用的内存地址，并步减字节计数。
	* （如果字节计数仍大于0，则重复第二步到第四步）
6. 重温中断：
	* 当一个I/O设备完成交给它的工作时 ，它就产生一个中断，通过在分配给它的一条总线信号线上置起信号而产生中断的。
	* 该信号被主板上的中断控制器芯片检测到，由终端控制器芯片决定做什么。
	* 为了处理中断，终端控制器在地址线上放置一个数字表明哪个设备需要关注，并置起一个中断CPU的信号。
	* 地址线上的数字被用作指向一个称为中断向量的表格的索引，以便读取一个新的程序计数器。这个程序计数器指向相应的中断服务过程的开始。
	* 中断服务运行后，它立刻将一个确定的值写到中断控制器的某个I/O端口来对中断作出应答。
7. 精确中断的特性：
	* PC保留存在一个已知的地方。
	* PC所指向的指令之前的所有指令已经能完全执行。
	* PC所指向的指令之后的所有指令都没有执行。
	* PC所指向的指令的执行状态是已知的。
	* note：对于PC所指向的指令之后的那些指令，此处并没有禁止他们开始执行，而只是要求在中断发生之前必须撤销它们对寄存器或内存所作的任何修改。PCPC所指向的指令有可能已经执行了，也可能还没执行，但必须清楚是哪种情况。
8. I/O软件的目标：设备独立性、统一命名、错误处理、同步和异步、缓冲。
9. I_O可以用三种不同的方式实现：程序控制I_O，中断驱动I_O，使用DMA的I_O。
	* 程序控制I/O：让CPU做所有的工作。
		* 首先，数据被复制到内核空间。
		* 然后，操作系统进入一个密闭的循环，一次输出一个字符。
		* 输出一个字符之后，CPU要不断地查询设备以了解它是否就绪准备接受另一个字符。这一行为称为轮询或者忙等待。
```
copy_from_user(buffer,p,counnt);
for (i = 0;i < count; i++){
		while(*printer_status_reg != READY);
		*print_data_register = p[i];
}
return_to_user();
```
	* 中断驱动I/O：允许CPU在等待打印机变为就绪的同时做某些其他事情。
	* 使用DMA的I_O：有DMA控制器而不是主CPU做全部工作。需要特殊的硬件（DMA控制器），但是使CPU获得自由从而可以在I_O期间做其他工作。
10. I/O软件层次通常分成四个层次：
4				用户级I/O软件
3		与设备无关的操作系统软件
2				设备驱动程序
1				中断处理程序
0				  	硬件
11. 磁盘臂调度算法：
* 先来先服务（FCFS）：按照请求接收顺序完成请求。
* 最短寻道优先（SSF）：下一次总是处理与磁头距离最近的请求以使寻道时间最小化。
* 电梯算法
12. 时钟，由三个部件构成：晶体振荡器，计数器和存储寄存器。
- - - -
Q1:1.芯片技术的进展已经使得将整个控制器包括所有总线访问逻辑放在一个便宜的芯片上成为可能。这对于图1-6的模型具有什么影响？
![](Chapter05/E780AEC2-A208-406D-9CD4-D87D2906B007%202.png)

A：在此图中，一个控制器有两个设备。单个控制器可以有多个设备就无需每个设备都有一个控制器。如果控制器变得几乎是自由的，那么只需把控制器做入设备本身就行了。这种设计同样也可以并行多个传输，因而也获得较好的性能。
- - - -




