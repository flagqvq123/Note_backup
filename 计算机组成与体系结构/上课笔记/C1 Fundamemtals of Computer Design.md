##### Two Kinds of Parallelism in Applications
[[1_fundamentals_A.pdf#page=40|1_fundamentals_A, 页面 40]]

##### Flynn's Taxonomy
	- Single instruction stream, single data stream(SISD)
		一般的单核计算机
	- Single instruction stream, multiple data streams (SIMD)
		GPU架构，或者说向量

### Defining Computer Architecture
旧有的观点：==ISA==
	寄存器设计、内存设计、......
	![[1_fundamentals_B.pdf#page=3|1_fundamentals_B, 页面 3]]

![[1_fundamentals_B.pdf#page=4]]



### 1.9 Quantitative Principles 定量原则
	三条原则
	阿姆达尔定律与经常性事件

- 三条原则：
	- 使用并行优势
		- 数据与任务并发
		- 流水线、缓存技术
		- 多处理器、多核、向量
	- 局部性原理
		- 时间局部性：在调用一条指令、数据之后的短时间内会多次再次调用该地址。
		- 空间局部性：在调用一条指令、数据之后的短时间内会多次调用附近的地址。
	- 关注经常性事件
		- ==阿姆达尔定律==：speedup = 1/(1-x+x'/x)
		- 使用新功能的性能提升受限于新功能可用时间的百分比

- CPU执行时间：CPU所用周期数 * 时钟周期时间
	- ==CPI==：`clockcycle per instruction`单指令平均时钟周期数
	- CPU time = CPI * n * Cycletime
		- n = Instruction count
	- 这个公式告诉我们：提升CPU性能可以从==三个方向入手==：CPI, Cycletime、instruction count
		- clocktime : 硬件升级，增加CPU频率
		- CPI：ISA与ISA的实现
		- instruction count：ISA与编译技术、算法。


- CPI分类：如果一个指令集里面某些指令的确与其他指令CPI相差太大，此时平均难以实现，则考虑分类为几个type分别计算CPI
- 
