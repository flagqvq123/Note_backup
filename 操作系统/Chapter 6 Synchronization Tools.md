	在第六章和第八章中充斥着这样的矛盾：进程之间协作、依赖以及它们带来的同步问题

- Background
	- 生产者、消费者问题：
		- 生产者进程运行程序，需要定期将结果存储在内存缓冲区中。（生产数据）
		- 消费者进程负责使用生产者的结果（比如永久保存这些数据到硬盘），使用之后会清除这些数据。
		- 两个进程共享同一buffer。
		- 在实际情况中，消费者、生产者不是一个而是一组。
	- Race Condition 竞态问题
		- 生产者进程和消费者进程共享全局变量`counter`记录`buffer`中数据量。
		- 例如：producer的操作：
		- ```
		  LOAD  R1, COUNT
		  ADD   R1, 1
		  STORE R1, COUNT
		  ```
		- 而consumer的操作
		   - ```
	     LOAD  R2, COUNT
	     DEL   R2, 1
		 STORE R2, COUNT
		     ```
		- 交错执行：
		- ```
		  S0: LOAD  R1, COUNT
		  S1: ADD   R1, 1
		  S2: LOAD  R2, COUNT
		  S3: DEL   R2, 1
		  S4: STORE R1, COUNT
		  S5: STORE R2, COUNT
		  ```
		- 两个程序竞争修改同一个值，这被称为竞态
		- 解决方法：把并行变成串行(mutual exclusion 互斥)

### 互斥并行，临界区问题
- 解决方法：冲突代码需要标识在代码中，由操作系统完成临界的分配
```
do{
	(entry section)
		critical setion
	(exit section)
		remainder section
}while(true);
```
- 解决方法：
	- 1. 互斥：有且仅有一个进程能够在同一时间执行临界区代码
	- 2. 处理：进程执行完临界区代码后，就一定要通知操作系统立刻让下一个进程进入临界区并且不能推迟
	- 3. 有限等待：在这个临界区的进程等待队列中的进程要保证能够在有限时间内被响应。

- 解决方案：
	- 最早期解决方法：两个进程间的解决方案
		- 添加一个`flag`的布尔数组，表示进程准备状态
			- 进程到达入口代码前会将自己的`flag`置1
		- 添加`turn`这个全局变量作为标记符号，判断两个进程执行的先后
			- 如果没有它，那么两个进程同时置flag的时候就会锁死
			- 后修改turn的会使先修改turn的跳出while
		- 添加`while`循环，当另一个进程执行结束后跳出循环。

### Synchronization Hardware 硬件同步
##### Memory Barriers
- Memory model:
	- 强顺序：一个处理器的内存修改会立即被所有其他处理器看到
	- 弱顺序：其中一个处理器的内存修改可能不会立即被所有其他处理器看到。
- 内存屏障：让一块内存空间只能被==一个进程==访问
	- Store memory barrier 写操作屏障
		- 不可以被写入，但是可以读。
	- Load memory barrier 读操作屏障
	- Full memory barrier 读写都受到保护
- 简单粗暴的做法：破坏了进程共享性，损失了很多良好的性质。==很少使用==

##### Hardware Instructions
- Test-and-Set instruction:
	- 在临界区代码两端分别进行加锁、解锁操作。
	- 相比memory barrier，它具有动态性，不会破坏共享性。
	- ```
	  while (test_and_set(&lock)){ // 先找到锁的地址，找到说明没有被锁，此时上锁
		  /* critical section */
	  
			lock = false;	        // 解锁，让其他进程能够找到该锁
		}
	  ```
- Compare_and_swap instruction:
	- ```
	  int compare_and_swap(int *value, int expected, int new_value) {
		  int temp = *value
		  
		  if (*value == expected)
			  *value = new_value;
			  
		return temp;
	  }
	  ```

##### Atomic Variables 原子操作
- 原子操作是指令的最小单位，不能被打断不能被分割
- 在声明一个变量时使用`atomic`标识，表明其具有原子性，所有与它相关的操作都不能被分割。
- 原子操作需要CPU架构的支持。

### OS Synchronization
	硬件方式实现太过简单，不具有对复杂情况的普遍性，所以软件实现更通用
##### Semaphore 信号量
- 信号量的定义是一个整形变量(例如S)
- 操作系统为此变量配置了两套函数（加锁、解锁）
	- `wait(synch)` 加锁
		- ```
		  wait (S) {
			  while(S<=0); // busy wait 忙等待
			  S--;
		  } // S=1时成功执行并且将S置0，其他程序调用此函数的时候就会停止。
		  ```
	- `signal(synch)` 解锁
		- ```
		  signal (S) {
			  S++; // S自增，让其他进程继续执行。
		  }
		  ```
	- 两个函数都是封装结束的原子操作。
	- 本质上是通过控制S的量来实现的
		- 更复杂的情况：S不一定是1，而是一个更高的值，保持进入该临界区的进程数量不超过某个数。
	- busy wait 忙等待
		- 表示进程得到了执行，但是进入不了临界区的状态
			- 此时进程是running状态，但是却在保持运行中的等待状态。
		- 在进程数量特别多的时候会极大幅拖慢系统速度
		- 处理方法：
			- block：将忙等待的进程放置在特定的等待队列中。
			- wakeup：唤醒等待队列中的处于忙等待状态的进程。

- 计数信号量与二进制信号量
	- 计数信号量是个整形，二进制信号量是一个二进制数（互斥锁）。

- 死锁(Deadlock)
	- 考虑这样的情况：
		- P0进程需要S、Q两把锁，P1进程需要Q、S两把锁
		- 两个进程分别先获得了第一个锁，发现第二个锁没有解锁
		- 两个进程此时卡死。
	- 解决方法在第八章揭晓

- 优先级反转：
	- 优先级反转会加剧饥饿现象

>[!tip]- 优先级反转
>![[lec6-OS-new.pdf#page=95]]
>- 资源获取的先后会导致优先级反转，此时一定是先获取到资源的进程优先运行。
- 
