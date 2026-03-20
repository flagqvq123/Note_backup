### The language a computer understand
- 指令：类似于语言的词汇
- 指令集（ISA）：词汇表
- 不同的电脑拥有不同的ISAs，例如ARM指令集、x86指令集
	- 也有相同的指令集
- 指令使用二进制编码，所以实际上机器无法区分这是一个数还是指令
	- 这是指令还是其他的取决于

##### MIPS 指令分块

| op   | rs   | rt   | rd   | shamt | funct |
| ---- | ---- | ---- | ---- | ----- | ----- |
| 6bit | 5bit | 5bit | 5bit | 5bit  | 6bit  |
- funct是op的扩展位，让操作指令从64个指令到最多4096个指令操作。

### MIPS(RISC) 设计原则
- RISC : reduced, 简单指令集。相对于变长的指令集结果CISC（complex）
- 简单而产生规律性：
	- 简单的指令集产生统一的长度
	- 少量的指令类型（3种）
	- opcode总是在前6位
- 小就是快：
	- 小指令集、小寄存器数、少寻址模式
- 加速经常性事件：
	- 算数运算放在寄存器
	- 允许立即数指令

### MIPS-32 ISA(由于简单性，只讲32位)
- 指令分类：
	- 运算
	- 加载/存储
	- 跳转分支
	- 浮点数
	- 内存管理
	- 其他特殊
- 3种指令格式：一共32位
	- R型：register type
	- I型：immediate type, 把最后16位拿来存一个常数
	- J型：jump type, 将后26位存一个跳转地址
	- 由于rs, rt, rd各自有5bit，所以可以定位32个寄存器。
- 3个特殊寄存器：PC、HI、LO
	- PC：存储指令地址
	- HI：high
	- LO：low
- MIPS中一个字是32位。

##### MIPS Register file 寄存器组
- 有32个寄存器，两个读端口，一个写端口
	- 可以同时读两个寄存器，但是只能写一个寄存器。
	- 每一个读写口也对应一个5位的读写的地址输入输出口。
	- 写端口还有一个控制信号，保证寄存器内数据的安全。
	- 有32个地址，0~31号。
	- 同时还有一个CLK信号，图上没画
- “通用寄存器”：更易于编译器使用

>[!tip]- 看图
>![[2_MIPS ISA.pdf#page=7]]

##### Aside : MIPS Register Convention 寄存器约定
- 0号：常数0，无法改变。（非常常见）
- 1号：仅给汇编器预留，无法使用。
- 2-3号：储存返回值（函数）
- 4-7号：函数参数
- 8-15
- 28：全局指针
- 29：栈指针
- 30：

##### MIPS Arithmetic Instructions
- add, sub
- 每个算数指令进行一个操作，需要三个操作数
- `add $t0, $s1, $s2`
	- 两个源操作数，结果存在目标寄存器。
- 对应的指令编码：R型
	- `op: instruction, 
	- `rs: $s1, 
	- `rt: $s2, 
	- `td: $t0, 
	- `ams: 全0
	- `funct: instruction

##### MIPS Memory Access Instruction
- lw, sw
- `lw   $t0, 4($s3)   # load word from memory`
- `sw   $t0, 8($s3)   # store word to memory`
	- 读写
- 内存的表示：基地址+偏移量。基地址32位存在一个寄存器里。
	- 例如`4($s3)`就是{s3}+4的意思
- 对应的指令编码，I型
	- `op: instruction`
	- `rs: $s3`
	- `rt: $t0`
	- `immediate: 数字`
- 