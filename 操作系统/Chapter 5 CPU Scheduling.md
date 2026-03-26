- 最根本的原因：CPU计算资源与IO设备延时的不对等
	- I/O设备处理、运行需要很长时间
	- 如果一个程序串行执行，CPU与IO分散执行，且IO执行非常缓慢，会导致CPU的大量资源被浪费。
		- 大量爆发性使用
		- 少量持续性使用

### CPU Scheduler CPU调度器
	背景：CPU调度分为抢占式调度与非抢占式调度。
		抢占式调度：优先级高的任务会中断当前一切任务执行，使用所有资源
		非抢占式调度：等待当前任务执行完后，再插队分配。
	操作系统中有三种调度器：
		长时间调度器（一般用于I/O调度、慢速设备调度）
		中期调度器（适用于中速设备，比如快速SSD等百微秒、微秒级设备）
		短期调度器（常用于CPU）
		在OS中，三种调度器同时存在。
	对应调度器会有三个主要队列：
		Job Queue/Waiting Queue: 等待队列
		Ready Queue: 准备队列
		I/O Queue: IO队列
		等待队列等待完成后进入准备队列，进行CPU调度。CPU执行过程中如果牵扯到IO，则进入IO队列等待IO响应。响应结束后再由IO队列重新进入准备队列。

##### Dispatcher
- 调度器将CPU的控制权交给
- 调度延迟：调度程序停止一个进程并开始另一个进程运行所需的时间：10微秒左右

##### Scheduling Criteria 调度评估准则（三个时间==易考==）
- CPU utilization          CPU利用率
- Throughput               吞吐：处理器能同一时间处理的任务数量
- Turnaround time       ==周转时间==(服务时间)：完成进程的时间
- Waiting time             ==等待时间==：在准备队列中等待的时间
- Response time          ==响应时间==：进程在第一次提交到执行之间的时间
- Excution time            ==执行时间==：假设执行时间恒定。

| Response | run | wait | run | wait | run |
| -------- | --- | ---- | --- | ---- | --- |
`CPU利用率和吞吐受硬件影响太大，不好复现/不具有泛化能力。所以着重考虑后面四个时间更优秀的算法。`
##### FCFS（first come, first served) 先进先出算法
	linux里面叫FIFO(first in first out)
- Convoy effect：短任务被排到长任务后面，导致响应时间、等待时间、周转时间都爆了

##### SJF(Shortest-Job-First) Scheduling 最短任务优先算法
- 核心：优先执行短任务，后执行长任务。
- 问题：不同任务到达cpu准备队列的时间不同，那么如何平衡？
	- 抢占式：打断当前任务执行。
	- 非抢占式：等待当前任务执行结束后，再执行最短的任务。
		- 不做特别说明，==SJF是非抢占式的算法==
- 短任务优先算法在等待时间上是==最优的！==
	- 最少的平均等待时间（已严格证明）
- 抢占式算法：对已执行部分的算法，需要按照剩余时间来计算

##### SRTF(Shortest-Remaining-Time-First) 最短剩余时间优先算法
	也叫抢占式最短任务优先算法(preemptive SJF)
- 最大的问题：对剩余的时间只能估算。（并且剩的越少估算越准）

##### Round Robin(RR) 轮询算法
- 假设每一个进程的优先级都是一样的，那么就将时间分成小的片，每个进程循环执行。（通常一片在10-100毫秒）
- time quantum(q): 时间片
	- q large = FIFO
	- q small：频繁的切换产生巨大的开销。
	- 最好保证80%的短任务执行1个时间片就能完成；仅仅影响剩下来的长任务。
- 最高响应率优先：

##### Priority Scheduling 优先级调度
- 每个进程都和一个整数优先级相关
	- Linux：小的优先级高
	- Windows：大的优先级高
- 问题：”饥饿(starvation)“：低优先级任务可能会一直被打断导致永远被排挤，无法得到机会执行。
- 解决方法：”衰老(Aging)“：随着等待时间变长，优先级会增长。

##### Priority Scheduling w/ Round-Robin 优先级调度/轮询调度
##### Multilevel Queue 多级队列（优先级+队列）
- 将调度队列划分成多个优先级
- 调度策略：优先完成高优先级队列
- 每个队列内部轮询执行

##### Multilevel Feedback Queue
>[!tip]- 实际的类似操作系统的调度模式
>![[lec5-OS-new.pdf#page=41]]


##### Thread Scheduling
- 因为存在内核级线程和用户级线程，所以操作系统会用两套算法进行管理。

##### Multiple-Processor Scheduling 多处理器调度
- 对称式的多处理器调度相对比较简单。
- 四个多处理器调度的技术

##### 2.Multhreaded Multicore System 多线程/超线程
- 一个线程的运行阶段一定分为cpu阶段和memory阶段
- 将一个核共用两个线程，一个进行cpu阶段时另一个进行memory阶段
- 两倍的线程数
	- ”超线程“，因特尔(Hyperthreading)
	- 对操作系统来说，等效扩大了一倍的core数量

##### 3.Load Balancing 负载平衡
- 木桶效应：不均衡的负载会影响调度成效，而且还可能影响软硬件寿命。
- Push migration：定期向cpu推送任务
- Pull migration：空闲cpu向繁忙cpu拉取任务

##### NUMA
- 如果一个要进行一个慢速的内存访问操作，就需要将这个时间考虑进统筹。

##### Real-Time CPU Scheduling 实时CPU调度
- 有很多影响因素需要考虑
	- 中断延时
	- 导入延时
	- deadline

##### \*Priority based scheduling 基于优先级的实时调度
- 周期性任务：每隔一段时间，有一系列任务
- 每一个任务都拥有一个`deadline`
- RMS（rate montonic scheduling) 固定速率调度算法
	- 静态优先。
		- 周期短任务高优先级
		- 周期长任务低优先级

##### \*Earliest Deadline First Scheduling(EDF)
- 将截止时间最早的任务优先

##### Linux Scheduling in Version 2.6.23
- CFS Completely Fair Scheduler 公平调度算法
	- 根据进程等待时间确定`nice value (-20 - 19)`
- `weight = 1024 * 1.25^(-nice)`
- `virtual run time`虚拟运行时间
	- `virtual run time += 实际运行时间 * 1024 / weight`
- 最后依靠virtual run time构建红黑树
	- 系统永远调度`virtual run time`最小的进程/线程
	- 高权重的任务，virtual run time会增长得更快，执行一段时间后就不是最优先任务了
	- 保证低优先级任务的执行
- O(1) 找下一个进程，O(logn) 增删子节点

##### \*Windows Scheduling
- 与linux相反，Windows将大数作为高优先级
- 优先级在`1-31`之间

