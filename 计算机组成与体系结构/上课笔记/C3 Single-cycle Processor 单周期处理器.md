##### 引入：指令层级
- 高级语言：C, Java, Python
- 汇编语言：除了label以外没有符号，一行一个操作
- 机器语言：二进制编码，label转化为地址

##### Processor：功能的集成
- Datapath：从输入到输出的通路
- Control Unit：数据流动的控制器
- Clocking methodology：时间的控制

### The Processor: Datapath & Control
- 我们构建一个仅处理八条数据的MIPS指令集
- Single-cycle processor：每一个指令周期都只执行一条指令
	- PC递增并取指令`pc = pc + 4`
	- 解码指令
	- 执行指令
- (除了J指令外)，所有指令都需要ALU
-  RTL：定义指令的含义
	- 所有指令首先取指令

##### Step1：
- Memory：读取
- Registers：
	- read rs
	- read rt
	- write rt or rd
- PC
- Extender(sign/zero extend)
- Add/Sub/OR unit
- Add 4 to PC

##### Step 2：Components of the Datapath
- 组合元件：没有时钟概念，只要有信号就会输出
	- Adder
	- MUX
	- ALU
- 存储元件：输出不是随时随地发生变化的
	- 寄存器组：Register File
	- ![[Pasted image 20260323111323.png]]
	- 两个寄存器读端口，一个寄存器写端口
	- W,A,B对应分别一个地址线
	- CLK：
		- 读寄存器的时候像组合元件，信号发生就输出
		- 进行写操作时，只有当CLK跳变的时候才会发生（上升沿或下降沿）
			- 意味着一个数据至少停留一个周期
	- WE：Write Enable，根据控制器产生，只有WE为1时才能写入。

##### 时钟同步：
- 时钟周期决定了什么时候存储、什么时候改变数据。
	- 边缘驱动：只有当上升沿或下降沿发生时，指令才会执行，状态才会改变。
- 基本过程：==状态1== ------------> 组合逻辑电路 ----==时钟跳变==----> ==状态2==
	- 保证在时钟跳变前，数据应当在储存元件前待命
	- 要求时钟周期的长度必须考虑到组合逻辑电路的速度。
		- ==Cycletime== = CLK-to-Q + ==Longest Delay Path== + setup（稳定输入信号，在跳变前一段时间就不能改变了） + Clock Skew（容错）
		- Cycletime不可能无限短，它主要被Longest Delay Path卡住了。