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

>[!tip]+ Datapath
>- Fetch -> Decode -> Execute
>- Combinational / Storage Elements 
>- Clocking Methodology
>在单周期处理器中，这些过程在一个周期内执行。就像水流从取指令开始，在一个周期内沿着电路一直到达终点。

##### Step3： Assemble DataPath meeting our requirements
- Register Transfer Requirements
- Instruction Fetch
- Read Operands and Execute Operation

>[!tip]+ 
>![[3_processor(single cycle).pdf#page=17]]

- 实际上指令内存和数据内存不在CPU中，对应位置是缓存(cache)
- Fetching Instruction
	- 依靠PC寄存器读出相应的指令
	- 同时使用一个加法器使下一个周期的PC增加4
		- 时钟周期内，加法器的数据一直在等待，直到时钟跳变再写入PC
		- 下一个时钟周期，PC读入新数据，很快使得输出稳定，让新值参与后续的组合电路

- Decoding Instruction
	- PC值输入到内存后得到一条指令。这个时候先让控制器对指令进行译码，同时对应地址的几位接入寄存器组的读端口。
	- 寄存器组中，读逻辑是一个组合逻辑。不论该指令是否会使用对应位置的寄存器，寄存器组都照读不误，在输出口输出对应的值。
		- 如果不使用也没关系，后面不使用就行了，这样并发也加快了速度。
	- 控制器是一个组合电路并且非常快速。（都是逻辑门控电路），根据指令会在对应控制线输出控制信号。指令的路线在此刻被规定，数据接下来就会沿着这条路一直走下去。

- Datapath of RR (R-type)
	- 输出寄存器的值经过ALU（受到控制信号控制指令ALUctr控制运算类型），将结果与写入寄存器的线联通，等待下个周期。同时控制信号会提示寄存器允许写入。下个周期时钟跳变的时候写入值。(RegWr)

- Datapath of OR Immediate Instruction
	- 指令的后16位进行0符号扩展为32位，同时接入ALU。此时需要一个选择控制信号来控制是寄存器的信号还是立即数的信号接入ALU，而这个信号也来自于CU(ALUsrc)
	- 由于写入的时候Rd、Rt都有可能写入，所以在写入的时候也需要一个选择控制信号控制谁的信号接入寄存器组的写信号。(RegDst)

- Datapath for Load Instruction
	- 对比之前的几个指令，lw指令需要额外添加与内存读取相关的线路
	- 立即数的扩展要分为有符号和无符号，所以需要控制信号(ExtOp)
	- ALUsrc设置1，ALUctr设置加法，此时ALU输出一个地址访问Memory
	- 内存读出一个值，类似寄存器组一样的组合电路。
	- 此时还需要一个选择：结果要兼容之前的电路，所以我们需要一个新的选择器和控制信号(MemtoReg)

- Datapath for Store Word
	- 寄存器组的二号线直接连接到内存的写入地址线
	- 类似寄存器组，还有一个控制写入的信号(MemWr)保证写入操作的安全。

- Datapath for Branch
	- 这个指令需要对PC进行更新
	- 先对两个寄存器做减法，再判断这个值是否为0，最后由这个判断决定是否更新PC
	- ALU减法后进行一个比较，输出一个“zero”控制信号。
	- 重写Next Addr Logic，接受zero控制信号、imm地址信号、PC的上次的值以及控制器的branch信号
		- 先执行加4操作得到第一个值
		- imm需要进行符号扩展
		- 将PC+4和imm相加得到第二个值
		- 这两个得到的值就是可能的PC下一步的值，所以通过zero、branch两个控制信号共同决定是否跳转

##### Step 4: Adding the Control
- 控制器主要通过opcode来输出控制信号，R型指令还有5位funct接入控制系统
- 将控制器分为两大类：主控和ALU控制信号。
	- 设计的深意：opcode是0时一定是Rtype，其他情况另算
	- 所以可以将Rtype与其他控制分开
	- 为什么主控不一起把所有活都干了？
		- 输入越少，实现越简单。