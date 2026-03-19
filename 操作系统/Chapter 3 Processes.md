### Process 进程 - 正在执行的程序
内存中分为多个区域：
- 代码段
- 数据段：储存全局参数
- 栈(Stack)：储存局部参数
- 堆(Heap)

##### ps command : 查看正在运行的进程
```
PID    进程编号
TTY    类型
STAT   进程状态
START  开始运行的运行时间
```

##### top command, htop, atop command

##### pgreg [process name] 搜索一个进程名，返回pid

### Process State
- 一个进程有五种状态：
	- New：正在创建。
	- Running：正在执行。
	- Waiting：正在等待某些事件发生。
	- Ready：正在CPU中分时进行。
	- Terminated：进程执行完毕，终止。
- 这五个状态贯穿了整个进程的生命周期。

##### Process Control Block (PCB)
- 用于存储每个进程的信息。
- pid：十六位二进制数，表示进程的唯一编号。也就是说操作系统最多负载65535个进程数。
- 调度信息
- 父子进程关系信息（链表形式）
- 打开文件列表
- task_struct 结构体非常长，有几百行代码。
	- 进程运行有process（进程）和thread（线程）两种模式。进程与用户相关，它的代码量不可估计，往往很大。我们需要将它的代码切分运行，放在多个核/分时运行，成为线程。这种被称为LWP(light weight process)。
	- 进程往往和服务相关。面对服务器等场景的多服务情景，这个时候一般通过轻量线程来完成服务。

### Process Scheduling
 进程控制器从可用进程中选择下一个在CPU核上执行。而这个选择是从队列中选择：
- Ready queue：可以执行的进程。
- Wait queue：等待中的进程。
	- 进程在不同队列中迁移。

### Context Switch(上下文切换)
- 在切换进程时候，需要保存老进程信息，同时加载新进程信息。
- 这个过程被称为上下文切换。
- 上下文切换开销在百微秒级别，这个开销实际上很高。在设计切换时需要考虑到切换的开销。

### Process Creation
- 进程创建总是由父进程创建子进程。进程的继承和类的继承很相似，可以继承父进程的所有内存空间（默认）
	- 也可选择只继承父进程的部分资源、或是完全不共享资源。
- 父进程通常等待子进程结束后继续执行。
- 地址空间：子进程通常继承父进程的地址空间。
	- Unix examples：
		- `fork()`：创建子进程，会克隆所有的信息。
		- `exec()`：执行进程的任务
		- `exit()`：关闭子进程
		- `wait()`：父进程继续执行

##### C program forking separate process
- `pid = fork()`创建子进程

### Process Termination
- 进程执行结束后使用`exit()`结束。将状态数据从子进程返回给父进程 (通过`wait()`)
- 如果父进程被终止，那么子进程将会被同时终止。

### 生产者-消费者问题
- 对于协作进程来说，生产者进程生产数据存在内存中，而消费者进程负责消耗数据。
- 对于共享的内存空间，倘若两个进程在交错进行时同时访问内存，则可能会导致该内存空间错乱。


### Pipes 通信管线
- Ordinary Pipes 日常管道：普通传输通道，带宽有限。
	- 父进程创建一个子进程，原先的数据变为父子进程通过管道同步共享。
- Named Pipes 命名管道：带宽更大。
- 管道调度算法：
	- FIFO：先进先出。（等同于FCFS）
- 管道命令：
	- `mkfifo pipe1`使用**mkfifo**创建FIFO管道。
- 实现管道的代码：
	- c语言中用mknod创建管道。

### Sockets 套接字
- `socket`被定义为网络上的终端。在客户端与服务端分别创建一个socket进程，在获得两边的IP地址和端口号后，利用TCP的技术传输。
	- IP：internet protocol 互联网协议，其定义互联网上每一个地址。
	- TCP：Transmission Control Protocol ： 传输控制协议，这是一个传输层协议。它 的传输模式是点对点的传输。
		- 传输细节：
			- 三次握手同步。
			- 同步完成后开始数据传输。
			- 数据丢失会重新传输。
		- 相对稳定可靠。
		- 缺点：一对多、多对多情况下不高效。
	- UDP：User Datagram Protocol： 用户数据电报协议。牺牲可靠性，用广播、多播的方式对一对多的情境下进行传输。

### 5 IPCs in Linux：
- Pipe
- Message Queue
	- 类似发微信，量比较大
- Signal
	- 仅能发送标识，信息量更少、更精简。
- Semophare
	- 解决死锁
- Shared Memory

### MPI：Message Passing Interface
- 消息传递接口是一个标准化与可移植的函数库用于并行计算，特别是高性能计算环境。
- 在80年代末到90年代初，慢慢形成了 MPI 的框架。
- MPI 的影响非常广泛。
- 常见的MPI通信：
	- broadcast：广播。
	- scatter：将数据分块切分传输。
	- gather：聚合。
	- reduction：（最常见）将任务分散后通过gather聚合，再通过计算等获得最终结果。
- 程序的运行：
	- 导入：MPI include file
	- 声明运行，入口：Initialize MPI environment
	- 声明结束，出口：Terminate MPI environment

